# Local Classes

Java'da bir sınıfı sadece tek bir metodun gövdesi içinde tanımlamak mümkündür. Bu "local class" (yerel sınıf), üye iç sınıflardan (`inner class`) ve statik iç sınıflardan farklı olarak dışarıya hiç görünmez; sadece tanımlandığı blok içinde kullanılabilir. Bir yardımcı tip sadece bir metodun uyguladığı algoritma için gerekliyse, onu sınıfın genel alanına taşımak yerine doğrudan kullanıldığı yere gömmek, kapsamı en sıkı haliyle sınırlar.

## Metot içinde tanımlama ve kapsam

```java
import java.util.List;

class ReportBuilder {
    List<String> build(List<Integer> scores, int passLimit) {
        class ScoreLine {
            final int score;
            final boolean passed;

            ScoreLine(int score) {
                this.score = score;
                this.passed = score >= passLimit;
            }

            @Override
            public String toString() {
                return score + " -> " + (passed ? "PASS" : "FAIL");
            }
        }

        return scores.stream()
                .map(ScoreLine::new)
                .map(Object::toString)
                .toList();
    }
}
```

`ScoreLine`, sadece `build` metodunun içinde var olur; metodun dışından `ReportBuilder.ScoreLine` diye erişilemez ve bu isim derleme biriminin başka hiçbir yerinde görünmez. `passLimit` parametresi burada `effectively final` olduğu için `ScoreLine`'ın constructor'ı tarafından doğrudan okunabilir; yerel sınıf, çevreleyen metottaki `effectively final` değişkenleri kendi kopyası üzerinden yakalar, tıpkı bir lambda'nın yaptığı gibi.

## Anonim sınıfla farkı

```java
Runnable namedTask;
{
    class LoggingTask implements Runnable {
        public void run() {
            System.out.println("calisiyor: " + getClass().getSimpleName());
        }
    }
    namedTask = new LoggingTask();
    namedTask = new LoggingTask(); // ayni tipten ikinci bir ornek
}
```

Anonim bir sınıf (`new Runnable() { ... }`) adsızdır ve yalnızca bir kez örneklenebilir; yerel sınıf ise bir isme sahiptir, bu yüzden aynı blok içinde birden fazla örneği oluşturulabilir, `implements`/`extends` ile birden fazla kurucu tanımlayabilir ve kendi statik olmayan yardımcı metotlarına sahip olabilir. İhtiyaç sadece tek bir arayüzün tek seferlik bir uygulamasıysa anonim sınıf veya lambda yeterlidir; birden fazla örnek, kurucu overload'u veya ek metotlar gerekiyorsa yerel sınıf daha uygundur.

## Ne zaman işe yarar

- Bir yardımcı tip sadece bir metodun iç mantığında kullanılıyorsa ve dışarıya sızdırılması anlamsızsa.
- Anonim sınıfın tek kurucu/tek metot sınırlaması yetersiz kaldığında (birden fazla constructor, ekstra alan veya ekstra public metot gerektiğinde).
- Yerel sınıf da `static` değilse çevreleyen örneğe (`this` referansına) örtük bir bağ taşır; bu bağ gereksiz yere nesneyi canlı tutabileceğinden, çevreleyen durumuna ihtiyacı yoksa yerel sınıfı `static` yapmak (Java 16+) tercih edilmelidir.

## Denemek için

```
jshell> class Outer {
   ...>   void run() {
   ...>     class Counter { int n = 0; int next() { return ++n; } }
   ...>     Counter c = new Counter();
   ...>     System.out.println(c.next() + "," + c.next());
   ...>   }
   ...> }
jshell> new Outer().run()
```

`Counter` sınıfı sadece `run()` metodunun içinde tanımlıdır; `jshell`'de `run()` dışında `new Counter()` yazılmaya çalışıldığında derleyici bu ismi tanımadığını bildirir.

[Kaynak: https://docs.oracle.com/javase/tutorial/java/javaOO/localclasses.html]
