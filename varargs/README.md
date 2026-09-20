# varargs

Bir metoda kaç tane argüman geleceği önceden bilinmediğinde klasik çözüm aşırı yüklenmiş (`overload`) metotlar yazmak ya da çağrı tarafında elle bir dizi oluşturmaktı. `varargs` (variable arguments), parametre listesinin sonuna `...` eklenerek bu ihtiyacı derleyici seviyesinde çözer: metot, sıfır ya da daha fazla argümanı tek bir dizi gibi kabul eder.

## Temel kullanım

```java
static int sum(int... numbers) {
    int total = 0;
    for (int n : numbers) {
        total += n;
    }
    return total;
}

sum();          // 0
sum(1, 2);      // 3
sum(1, 2, 3, 4); // 10
```

`int... numbers` bildirimi derleme zamanında `int[] numbers`'a dönüştürülür; metot gövdesi içinde `numbers` sıradan bir dizidir. Çağrı tarafında argümanlar virgülle ayrılmış tek tek değerler olarak yazılabildiği gibi, doğrudan bir `int[]` de geçirilebilir. Bir metotta en fazla bir `varargs` parametresi olabilir ve bu parametre her zaman listenin son sırasında yer almalıdır.

## Overload çözümlemesiyle etkileşim

```java
static void print(String s) { System.out.println("sabit: " + s); }
static void print(String... s) { System.out.println("varargs: " + s.length); }

print("a"); // "sabit: a" basılır
```

Derleyici, `varargs` içermeyen tam eşleşen bir `overload` bulunduğunda onu her zaman `varargs` versiyonuna tercih eder. Bu yüzden `varargs` kabul eden bir metotla birlikte aynı isimde sabit parametreli bir `overload` tanımlamak, hangi çağrının hangisine gittiğini karıştırmaya çok açıktır ve genelde kaçınılması gereken bir tasarımdır.

## Dikkat edilmesi gerekenler

- `varargs` parametresi çağrı tarafında `null` olarak geçilebilir (`sum((int[]) null)`), bu da dizi üzerinde döngüye girildiğinde `NullPointerException` fırlatır; sıfır argümanlı çağrı (`sum()`) ise boş bir dizi üretir ve güvenlidir.
- Her çağrıda arka planda yeni bir dizi tahsis edilir; performansa duyarlı, sık çağrılan kod yollarında bu ek maliyet göz ardı edilmemelidir.
- İlkel olmayan tipte (`Object...`) bir `varargs` parametresine tek bir dizi geçirilirse, derleyici bunu yeni bir dizi ile sarmalamaz; doğrudan o dizi kullanılır — bu da `printf`-tarzı metotlarda beklenmedik `ClassCastException` ya da yanlış eleman sayısına yol açabilir.

## Denemek için

```
jshell> static int sum(int... n) { int t = 0; for (int x : n) t += x; return t; }
jshell> sum(1, 2, 3)
jshell> sum()
jshell> sum(new int[]{5, 10})
```

Son satırda doğrudan bir diziyi de argüman olarak geçirebildiğinizi, sonucun tek tek değer verildiğindekiyle aynı şekilde çalıştığını göreceksiniz.

[Kaynak: https://docs.oracle.com/javase/tutorial/java/javaOO/arguments.html]
