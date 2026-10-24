# List.subList ile görünüm (view) semantiği

Bir listenin belirli bir aralığını işlemek için çoğu zaman `subList()` çağrılır ve dönen nesne bağımsız, yeni bir liste gibi düşünülür. Oysa `List.subList(from, to)` kopya üretmez; orijinal listenin üzerinde bir **pencere** açar. Bu pencere üzerinden yapılan ekleme/silme orijinal listeyi de değiştirir ve tersine, orijinal liste alt liste dışından değiştirilirse alt liste kullanılamaz hale gelir.

## Yazma orijinale yansır

```java
List<String> isimler = new ArrayList<>(List.of("ali", "veli", "ayse", "fatma", "can"));
List<String> orta = isimler.subList(1, 4); // veli, ayse, fatma

orta.clear();
System.out.println(isimler); // [ali, can]
```

`subList` ile dönen `orta`, `isimler` üzerindeki aynı dizi hücrelerine bakan bir görünümdür. `orta.clear()` çağrısı alt listedeki elemanları silmekle kalmaz, orijinal `isimler` listesinden de o aralığı çıkarır. Bu özellik aslında kasıtlı bir API tasarımıdır: bir listenin ortasından toplu silme veya ekleme yapmak için `list.subList(a, b).clear()` idiomu resmi olarak önerilir.

## Orijinali dışarıdan değiştirmek alt listeyi bozar

```java
List<Integer> sayilar = new ArrayList<>(List.of(10, 20, 30, 40));
List<Integer> altListe = sayilar.subList(1, 3); // 20, 30

sayilar.add(50); // yapisal degisiklik dogrudan ana listeye
altListe.get(0); // ConcurrentModificationException
```

`subList`'in döndürdüğü görünüm, arkasındaki `ArrayList`'in `modCount` değerini paylaşır. Ana liste alt liste *üzerinden değil* kendi referansı üzerinden yapısal olarak değiştirilirse (ekleme, silme, `clear`), alt liste bu değişikliği fark eder ve bir dahaki erişiminde `fail-fast` davranışıyla `ConcurrentModificationException` fırlatır. Alt listeyi güvenle kullanmaya devam etmek istiyorsanız, ana listeyi yalnızca o alt liste üzerinden değiştirmeniz gerekir.

## Dikkat edilmesi gerekenler

- `subList` sonucu `ArrayList` gibi bağımsız bir kopya **değildir**; kopya istiyorsanız `new ArrayList<>(list.subList(a, b))` yazmalısınız.
- Alt liste üzerinde yapılan `set`, `get`, `add`, `remove` çağrıları her zaman orijinal listenin ilgili indekslerine yönlendirilir (indeks hesaplaması `offset` ile yapılır).
- Alt listeyi uzun süre elde tutup ana listeyi başka bir yoldan değiştirmek, "kullanılamaz" bir referans bırakır; bu durum derleme zamanında yakalanmaz, sadece çalışma zamanında patlar.

## Denemek için

```
jshell> var liste = new java.util.ArrayList<>(java.util.List.of("a","b","c","d","e"))
jshell> var alt = liste.subList(1, 4)
jshell> alt.set(0, "X")
jshell> liste
```

`liste`'nin `[a, X, c, d, e]` olduğunu, yani alt listedeki değişikliğin orijinale yansıdığını görürsünüz. Ardından `liste.add("f")` çalıştırıp `alt.get(0)` çağırırsanız `ConcurrentModificationException` alırsınız.

[Kaynak: https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/List.html#subList(int,int)]
