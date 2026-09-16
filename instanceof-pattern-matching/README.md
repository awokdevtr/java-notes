# instanceof pattern matching

Klasik `instanceof` kontrolü sadece bir tip sorusuna cevap verir; asıl işi, yani değişkeni o tipe cast etmeyi programcıya bırakır. Bu da hemen ardından gelen `(Type) obj` cast'ini ve genellikle tekrarlanan bir değişken adını getirir. Java 16 ile gelen `pattern matching for instanceof`, kontrolü ve cast'i tek adımda birleştirip sonucu doğrudan yeni bir değişkene bağlar.

## Temel kullanım

```java
Object obj = "merhaba";

if (obj instanceof String s) {
    System.out.println(s.length());
}
```

`obj instanceof String s` ifadesi, `obj` gerçekten bir `String` ise hem `true` döner hem de `s` adlı değişkeni o `String`'e bağlar. Ayrı bir cast satırına gerek kalmaz; `s` sadece `if` bloğunun `true` dalında, yani derleyicinin tipin doğru olduğunu kanıtlayabildiği kapsamda görünür olur. Bu değişkene `pattern variable` denir.

## Kapsam ve `&&` ile daraltma

```java
Object obj = "merhaba";

if (obj instanceof String s && !s.isBlank()) {
    System.out.println(s.toUpperCase());
}
```

Derleyici `flow scoping` denen bir analiz yapar: `&&`'in sağ tarafında `s` zaten güvenle kullanılabilir, çünkü sol taraf `false` olsaydı kısa devre yüzünden oraya hiç gelinmezdi. Aynı mantık negatif kontrollerde de işler — `if (!(obj instanceof String s)) return;` yazıp `s`'i metodun geri kalanında kullanmak geçerlidir, çünkü `s`'in tanımsız olduğu tek yol zaten `return` ile bitmiştir.

## Dikkat edilmesi gerekenler

- Pattern variable, sadece derleyicinin tipin doğru olduğunu kesin olarak çıkarabildiği kapsamda kullanılabilir; bunun dışında kullanmaya çalışmak derleme hatasıdır.
- Aynı isimde bir pattern variable'ı aynı kapsamda iki kez tanımlayamazsınız; klasik değişken gölgeleme kurallarına tabidir.
- Java 21 ile `switch` ifadelerinde de aynı mekanizma `case String s ->` şeklinde kullanılabilir hale geldi; `instanceof` burada bu yaklaşımın başlangıç noktasıdır.

## Denemek için

```
jshell> Object obj = 42
jshell> if (obj instanceof Integer i) System.out.println(i * 2)
jshell> if (obj instanceof String s) System.out.println(s); else System.out.println("String degil")
```

İlk `if`'in `84` yazdırdığını, ikincisinin ise `s`'e hiç girmeden `else` dalına düştüğünü görürsünüz — `obj` bir `Integer` olduğu için `String` pattern'i eşleşmez.

[Kaynak: https://docs.oracle.com/en/java/javase/17/language/pattern-matching.html]
