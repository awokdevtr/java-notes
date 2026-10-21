# StringJoiner

Bir koleksiyondaki elemanları virgülle ayırıp köşeli parantez içine almak gibi basit bir biçimlendirme işini `StringBuilder` ile yapmak, döngü içinde "son elemandan sonra ayraç koyma" gibi kenar durumlarını elle yönetmeyi gerektirir. `java.util.StringJoiner`, ayraç (`delimiter`), önek (`prefix`) ve sonek (`suffix`) mantığını kendi içinde taşıyarak bu tekrar eden kod kalıbını ortadan kaldırır.

## Temel kullanım

```java
StringJoiner joiner = new StringJoiner(", ", "[", "]");
joiner.add("elma").add("armut").add("muz");
System.out.println(joiner.toString());
```

Bu kod `[elma, armut, muz]` çıktısını verir. `add()` her çağrıldığında, ilk eleman dışındakilerin önüne otomatik olarak `delimiter` eklenir; `prefix` ve `suffix` ise sadece `toString()` çağrıldığında bir kez eklenir. Eleman sayısı ve ayraç yerleşimiyle ilgili tüm hesap `StringJoiner` nesnesinin içinde tutulur, çağıran kod sadece `add()` çağırır.

## Boş durum ve `setEmptyValue`

```java
StringJoiner empty = new StringJoiner(", ", "[", "]");
empty.setEmptyValue("(bos)");
System.out.println(empty.toString());
```

Hiç `add()` çağrılmadan `toString()` çağrılırsa normalde sadece `prefix` + `suffix` (yani `[]`) döner. `setEmptyValue()` ile bu varsayılan davranış değiştirilebilir; yukarıdaki örnek `(bos)` yazdırır. Bu, boş bir listeyi kullanıcıya göstermeden önce özel bir mesajla değiştirmek istediğinizde kullanışlıdır.

## Ne zaman işe yarar

- `String.join()` sadece sabit bir ayraçla birleştirme yapar; `prefix`/`suffix` veya aşamalı `add()` çağrısı gerekiyorsa `StringJoiner` daha uygundur.
- `Collectors.joining(delimiter, prefix, suffix)` aslında arka planda `StringJoiner` kullanır; stream dışında elle biriktirme yapıyorsanız doğrudan `StringJoiner`'ı tercih edin.
- İki `StringJoiner`'ı `merge()` ile birleştirip tek bir sonuç elde edebilirsiniz; bu, alt gruplardan oluşan sonuçları tek bir biçimde toplamak için kullanışlıdır.

## Denemek için

```
jshell> var j = new java.util.StringJoiner(" - ", "<", ">")
jshell> j.add("a").add("b").add("c")
jshell> j.toString()
```

Çıktının `<a - b - c>` olduğunu, her `add()` çağrısının zincirlenebilir olduğunu (kendi referansını döndürdüğünü) göreceksiniz.

[Kaynak: https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/StringJoiner.html]
