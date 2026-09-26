# Dizi Kovaryansı ve ArrayStoreException

Java'da diziler **kovaryanttır**: `String[]`, bir `Object[]` referansına atanabilir, çünkü `String` `Object`'ten türer. Bu, derleyicinin yıllar önce jenerik tipler yokken dizi kullanan genel amaçlı metotlara (örneğin `Arrays.sort(Object[])`) izin vermek için yaptığı bir tasarım tercihidir. Ancak bu esneklik, tip güvenliğini derleme zamanından çalışma zamanına iter: dizinin gerçek çalışma zamanı tipiyle uyuşmayan bir eleman atandığında program derlenir ama çalışırken patlar.

## Kovaryant atama ve runtime kontrolü

```java
Object[] objects = new String[3];
objects[0] = "merhaba"; // sorunsuz
objects[1] = 42;        // derlenir, ama ArrayStoreException fırlatır
```

`objects` değişkeninin derleme zamanı tipi `Object[]` olduğu için `objects[1] = 42` derleyici açısından geçerlidir. Fakat JVM her dizi elemanı atamasında (`aastore` bytecode talimatında) dizinin **gerçek** bileşen tipini kontrol eder; burada gerçek tip `String[]` olduğundan bir `Integer` yerleştirilmesi `ArrayStoreException` ile reddedilir. Bu, `unchecked` bir istisnadır (`RuntimeException` alt sınıfı), yani derleyici `catch` etmenizi zorlamaz.

## Generics ile karşılaştırma: invariance

```java
List<Object> list = new ArrayList<String>(); // derleme hatası
```

Jenerik koleksiyonlar tam tersine **invariant**'tır: `List<String>`, bir `List<Object>` referansına atanamaz, derleyici bunu doğrudan reddeder. Bu tasarım, dizilerin yaşadığı runtime sürprizini generics'te baştan engeller; hatanın maliyeti derleme zamanına taşınır. Bu yüzden `varargs` gibi dizi tabanlı jenerik API'lerde (`List<String>... lists`) derleyici "heap pollution" uyarısı verir: dizi kovaryansı ile generics'in tip silme (`type erasure`) davranışı bir araya gelince tip güvenliği garanti edilemez.

## Dikkat edilmesi gerekenler

- `ArrayStoreException` yalnızca referans tipli (`Object[]` alt tipi) dizilerde oluşur; ilkel tip dizileri (`int[]`, `double[]`) kovaryant değildir çünkü farklı ilkel tipler arasında böyle bir atama zaten söz konusu değildir.
- Her `aastore` çağrısında yapılan runtime tip kontrolü küçük bir performans maliyetine yol açar; `Object[]` yerine gerçek bileşen tipiyle dizi oluşturmak (örneğin doğrudan `String[]`) bu kontrolü gereksiz kılar.
- `Arrays.asList` ve benzeri eski API'lerin `Object[]` üzerine kurulu olması, kovaryansın tarihsel nedenidir; yeni kod yazarken mümkünse jenerik koleksiyonları tercih etmek bu sınıf hataları tamamen ortadan kaldırır.

## Denemek için

```
jshell> Object[] arr = new String[2]
jshell> arr[0] = "ok"
jshell> arr[1] = 5
```

Son satır `java.lang.ArrayStoreException: java.lang.Integer` fırlatır; `arr`'ın derleme zamanı tipi `Object[]` olsa da JVM'in elinde tuttuğu gerçek tip hâlâ `String[]`'tir.

[Kaynak: https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/lang/ArrayStoreException.html]
