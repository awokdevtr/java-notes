# arrays-binarysearch-contract

Büyük bir dizide belirli bir elemanı aramak için diziyi baştan sona taramak `O(n)` sürer. `Arrays.binarySearch()`, ikili arama algoritmasıyla bu maliyeti `O(log n)`'e indirir, ama bunun karşılığında sessiz bir ön koşul dayatır: dizinin önceden **sıralı** olması. Bu koşul derleme zamanında kontrol edilmez; metot imzasında da görünmez, dolayısıyla ihlal edilmesi derhal bir istisna değil, sinsi bir yanlış sonuç üretir.

## Sıralı dizide kullanım ve sonuç kodu

```java
int[] values = {1, 3, 5, 7, 9, 11};

int found = Arrays.binarySearch(values, 7);
System.out.println(found); // 3 (index)

int missing = Arrays.binarySearch(values, 6);
System.out.println(missing); // negatif: -(insertionPoint) - 1
```

Eleman bulunduğunda metot onun indeksini döndürür. Bulunamadığında ise rastgele bir `-1` değil, elemanın sıralı dizide hangi noktaya eklenmesi gerektiğini kodlayan negatif bir sayı döner: `-(insertionPoint) - 1`. Bu sayı `(-result - 1)` ile geri çözülerek ekleme noktası elde edilebilir; bu da `binarySearch`'ü sadece arama değil, sıralı yapıyı bozmadan ekleme noktası bulma aracı olarak da kullanılabilir kılar.

## Sıralanmamış dizide tanımsız davranış

```java
int[] unsorted = {9, 3, 7, 1, 5};

int result = Arrays.binarySearch(unsorted, 5);
System.out.println(result); // -3 -- ama 5, index 4'te duruyor!
```

Algoritma her adımda ortadaki elemana bakıp aramayı dizinin sol veya sağ yarısına daraltır; bu daraltma kararı dizinin sıralı olduğu varsayımına dayanır. Dizi sıralı değilse bu varsayım geçersiz kalır ve algoritma yanlış yarıyı eleyebilir. Sonuç: eleman dizide fiilen bulunsa da `-1` ya da tutarsız bir negatif değer dönebilir, hatta bazı girdilerde var olmayan bir elemanı "bulduğunu" iddia edebilir. Javadoc bunu açıkça "sonuç tanımsızdır (`undefined`)" olarak belirtir, `ArrayIndexOutOfBoundsException` gibi bir uyarı fırlatılmaz.

## Dikkat edilmesi gerekenler

- Metodu çağırmadan önce dizi `Arrays.sort()` ile sıralanmış olmalı; `binarySearch` bunu kendisi doğrulamaz ve doğrulamak da `O(n)` sürer, yani kazancı sıfırlar.
- Nesne dizilerinde (`Object[]`) kullanılan `Comparator`, dizinin sıralandığı `Comparator` ile **aynı** olmalı; farklı bir karşılaştırıcıyla arama yapmak sıralı dizide de tanımsız sonuç verir.
- Dönen değer negatifse elemanın yokluğu kesindir, ancak pozitif/sıfır bir indeks yalnızca eşit anahtarlardan **birinin** konumunu garanti eder; dizide birden fazla eşit eleman varsa hangisinin döneceği belirtilmemiştir.

## Denemek için

```
jshell> int[] a = {5, 2, 8, 1}
jshell> java.util.Arrays.binarySearch(a, 8)
jshell> java.util.Arrays.sort(a)
jshell> java.util.Arrays.binarySearch(a, 8)
```

Sıralamadan önceki ve sonraki çağrıların döndürdüğü değerleri karşılaştırmak, aynı aramanın sıralı olmayan veri üzerinde nasıl güvenilmez sonuç verdiğini gösterir.

[Kaynak: https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/Arrays.html#binarySearch(int%5B%5D,int)]
