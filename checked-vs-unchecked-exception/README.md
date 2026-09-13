# checked vs unchecked exception

Java'da istisnalar iki kümeye ayrılır ve bu ayrım sadece isimlendirme meselesi değildir: derleyici, `checked exception` fırlatabilecek bir kodu `try-catch` ya da `throws` ile ele almadığınız sürece derlemeyi reddeder. `unchecked exception` için böyle bir zorunluluk yoktur. Bu farkı bilmemek, ya gereksiz yere her yere `throws Exception` eklemeye ya da programlama hatalarını (`bug`) iş kurallarıymış gibi yakalamaya götürür.

## Checked exception: derleyicinin zorladığı sözleşme

```java
public void readConfig(String path) throws IOException {
    var content = Files.readString(Path.of(path));
    System.out.println(content);
}
```

`IOException`, `java.lang.Exception`'ın alt sınıfıdır ama `RuntimeException`'dan türemez; bu yüzden `checked` sayılır. `Files.readString` bu istisnayı fırlatabildiği için, onu çağıran metot ya `catch` ile yakalamalı ya da kendi imzasına `throws IOException` eklemelidir. Derleyici bu kontrolü yapar çünkü dosya sisteminin kullanılamaz olması, çağıranın önceden bilip önlem alması gereken, programın kontrolü dışındaki bir durumdur.

## Unchecked exception: programcı hatasının işareti

```java
public int firstElement(List<Integer> values) {
    return values.get(0);
}
```

Liste boşsa bu metot `IndexOutOfBoundsException` fırlatır; bu sınıf `RuntimeException`'ın alt sınıfıdır ve derleyici hiçbir `throws` bildirimi talep etmez. `NullPointerException`, `IllegalArgumentException`, `ArithmeticException` gibi tüm `RuntimeException` alt sınıfları aynı kategoridedir: bunlar genellikle çağıranın kodu yanlış kullandığının, dış koşulların değil mantık hatasının göstergesidir. `Error` ve alt sınıfları (`OutOfMemoryError` gibi) da unchecked'tır ama bunlar tipik olarak yakalanıp kurtarılabilecek durumlar değildir.

## Dikkat edilmesi gerekenler

- Bir istisnayı sadece derleyiciyi susturmak için boş `catch` bloğuyla yutmak, hatanın nedenini gizler; en azından loglamak gerekir.
- Kendi istisna sınıfınızı tasarlarken `Exception`'dan mı yoksa `RuntimeException`'dan mı türeteceğinize, çağıranın bu durumdan kurtulup kurtulamayacağına bakarak karar verin.
- `throws Exception` ile genel bir bildirim yapmak, hangi somut hataların oluşabileceği bilgisini gizler ve çağıranı gereksiz yere genişletir.

## Denemek için

```
jshell> void bad() { throw new IllegalStateException("kirli durum"); }
jshell> bad()
```

`IllegalStateException`'ın `RuntimeException` alt sınıfı olduğu için `bad()` metodunun `throws` bildirimi olmadan derlendiğini, ama yine de çalışma zamanında fırlatıldığını görürsünüz. Aynı denemeyi `throw new java.io.IOException("io hatası")` ile tekrarlarsanız, metodun gövdesi tek başına derlenmez; `throws IOException` eklemeniz gerekir.

[Kaynak: https://docs.oracle.com/javase/tutorial/essential/exceptions/runtime.html]
