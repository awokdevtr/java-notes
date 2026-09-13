# ArrayList vs LinkedList iç yapısı

`List` arayüzünü uygulayan bu iki sınıf birbirinin yerine geçebilir gibi görünür ama iç veri yapıları tamamen farklıdır: `ArrayList` büyüyebilen bir dizi (`Object[]`) üzerine kuruludur, `LinkedList` ise çift yönlü bağlı liste (`doubly linked list`) düğümlerinden oluşur. Bu fark, hangi operasyonun hızlı hangisinin yavaş olacağını doğrudan belirler; yanlış seçim, veri büyüdükçe fark edilen performans sorunlarına yol açar.

## Rastgele erişim vs ekleme/çıkarma

```java
List<String> dizi = new ArrayList<>();
dizi.add("a");
dizi.add("b");
dizi.get(1);

List<String> bagli = new LinkedList<>();
bagli.add("a");
bagli.add("b");
bagli.get(1);
```

`ArrayList.get(index)` doğrudan dizi indeksine erişir, bu yüzden `O(1)`'dir. `LinkedList.get(index)` ise baştan veya sondan başlayıp düğüm düğüm ilerlemek zorundadır, dolayısıyla `O(n)` maliyetlidir. Buna karşılık listenin başına veya ortasına eleman ekleme/çıkarma `LinkedList`'te sadece komşu düğümlerin referanslarını güncellemeyi gerektirdiği için `O(1)`'dir; `ArrayList`'te ise ekleme noktasından sonraki tüm elemanların bir konum kaydırılması gerektiğinden `O(n)`'dir.

## Dizi büyümesi (capacity growth)

```java
ArrayList<Integer> sayilar = new ArrayList<>();
for (int i = 0; i < 20; i++) {
    sayilar.add(i);
}
```

`ArrayList` başlangıçta boş veya küçük bir iç diziyle oluşturulur; kapasite dolduğunda yeni ve daha büyük bir dizi (`%50` oranında büyüme) ayrılır ve eski elemanlar `Arrays.copyOf` ile kopyalanır. Bu ara sıra oluşan `O(n)` kopyalama maliyeti amortize edildiğinde `add()` ortalama `O(1)` kalır, ancak eleman sayısı önceden biliniyorsa `new ArrayList<>(kapasite)` ile gereksiz kopyalamalar önlenebilir.

## Ne zaman ise yarar

- Sık `get(index)` çağrısı veya iterasyon varsa `ArrayList` tercih edilmeli; bellek düzeni ardışık olduğundan `cache locality` da avantaj sağlar.
- Listenin başına/ortasına sık ekleme-çıkarma yapılıyorsa ve indeksle rastgele erişim nadirse `LinkedList` düşünülebilir; ama pratikte `ArrayDeque` çoğu kuyruk/yığın senaryosunda `LinkedList`'ten daha hızlıdır.
- `LinkedList` her eleman için ekstra düğüm nesnesi (`prev`/`next` referansları) tuttuğundan `ArrayList`'e göre belirgin şekilde daha fazla bellek kullanır.

## Denemek için

```
jshell> var l = new java.util.LinkedList<Integer>(); for (int i=0;i<100000;i++) l.add(i); long t0=System.nanoTime(); l.get(90000); System.out.println((System.nanoTime()-t0)/1000);
```

Aynı kodu `ArrayList` ile tekrarlayınca `get(90000)` süresinin mikrosaniyeler mertebesinden pratik olarak sıfıra düştüğü gözlenir; çünkü `LinkedList` 90.000 düğüm boyunca gezerken `ArrayList` doğrudan indekse atlar.

[Kaynak: https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/LinkedList.html]
