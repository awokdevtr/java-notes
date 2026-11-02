# assert anahtar kelimesi

Geliştirme sırasında bir metodun asla gerçekleşmemesi gereken bir duruma düştüğünü kontrol etmek isteriz, ama bu kontrolün üretimde çalışan koda her zaman ek maliyet ve karmaşıklık getirmesini istemeyiz. `if` ile elle yazılan hata fırlatma kodu kalıcıdır ve kapatılamaz. Java'nın `assert` anahtar kelimesi, bu tür iç tutarlılık kontrollerini varsayılan olarak devre dışı, istenince JVM bayrağıyla açılabilen bir mekanizmaya dönüştürür.

## Temel kullanım

```java
void withdraw(int amount) {
    assert amount > 0 : "Miktar pozitif olmalı: " + amount;
    balance -= amount;
}
```

`assert` ifadesindeki koşul `false` olduğunda `AssertionError` fırlatılır; `:`'den sonraki ifade hata mesajına dönüşür. Kritik nokta: bu kontrol varsayılan olarak çalışmaz. JVM `assert` deyimlerini derler ama `-ea` (`-enableassertions`) bayrağı verilmeden çalıştırıldığında tüm `assert` satırları no-op gibi davranır, hiçbir koşul değerlendirilmez.

## Etkinleştirme ve kapsam

```java
public static void main(String[] args) {
    assert args.length > 0 : "En az bir argüman bekleniyor";
    System.out.println(args[0]);
}
```

Bu kodu `java Main` ile çalıştırırsanız boş `args` dizisi hiçbir hata vermez; `assert` satırı tamamen atlanır. `java -ea Main` ile çalıştırdığınızda ise koşul gerçekten değerlendirilir ve hata fırlatılır. `-ea:com.example...` gibi paket bazlı etkinleştirme de mümkündür, böylece sadece belirli modüller test edilirken açılabilir.

## Dikkat edilmesi gerekenler

- `assert` içindeki ifade yan etki üretmemelidir (örneğin bir listeyi değiştirmemelidir); çünkü etkinleştirilmemiş haliyle bu kod hiç çalışmaz ve davranış `-ea` bayrağına göre değişir.
- `assert`, dışarıdan gelen girdiyi (kullanıcı girişi, API parametresi) doğrulamak için kullanılmamalıdır; bunun için her zaman çalışan açık istisna fırlatma (`IllegalArgumentException` vb.) gerekir.
- Üretim ortamında `assert`'ler genellikle kapalı çalışır, dolayısıyla bunlara dayanan hiçbir iş mantığı güvenilir olamaz; bu özellik sadece geliştirici varsayımlarını belgelemek ve test sırasında doğrulamak içindir.

## Denemek için

```
jshell> boolean b = true; assert !b : "b true olmamalı";
jshell> jshell --execution local -R-ea
jshell> assert 1 > 2 : "bu asla olmamalı";
```

jshell varsayılan olarak assertion'ları etkin çalıştırır; ilk komutta `b` `true` olduğundan `AssertionError` fırlatıldığını, `-ea` olmadan başlatılan sıradan bir `java` çalıştırmasında ise aynı kontrolün sessizce atlanacağını karşılaştırarak görebilirsiniz.

[Kaynak: https://docs.oracle.com/javase/8/docs/technotes/guides/language/assert.html]
