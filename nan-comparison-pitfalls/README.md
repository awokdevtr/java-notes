# NaN Karşılaştırma Tuzağı

IEEE 754 standardına göre `NaN` (Not a Number), kendisiyle bile eşit kabul edilmeyen tek kayan nokta değeridir. Bu kural `==` operatörüyle yapılan karşılaştırmaları ve koşullu akışları sessizce bozar; bir sonucun geçersiz (`NaN`) olduğu fark edilmeden döngüler sonsuza gider veya filtre koşulları beklenmedik elemanları kaçırır.

## `==` neden yanılıyor

```java
double x = 0.0 / 0.0;
System.out.println(x == x);          // false
System.out.println(x == Double.NaN); // false
System.out.println(Double.isNaN(x)); // true
```

`0.0 / 0.0`, `Math.sqrt(-1)` gibi tanımsız işlemler `double`/`float` türünde `NaN` üretir ve IEEE 754 kuralına göre `NaN`, kendisi dahil hiçbir değere `==` ile eşit sayılmaz. Bu yüzden `x == x` bile `false` döner ve bir değerin `NaN` olup olmadığını `==` ile test etmek mümkün değildir; tek doğru yol `Double.isNaN()` veya `Float.isNaN()` statik metotlarıdır.

## `equals()` ve koleksiyonlardaki ters davranış

```java
Double a = Double.NaN;
Double b = Double.NaN;
System.out.println(a.equals(b));           // true
System.out.println(Double.compare(a, b));  // 0

Set<Double> set = new HashSet<>();
set.add(Double.NaN);
System.out.println(set.contains(Double.NaN)); // true
```

`Double.equals()` ve `Double.compare()`, IEEE 754 karşılaştırma kuralını değil bit düzeyinde eşitliği esas alır; bu yüzden kutulanmış (`boxed`) `Double` nesneleri arasında `NaN.equals(NaN)` `true` sonucu verir ve `HashMap`/`HashSet` gibi koleksiyonlarda `NaN` normal bir anahtar gibi çalışır. `Double.compare()` ayrıca `NaN`'ı her zaman diğer tüm değerlerden (pozitif sonsuzluk dahil) büyük sayar, bu da `Arrays.sort()` ile sıralanan dizilerde `NaN` değerlerinin sona yığılmasına yol açar.

## Dikkat edilmesi gerekenler

- `list.stream().filter(d -> d != someNaNValue)` gibi bir filtre, `NaN` değerlerini elemeyi başaramaz; `NaN` içeren veriyi ayıklamak için her zaman `Double.isNaN()` kullanılmalıdır.
- `Collections.max()`/`min()` bir dizide `NaN` varsa sıralama sırasına bağlı tutarsız sonuçlar üretebilir; toplama ve karşılaştırma öncesi `NaN` ayıklanmalıdır.
- `Double.NaN == Double.NaN` derleyici tarafından sabit ifade gibi görünse de çalışma zamanında yine `false` döner; statik analiz araçları genellikle bu satırı uyarı olarak işaretler.

## Denemek için

```
jshell> double x = 0.0 / 0.0
jshell> System.out.println(x == x)
jshell> System.out.println(Double.isNaN(x))
```

İlk karşılaştırma `false`, ikincisi `true` yazdırır; aradaki fark `==`'in IEEE 754 bit karşılaştırması yapması, `isNaN()`'ın ise özel bit kalıbını doğrudan tanımasıdır.

[Kaynak: https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/lang/Double.html#isNaN()]
