# generic wildcard (extends / super)

`List<Number>` parametresi alan bir metot, `List<Integer>` ile çağrılamaz — generic tipler invariant'tır, yani `Integer` `Number`'ın alt tipi olsa bile `List<Integer>`, `List<Number>`'ın alt tipi sayılmaz. Wildcard (`?`), bir metodun farklı somut tip parametreleriyle gelen koleksiyonları kabul edebilmesini sağlar; `extends` ve `super` ise bu esnekliğin okuma/yazma güvenliğini belirler.

## `? extends T` ile üretici (producer)

```java
static double sum(List<? extends Number> list) {
    double total = 0;
    for (Number n : list) {
        total += n.doubleValue();
    }
    return total;
}

sum(List.of(1, 2, 3));       // List<Integer>
sum(List.of(1.5, 2.5));      // List<Double>
```

`List<? extends Number>`, bilinmeyen ama `Number`'ın bir alt tipi olan bir listeyi ifade eder. Listeden eleman okumak güvenlidir çünkü her eleman en azından `Number`'dır. Listeye eleman eklemek ise güvenli değildir — derleyici gerçek tipin `Integer` mi `Double` mı olduğunu bilmediği için `list.add(...)` çağrısına izin vermez (`null` hariç). Bu yüzden `extends` wildcard'ı **üretici (producer)** rolündeki parametrelerde kullanılır: metot listeden veri okur, listeye yazmaz.

## `? super T` ile tüketici (consumer)

```java
static void addNumbers(List<? super Integer> list) {
    list.add(1);
    list.add(2);
    list.add(3);
}

List<Number> numbers = new ArrayList<>();
addNumbers(numbers);

List<Object> objects = new ArrayList<>();
addNumbers(objects);
```

`List<? super Integer>`, `Integer`'ın bir üst tipi (kendisi dahil) olan bilinmeyen bir listeyi ifade eder. Bu listeye `Integer` eklemek güvenlidir çünkü gerçek liste ne olursa olsun (`List<Integer>`, `List<Number>`, `List<Object>`) bir `Integer`'ı kabul edebilir. Ancak listeden okunan eleman yalnızca `Object` olarak garanti edilir, çünkü gerçek tip `Integer`'dan daha genel olabilir. Bu yüzden `super` wildcard'ı **tüketici (consumer)** rolündeki parametrelerde kullanılır: metot listeye veri yazar, anlamlı şekilde okumaz.

## Dikkat edilmesi gerekenler

- Bu ayrım `PECS` kısaltmasıyla anılır: *Producer Extends, Consumer Super*.
- `? extends T` listesine `add` çağrısı (null dışında) her zaman derleme hatası verir; derleyici gerçek alt tipi bilemez.
- `? super T` listesinden `get` çağrısı yalnızca `Object` tipinde değer döner; daha spesifik bir tipe güvenle cast edilemez.
- Hem okuma hem yazma gerekiyorsa wildcard kullanılmaz, sade `List<T>` tercih edilir.

## Denemek için

```
jshell> List<? extends Number> nums = List.of(1, 2, 3)
jshell> double total = nums.stream().mapToDouble(Number::doubleValue).sum()
jshell> nums.add(4)
```

İlk iki satır sorunsuz çalışıp `total` için `6.0` üretir; üçüncü satır ise derleme hatası verir çünkü `? extends Number` listesine eleman eklenemez.

[Kaynak: https://docs.oracle.com/javase/tutorial/java/generics/wildcards.html]
