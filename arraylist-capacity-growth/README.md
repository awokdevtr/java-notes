# arraylist-capacity-growth

`ArrayList`, görünüşte sabit boyutlu bir dizi gibi çalışsa da aslında arka planda büyüyebilen bir `Object[]` tutar. Bu dizi doluverince `ArrayList` onu olduğu yerde genişletemez; yeni, daha büyük bir dizi ayırıp eski elemanları oraya kopyalar. Bu büyüme stratejisini ve maliyetini bilmemek, döngü içinde binlerce eleman eklerken fark edilmeyen bir performans kaybına yol açabilir.

## Varsayılan büyüme

```java
List<Integer> list = new ArrayList<>();
for (int i = 0; i < 20; i++) {
    list.add(i);
}
```

Parametresiz kurucuyla oluşturulan bir `ArrayList`, ilk eleman eklenene kadar iç dizisini hiç ayırmaz; ilk `add()` çağrısında kapasite `10`'a çıkar. Kapasite dolduğunda büyüme oranı yaklaşık `%50`'dir (`oldCapacity + (oldCapacity >> 1)`), yani `10 -> 15 -> 22 -> 33 ...` şeklinde ilerler. Her büyüme adımı `Arrays.copyOf` ile `O(n)` maliyetli bir kopyalama gerektirir, ancak bu maliyet amortize edildiğinde tek bir `add()` çağrısı ortalama `O(1)` kalır.

## Kapasiteyi önceden belirlemek

```java
List<String> names = new ArrayList<>(10_000);
for (int i = 0; i < 10_000; i++) {
    names.add("user-" + i);
}
names.trimToSize();
```

Eleman sayısı önceden biliniyorsa, kurucuya ilk kapasiteyi vermek tekrarlanan kopyalama adımlarının tamamını ortadan kaldırır; `ensureCapacity(int)` da aynı amaçla sonradan çağrılabilir. `trimToSize()` ise ters yönde çalışır: iç diziyi gerçek eleman sayısına küçültüp kullanılmayan boş slotların bellek tutmasını önler, ancak bu işlem de yeni bir dizi kopyalaması gerektirir.

## Dikkat edilmesi gerekenler

- `size()` ile kapasite farklı şeylerdir; `size()` eleman sayısını, kapasite ise iç dizinin boyutunu verir ve doğrudan sorgulanamaz.
- Büyüme sırasında geçici olarak hem eski hem yeni dizi bellekte bulunur; çok büyük listelerde bu anlık bellek tepe noktasına dikkat edilmelidir.
- `LinkedList`'in aksine `ArrayList`'te büyüme maliyeti öngörülebilir ama düzensizdir; gerçek zamanlı sistemlerde tekil `add()` çağrılarının gecikmesi sabit değildir.

## Denemek için

```
jshell> var list = new java.util.ArrayList<Integer>()
jshell> for (int i = 0; i < 12; i++) list.add(i)
jshell> list.ensureCapacity(100)
```

İlk döngü varsayılan `10` kapasitesinin `15`'e büyüdüğü anı tetikler; `ensureCapacity(100)` çağrısı ise kapasiteyi tek seferde istenen büyüklüğe çıkararak sonraki `add()` çağrılarının kopyalama yapmadan ilerlemesini sağlar.

[Kaynak: https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/ArrayList.html]
