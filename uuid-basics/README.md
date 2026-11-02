# uuid-basics

Dağıtık sistemlerde veya veritabanı anahtarlarında benzersiz bir kimlik üretmek gerektiğinde akla gelen ilk çözüm genelde artan bir sayaçtır, ama sayaç merkezi bir koordinasyon ister ve birden fazla düğüm aynı anda kimlik ürettiğinde çakışma riski taşır. `java.util.UUID`, herhangi bir merkezi koordinasyona ihtiyaç duymadan pratikte çakışmayacak kadar geniş bir kimlik uzayı sağlar.

## Rastgele UUID üretimi

```java
UUID id1 = UUID.randomUUID();
UUID id2 = UUID.randomUUID();

System.out.println(id1);              // örn: 3fa85f64-5717-4562-b3fc-2c963f66afa6
System.out.println(id1.equals(id2));  // false
System.out.println(id1.version());    // 4
```

`randomUUID()`, 128 bitlik değerin 122 bitini kriptografik olarak güçlü bir rastgele sayı üretecinden doldurur; kalan bitler `version` (4) ve `variant` alanlarına ayrılır. Bu kadar büyük bir rastgele uzayda, aynı değerin iki kez üretilme olasılığı pratikte göz ardı edilebilir düzeydedir ve bu yüzden `randomUUID()` dağıtık ortamlarda koordinasyonsuz kimlik üretimi için güvenle kullanılır.

## Ayrıştırma ve eşitlik

```java
UUID parsed = UUID.fromString("3fa85f64-5717-4562-b3fc-2c963f66afa6");
UUID same = UUID.fromString("3fa85f64-5717-4562-b3fc-2c963f66afa6");

System.out.println(parsed.equals(same));                 // true
System.out.println(parsed.getMostSignificantBits());     // üst 64 bit
System.out.println(parsed.getLeastSignificantBits());    // alt 64 bit
```

`UUID`, `equals()` ve `hashCode()`'u 128 bitlik değerin tamamı üzerinden hesaplar, dolayısıyla aynı metinsel gösterime sahip iki `UUID` nesnesi her zaman eşit sayılır ve `HashMap`/`HashSet` gibi koleksiyonlarda anahtar olarak güvenle kullanılabilir. `fromString()`, standart `8-4-4-4-12` hex formatını bekler; format bozuksa `IllegalArgumentException` fırlatır.

## Dikkat edilmesi gerekenler

- `randomUUID()` her çağrıda `SecureRandom` kullanır; sıkı döngüde milyonlarca kez çağırmak ölçülebilir bir maliyet yaratabilir.
- UUID, sıralı değildir: `ORDER BY` ile sorgulanan birincil anahtar olarak kullanıldığında B-tree indekslerinde rastgele yazma deseni yüzünden performans kaybına yol açabilir.
- `UUID.nameUUIDFromBytes()` ile versiyon 3 (MD5 tabanlı) deterministik UUID üretilebilir; aynı girdi her zaman aynı UUID'yi verir, bu `randomUUID()`'den tamamen farklı bir kullanım amacıdır.

## Denemek için

```
jshell> import java.util.UUID
jshell> UUID a = UUID.randomUUID()
jshell> UUID b = UUID.fromString(a.toString())
jshell> a.equals(b)
```

Son satır `true` döner — `toString()` ile yazdırılan metni `fromString()` ile geri ayrıştırmak, bit düzeyinde aynı değeri veren bir `UUID` nesnesi üretir.

[Kaynak: https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/UUID.html]
