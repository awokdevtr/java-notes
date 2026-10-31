# floorDiv ve floorMod ile Negatif Sayılarda Bölme Farkı

Java'nın `/` ve `%` operatörleri, negatif operandlarda matematikteki "kat bölme" (floored division) değil, sıfıra yuvarlayan (`truncated`) bölme yapar. Bu yüzden `-7 / 2` sonucu `-3` değil `-4` beklenirken `-3` çıkar ve `%` işleci işaretini bölünenden (`dividend`) alır. Takvim hesapları, dizi indeksleme veya döngüsel (`wrap-around`) mantık gibi negatif değerlerin de aktığı yerlerde bu fark sessizce yanlış sonuca götürür.

## `/` ve `%` Operatörlerinin Sıfıra Yuvarlaması

```java
System.out.println(-7 / 2);   // -3  (sıfıra doğru yuvarlar)
System.out.println(-7 % 2);   // -1  (işaret bölünenden gelir)
```

`-7 / 2` matematiksel olarak `-3.5`'tir; Java bunu sıfıra en yakın tam sayıya (`-3`) yuvarlar, aşağıya (`-4`) değil. `%` ise `a - (a / b) * b` tanımına sadık kalır, bu yüzden sonucun işareti her zaman bölünenle aynı olur, bölenle değil.

## `Math.floorDiv` ve `Math.floorMod` ile Kat Bölme

```java
System.out.println(Math.floorDiv(-7, 2));   // -4
System.out.println(Math.floorMod(-7, 2));   // 1
```

`Math.floorDiv`, sonucu negatif sonsuza doğru yuvarlar; yani bölüm her zaman gerçek matematiksel değerin "altında" kalır. `Math.floorMod` bu bölüme eşlik eder ve sonucun işaretini her zaman bölenle (`divisor`) aynı tutar — pozitif bölende sonuç hiçbir zaman negatif olmaz. Bu, saat dilimi hesapları veya dairesel dizi indeksleme (`index = Math.floorMod(i, size)`) gibi senaryolarda `%`'nin negatif sonuç üretme riskini ortadan kaldırır.

## Dikkat edilmesi gerekenler

- Dairesel indeksleme yaparken `i % size` yerine `Math.floorMod(i, size)` kullanmak, `i` negatif olduğunda `ArrayIndexOutOfBoundsException` riskini önler.
- `floorDiv`/`floorMod` çifti her zaman `a == floorDiv(a, b) * b + floorMod(a, b)` özdeşliğini korur; `/` ve `%` çifti de kendi tanımıyla tutarlıdır, sadece negatifte farklı bir sözleşmedir.
- Bölenin pozitif olmadığı durumlarda `floorMod`'un işareti de bölenle birlikte değişir; sonucu "her zaman pozitif" sanmak hataya yol açar.

## Denemek için

```
jshell> -7 / 2
jshell> -7 % 2
jshell> Math.floorDiv(-7, 2)
jshell> Math.floorMod(-7, 2)
```

İlk iki satır operatörlerin sıfıra yuvarlayan davranışını, son iki satır `Math.floorDiv`/`floorMod`'un negatif sonsuza yuvarlayan ve işareti bölenden alan davranışını gösterir.

[Kaynak: https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/lang/Math.html#floorDiv(int,int)]
