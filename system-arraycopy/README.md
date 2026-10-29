# system-arraycopy

Bir dizinin elemanlarını başka bir diziye (ya da aynı dizinin başka bir bölgesine) taşımak gerektiğinde elle yazılan bir `for` döngüsü hem yavaştır hem de kaynak ile hedef aralıkları çakıştığında sinsi bukadar hataya açıktır. `System.arraycopy`, bu kopyalama işini JVM'in `intrinsic` olarak optimize ettiği `native` bir metoda devreder ve çakışan aralıkları da doğru şekilde ele alır.

## Temel kullanım

```java
int[] source = {10, 20, 30, 40, 50};
int[] dest = new int[5];

System.arraycopy(source, 1, dest, 0, 3);
System.out.println(java.util.Arrays.toString(dest)); // [20, 30, 40, 0, 0]
```

İmza `arraycopy(Object src, int srcPos, Object dest, int destPos, int length)` şeklindedir. Metot `src` dizisinin `srcPos`'tan başlayan `length` elemanını `dest` dizisinin `destPos` konumuna kopyalar. Her iki dizi de aynı bileşen tipine (ya da referans tipi uyumluluğuna) sahip olmalıdır; aksi halde `ArrayStoreException`, sınırlar aşılırsa `IndexOutOfBoundsException` fırlatılır.

## Çakışan bölgeler (overlapping copy)

```java
int[] arr = {1, 2, 3, 4, 5};
System.arraycopy(arr, 0, arr, 1, 4);
System.out.println(java.util.Arrays.toString(arr)); // [1, 1, 2, 3, 4]
```

Kaynak ve hedef aynı dizi olduğunda ve aralıklar üst üste bindiğinde `arraycopy`, kopyalamayı sanki önce kaynak bölge ayrı bir geçici diziye alınmış gibi gerçekleştirir; yani kopyalanan değerler kendi üzerine yazılmadan önce okunur. Bu garanti, elle yazılan bir ileri yönlü döngünün çakışan bölgelerde elemanları bozmasını önler ve `Arrays.copyOfRange`, `ArrayList` içindeki kaydırma işlemleri gibi birçok standart kütüphane metodunun temelini oluşturur.

## Ne zaman ise yarar

- `Arrays.copyOf`, `Arrays.copyOfRange`, `ArrayList.add(index, e)` gibi metotlar perde arkasında `arraycopy`'yi çağırır; kendi kodunuzda dizi kaydırma veya genişletme yapıyorsanız döngü yazmak yerine doğrudan bunu kullanmak daha hızlı ve daha az hataya açıktır.
- Primitif dizilerde eleman bazlı kopyalama gerektiğinde `clone()`'a göre kısmi aralık kopyalamaya izin verir.
- Nesne dizilerinde yalnızca referanslar kopyalanır; elemanların kendisi `deep copy` edilmez.

## Denemek için

```
jshell> int[] a = {1,2,3,4,5}
jshell> System.arraycopy(a, 0, a, 2, 3)
jshell> java.util.Arrays.toString(a)
```

Sonucun `[1, 2, 1, 2, 3]` olduğunu, yani `a[0]` ve `a[1]`'in kaynak olarak okunduktan sonra hedefe yazıldığını görürsünüz; elle yazılmış naif bir ileri döngü bu durumda veriyi bozardı.

[Kaynak: https://docs.oracle.com/javase/8/docs/api/java/lang/System.html#arraycopy-java.lang.Object-int-java.lang.Object-int-int-]
