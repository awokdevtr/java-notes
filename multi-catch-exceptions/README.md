# Multi-catch ile istisna yakalama

Birbiriyle ilişkisiz iki istisna türüne aynı tepkiyi vermek gerektiğinde, Java 7 öncesinde ya `Exception`'ı yakalayıp gereksiz yere geniş bir ağ atmak ya da neredeyse birebir aynı gövdeyi iki ayrı `catch` bloğunda tekrar etmek gerekiyordu. `catch (IOException | SQLException e)` sözdizimi, bu tekrarı ortadan kaldırıp yakalanan türleri açıkça sınırlar.

## Tek blokta birden fazla tür yakalama

```java
try {
    riskyIoOperation();
    riskyDbOperation();
} catch (IOException | SQLException e) {
    logger.error("İşlem başarısız: " + e.getMessage(), e);
    throw new ServiceException(e);
} finally {
    cleanup();
}
```

Derleyici, `e` değişkenine iki türün en dar ortak üst sınıfını değil, `IOException | SQLException` birleşimini örtük tip olarak atar; bu yüzden `e` üzerinde yalnızca her iki türde de ortak olan metotlar (`getMessage()`, `printStackTrace()` gibi) çağrılabilir. `e` değişkeni `final`dir ve blok içinde yeniden atanamaz — bu, tek bir `catch` içinde hangi türün gerçekten yakalandığının belirsiz kalmasını önler.

## Birbirinin alt sınıfı olan türleri birleştirmenin derleme hatası olması

```java
try {
    parse();
} catch (NumberFormatException | IllegalArgumentException e) {
    // derleme hatası: NumberFormatException zaten IllegalArgumentException'ın alt sınıfı
}
```

`NumberFormatException`, `IllegalArgumentException`'ı extend ettiği için ikisini aynı multi-catch bloğunda birleştirmek gereksiz ve derleyici tarafından reddedilir; alternatiflerin gerçekten bağımsız dallar olması gerekir. Bu kısıtlama, `catch (Exception | RuntimeException e)` gibi anlamsız birleşimleri de daha yazım aşamasında engeller.

## Dikkat edilmesi gerekenler

- Bytecode seviyesinde multi-catch, aynı `catch` gövdesini her tür için ayrı ayrı üretmek yerine tek bir handler kullanır; bu yüzden ayrı `catch` bloklarına göre sınıf dosyası boyutunu küçültür.
- Derlenmiş `.class` dosyasında `e` değişkeni, birleşimdeki türlerin en dar ortak üst sınıfıyla temsil edilir; reflection ile bakıldığında gerçek çalışma zamanı tipi yine fırlatılan istisnanın kendisidir.
- Farklı türler için farklı loglama seviyesi veya farklı kurtarma adımı gerekiyorsa multi-catch uygun değildir — bu durumda ayrı `catch` blokları tercih edilmelidir.

## Denemek için

```
jshell> void risky(int x) throws java.io.IOException { if (x == 0) throw new java.io.IOException("io"); else throw new NumberFormatException("nfe"); }
jshell> try { risky(0); } catch (java.io.IOException | NumberFormatException e) { System.out.println(e.getClass().getSimpleName()); }
jshell> try { risky(1); } catch (java.io.IOException | NumberFormatException e) { System.out.println(e.getClass().getSimpleName()); }
```

İki çağrı da aynı `catch` bloğuna düşer ama `e.getClass()` her seferinde fırlatılan gerçek istisna türünü (`IOException` veya `NumberFormatException`) döner.

[Kaynak: https://docs.oracle.com/javase/tutorial/essential/exceptions/multicatch.html]
