# BitSet ile Verimli Bit Manipülasyonu

Çok sayıda `boolean` değeri tutmak gerektiğinde `boolean[]` veya `Set<Integer>` kullanmak hem bellek hem performans açısından verimsizdir: her `boolean` elemanı JVM'de en az 1 byte kaplar, `Set<Integer>` ise her eleman için ayrı bir nesne ve kutulama (`autoboxing`) maliyeti getirir. `java.util.BitSet`, keyfi uzunlukta bir bit dizisini `long[]` üzerinde paketleyerek bu maliyeti ortadan kaldırır ve kümelerde sık kullanılan birleşim/kesişim gibi işlemleri tek makine komutu seviyesinde hızlandırır.

## Temel kullanım ve boyut büyümesi

```java
BitSet bits = new BitSet();   // başlangıçta 64 bitlik kapasite
bits.set(3);
bits.set(10);
bits.set(100);                // gerektiğinde otomatik büyür

System.out.println(bits);        // {3, 10, 100}
System.out.println(bits.get(10)); // true
System.out.println(bits.cardinality()); // 3 -- set edilmiş bit sayısı
```

`BitSet`, sabit bir boyutla sınırlı değildir; `set(index)` çağrısı mevcut kapasiteyi aşarsa dahili `long[]` dizisi otomatik olarak büyütülür, bu yüzden constructor'a verilen başlangıç boyutu sadece ilk ayırmayı belirler, üst sınır değildir. `cardinality()` metodu set edilmiş bit sayısını döndürür ve bunu hesaplamak için `Long.bitCount()` kullanır, dolayısıyla tüm bitleri tek tek saymaktan çok daha hızlıdır.

## Küme işlemleri: and, or, xor

```java
BitSet evens = new BitSet();
BitSet primes = new BitSet();
for (int i = 0; i < 20; i += 2) evens.set(i);
for (int p : new int[]{2, 3, 5, 7, 11, 13, 17, 19}) primes.set(p);

BitSet evenPrimes = (BitSet) evens.clone();
evenPrimes.and(primes);      // kesişim: {2}

BitSet union = (BitSet) evens.clone();
union.or(primes);            // birleşim

evens.xor(primes);           // simetrik fark: sadece birinde olanlar
```

`and()`, `or()`, `xor()` ve `andNot()` metotları iki `BitSet`'i kendi `long` kelimeleri üzerinde bit düzeyinde birleştirir, bu da işlemi her elemanı tek tek dolaşan bir `Set<Integer>` birleşimine kıyasla kayda değer ölçüde hızlandırır. Bu metotlar çağrıldıkları nesneyi yerinde (`in-place`) değiştirir, dolayısıyla orijinali korumak isteniyorsa önce `clone()` alınmalıdır.

## Dikkat edilmesi gerekenler

- `BitSet` thread-safe değildir; eşzamanlı erişim gerekiyorsa harici senkronizasyon şarttır.
- Negatif indeksle `set()`/`get()` çağrısı `IndexOutOfBoundsException` fırlatır; `BitSet` sadece negatif olmayan indeksleri destekler.
- `equals()` ve `hashCode()`, mantıksal boyuttan (`length()`) bağımsız olarak sadece set edilmiş bitlere bakar; yani kapasiteleri farklı ama aynı bitler set edilmiş iki `BitSet` birbirine eşittir.
- Çok seyrek (`sparse`) ama çok büyük aralıklı veriler için bellek kazancı azalır; böyle durumlarda `Set<Integer>` daha uygun olabilir.

## Denemek için

```
jshell> var b = new java.util.BitSet()
jshell> b.set(1); b.set(4); b.set(9)
jshell> b.flip(4)
jshell> System.out.println(b)
```

`flip(4)` çağrısı 4. biti tersine çevirir (set edilmişse temizler), sonuçta `{1, 9}` kümesini görürsünüz.

[Kaynak: https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/BitSet.html]
