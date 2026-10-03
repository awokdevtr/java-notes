# copy-on-write-arraylist

Birden fazla thread'in aynı listeyi okuduğu ama yazmanın nadiren olduğu senaryolarda `ArrayList`'i doğrudan paylaşmak güvenli değildir: eşzamanlı değişiklik ya `ConcurrentModificationException` fırlatır ya da veri bozulmasına yol açar. `Collections.synchronizedList` bu sorunu çözer ama her okuma işlemini de kilitler, bu yüzden okuma-ağırlıklı yüklerde gereksiz çekişme yaratır. `CopyOnWriteArrayList`, yazma maliyetini artırarak okumaları tamamen kilitsiz ve tutarlı hale getiren alternatif bir stratejidir.

## Yazma sırasında dizi kopyalama

```java
var list = new java.util.concurrent.CopyOnWriteArrayList<String>();
list.add("a");
list.add("b");

for (String s : list) {
    list.add("c " + s); // iterator üzerinde ConcurrentModificationException FIRLATMAZ
}
System.out.println(list.size());
```

`add`, `remove`, `set` gibi her değiştirme işlemi, iç dizinin tamamını yeni bir kopyaya aktarır ve referansı atomik olarak günceller. Bu yüzden `n` elemanlı bir listeye eleman eklemek `O(n)` maliyete sahiptir; sık yazılan büyük listelerde performans ciddi şekilde düşer. Buna karşılık `iterator()` çağrıldığı anda alınan dizi referansı üzerinde çalışır, dolayısıyla döngü sırasında başka bir thread listeyi değiştirse bile iterator asla `ConcurrentModificationException` fırlatmaz.

## Anlık görüntü semantiği

```java
var snapshot = list.iterator();
list.add("yeni eleman");
// snapshot, "yeni eleman" eklenmeden önceki hali gösterir
snapshot.forEachRemaining(System.out::println);
```

Iterator, oluşturulduğu anki dizinin değişmez bir görüntüsünü (`snapshot`) tutar; sonradan yapılan eklemeler ya da silmeler bu iterator'a yansımaz. Bu davranış `fail-fast` koleksiyonların tersidir ve `weakly consistent` olarak adlandırılır: iterator her zaman tutarlı ama güncelliği garanti edilmeyen bir görünüm sunar.

## Ne zaman ise yarar

- Okuma sayısının yazmaya kıyasla çok yüksek olduğu, elemanların az sayıda ve nadiren değiştiği listelerde (örneğin listener/observer koleksiyonları) idealdir.
- Büyük listelerde ya da sık yazma yapılan senaryolarda her yazımın tüm diziyi kopyalaması nedeniyle tercih edilmemelidir; bu durumda `ConcurrentLinkedQueue` veya senkronize bir yapı daha uygundur.

## Denemek için

```
jshell> var l = new java.util.concurrent.CopyOnWriteArrayList<Integer>(java.util.List.of(1,2,3))
jshell> var it = l.iterator()
jshell> l.add(4)
jshell> it.forEachRemaining(System.out::println)
```

`it` sadece `1, 2, 3` yazdırır; `4` elemanı iterator oluşturulduktan sonra eklendiği için görünmez.

[Kaynak: https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/concurrent/CopyOnWriteArrayList.html]
