# switch expression

Klasik `switch` deyimi (`statement`) yıllardır aynı tuzağı taşır: `break` unutulursa bir case diğerine "düşer" (`fall-through`), her dal ayrı bir `case ... :` bloğu gerektirir ve deyim bir değer üretmez — sonucu değişkene atamak için önce değişkeni tanımlayıp sonra her dalda ona atama yapmak gerekir. Java 14 ile kalıcı hale gelen `switch expression`, `->` sözdizimiyle hem fall-through'u ortadan kaldırır hem de doğrudan bir değer döndürebilir.

## Değer döndüren switch

```java
int gunSayisi = switch (ay) {
    case 4, 6, 9, 11 -> 30;
    case 2 -> 28;
    default -> 31;
};
```

`->` ok işaretiyle yazılan her dal yalnızca kendi ifadesini çalıştırır, bir sonraki case'e düşmez; bu yüzden `break` yazmaya gerek kalmaz. Aynı sonucu üreten birden fazla değer `case 4, 6, 9, 11 ->` gibi virgülle tek satırda gruplanabilir. `switch` burada bir deyim değil bir `expression` olduğu için sonucu doğrudan `gunSayisi` değişkenine atanabilir; derleyici tüm olası dalların kapsandığından (`exhaustiveness`) emin olmak ister, bu yüzden `enum` dışındaki tiplerde `default` dalı genelde zorunludur.

## Blok gövde ve yield

```java
String kategori = switch (puan / 10) {
    case 10, 9 -> "A";
    case 8, 7 -> {
        System.out.println("iyi aralik");
        yield "B";
    }
    default -> "C";
};
```

Bir dalın birden fazla ifadeye ihtiyacı varsa `{}` ile blok gövde yazılabilir; bu durumda bloğun ürettiği değer `yield` anahtar kelimesiyle belirtilir. `yield`, metottan çıkan `return`'ün aksine yalnızca çevreleyen `switch expression`'ı sonlandırır ve onun değerini belirler.

## Dikkat edilmesi gerekenler

- Klasik `case X:` sözdizimi hâlâ geçerlidir ve fall-through davranışı sürer; `->` ile karıştırmamak gerekir, ikisi aynı `switch` bloğunda birlikte kullanılamaz.
- `expression` formunda derleyici tüm dalların kapsanmasını (`exhaustive`) zorunlu kılar; `default` eksikse ve olası tüm değerler kapsanmamışsa derleme hatası alınır.
- `sealed` bir tip hiyerarşisiyle birlikte kullanıldığında `default` dalına gerek kalmadan tüm alt tipler tek tek kapsanabilir.

## Denemek için

```
jshell> int ay = 4
jshell> int gun = switch (ay) { case 4, 6, 9, 11 -> 30; case 2 -> 28; default -> 31; }
jshell> gun
```

`gun` değişkeninin `30` olduğunu görürsünüz; klasik `switch` deyiminde aynı sonucu almak için önce `int gun` tanımlayıp her `case` içine ayrı ayrı atama ve `break` yazmanız gerekirdi.

[Kaynak: https://docs.oracle.com/en/java/javase/17/language/switch-expressions.html]
