# java.time API ile tarih ve zaman

Eski `java.util.Date` ve `Calendar` sınıfları mutable'dır, ay indeksi sıfırdan başlar ve thread-safe değildir; bir referansı paylaşan iki thread aynı `Calendar` nesnesini aynı anda değiştirirse tutarsız sonuçlar ortaya çıkar. Java 8 ile gelen `java.time` paketi bu sınıfların yerine immutable, thread-safe ve okunması net bir API sunar.

## Tarih ve saat: LocalDate, LocalDateTime, Instant

```java
LocalDate today = LocalDate.now();
LocalDate birthday = LocalDate.of(1995, Month.MARCH, 14);

LocalDateTime meeting = LocalDateTime.of(2026, 9, 21, 14, 30);
Instant timestamp = Instant.now();

LocalDate nextWeek = today.plusWeeks(1);
```

`LocalDate` sadece tarihi (yıl-ay-gün), `LocalDateTime` tarih ve saati birlikte, `Instant` ise UTC epoch'tan bu yana geçen zamanı zaman dilimi bilgisi olmadan tutar. Bu üç tip de immutable'dır: `plusWeeks` gibi metotlar mevcut nesneyi değiştirmez, her zaman yeni bir nesne döndürür. Ay artık `Month` enum'ı veya 1-12 arası bir sayı ile verilir, `Calendar`'daki sıfır tabanlı ay tuzağı yoktur.

## Süre hesaplama: Duration ve Period

```java
LocalDate start = LocalDate.of(2026, 1, 10);
LocalDate end = LocalDate.of(2026, 9, 21);
Period period = Period.between(start, end);
System.out.println(period.getMonths() + " ay, " + period.getDays() + " gün");

Duration duration = Duration.ofMinutes(90);
LocalDateTime finish = meeting.plus(duration);
```

`Period` gün/ay/yıl bazında insan takvimine göre farkı ifade ederken, `Duration` saniye/nanosaniye bazında ölçülebilir bir zaman aralığını temsil eder. İkisi de immutable'dır ve `between`, `of` gibi statik fabrika metotlarıyla oluşturulur; hiçbir zaman `new Period()` veya `new Duration()` çağrılmaz.

## Dikkat edilmesi gerekenler

- `LocalDateTime` zaman dilimi (timezone) taşımaz; farklı bölgelerdeki kullanıcılar için `ZonedDateTime` veya `OffsetDateTime` kullanılmalıdır.
- `Instant` ile `LocalDateTime` doğrudan karşılaştırılamaz; dönüşüm için `ZoneId` üzerinden `atZone()` veya `toInstant()` gerekir.
- Eski `Date`/`Calendar` API'siyle etkileşim gerekiyorsa `Date.from(Instant)` ve `date.toInstant()` köprü metotları kullanılır; iki API'yi karıştırıp elle alan kopyalamaktan kaçının.

## Denemek için

```
jshell> import java.time.*
jshell> var d1 = LocalDate.of(2026, 1, 1)
jshell> var d2 = LocalDate.now()
jshell> Period.between(d1, d2)
```

Çıktıda `P` ile başlayan ISO-8601 period formatını (`P8M20D` gibi) gözlemleyin.

[Kaynak: https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/time/package-summary.html]
