# SequencedCollection

`List` zaten sıralıydı ama ilk/son elemana erişmek koleksiyona göre farklı metotlar gerektiriyordu: `list.get(0)` ile `list.get(list.size() - 1)`, `LinkedList` için `getFirst()`/`getLast()`, `LinkedHashSet` içinse hiçbir doğrudan yol. Koleksiyonu ters sırada dolaşmak da her tür için ayrı bir numara: `Collections.reverse()` ile kopya almak, `ListIterator`'ı sona sarmak ya da `TreeSet`'te `descendingSet()`. Java 21, `SequencedCollection`, `SequencedSet` ve `SequencedMap` arayüzleriyle bu tutarsızlığı ortadan kaldırdı.

## Ortak uç erişim ve ekleme

```java
List<String> names = new ArrayList<>(List.of("Ayşe", "Mehmet", "Fatma"));

names.addFirst("Zeynep");
names.addLast("Can");

System.out.println(names.getFirst());   // Zeynep
System.out.println(names.getLast());    // Can

names.removeFirst();
```

`List`, `Deque`, `LinkedHashSet` ve `LinkedHashMap` artık ortak bir `SequencedCollection` sözleşmesini uygular: `addFirst`, `addLast`, `getFirst`, `getLast`, `removeFirst`, `removeLast`. Bu metotlar önceden de bazı tiplerde vardı (`Deque`'te mesela) ama şimdi `List` dahil hepsinde aynı isimle, aynı davranışla bulunuyor. `ArrayList` gibi sabit boyutlu olmayan bir dizi üzerine kurulu tipte `addFirst` çağırmak baştan kaydırma maliyeti taşır; bu, arayüzün değil altındaki veri yapısının bir kısıtıdır.

## Tersine görünüm: reversed()

```java
LinkedHashMap<Integer, String> sira = new LinkedHashMap<>();
sira.put(1, "bir");
sira.put(2, "iki");
sira.put(3, "üç");

for (var entry : sira.sequencedEntrySet().reversed()) {
    System.out.println(entry.getKey() + " -> " + entry.getValue());
}
```

`reversed()` koleksiyonu kopyalamaz; orijinaliyle canlı bağlantısı olan ters çevrilmiş bir görünüm (`view`) döndürür. Alttaki koleksiyon değişirse görünüm de güncellenir. `SequencedMap` için `reversed()` yerine `sequencedKeySet()`, `sequencedValues()` ve `sequencedEntrySet()` ile de aynı ters gezinti sağlanabilir; `Map` arayüzünün kendisi sıralı olmadığı için bu ek metotlar `LinkedHashMap` ve `TreeMap` gibi zaten sıra garantisi veren implementasyonlarda devreye girer.

## Dikkat edilmesi gerekenler

- `HashSet` ve `HashMap`, `Sequenced*` arayüzlerini uygulamaz; sıra garantisi olmayan bir koleksiyonda "ilk eleman" kavramı tanımsızdır.
- `Collections.unmodifiableList()` gibi değişmez görünümlerde `addFirst`/`removeFirst` çağrısı hâlâ `UnsupportedOperationException` fırlatır; arayüz yeni olsa da değişmezlik kuralı değişmedi.
- `reversed()` bir kopya değil görünüm olduğu için, görünüm üzerinde gezinirken orijinal koleksiyonu değiştirmek `ConcurrentModificationException`'a yol açabilir.

## Denemek için

```
jshell> var d = new java.util.ArrayDeque<Integer>(java.util.List.of(1, 2, 3))
jshell> d.reversed()
jshell> d.addFirst(0); d.getFirst(); d.getLast()
```

`d.reversed()` çağrısının `[3, 2, 1]` görünümünü anında verdiğini, bunun için hiçbir eleman kopyalanmadığını görürsünüz. Ardından `addFirst(0)` ile orijinal `d`'nin başına eleman eklediğinizde, daha önce aldığınız `reversed()` görünümünü tekrar yazdırırsanız yeni elemanın sonda belirdiğini fark edersiniz.

[Kaynak: https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/SequencedCollection.html]
