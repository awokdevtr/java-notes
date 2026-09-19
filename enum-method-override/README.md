# enum-method-override

Bir `enum`'daki her sabit genelde aynı davranışı paylaşır, ama bazen her sabitin kendine özgü bir mantığı olması gerekir. Bunu `switch` ile enum sabitine göre dallanarak çözmek mümkündür, ancak yeni bir sabit eklendiğinde `switch` bloğunu güncellemeyi unutmak kolaydır. Java, her sabitin kendi metot gövdesini tanımlamasına izin vererek bu riski ortadan kaldırır.

## Sabite özgü metot gövdesi

```java
public enum Operation {
    PLUS {
        @Override
        public int apply(int a, int b) { return a + b; }
    },
    MINUS {
        @Override
        public int apply(int a, int b) { return a - b; }
    },
    TIMES {
        @Override
        public int apply(int a, int b) { return a * b; }
    };

    public abstract int apply(int a, int b);
}
```

`apply` metodu `enum` gövdesinde `abstract` olarak bildirilir; her sabit ise süslü parantez içinde kendi gövdesini vererek anonim bir alt sınıf gibi davranır. Derleyici her sabitin `apply`'ı override ettiğini zorunlu kılar, bu yüzden yeni bir sabit eklenip metodu unutulursa kod derlenmez. Bu yaklaşım, `switch (this) { ... }` yazıp her dala ayrı `case` eklemekten daha güvenlidir çünkü eksik durumu compile-time'da yakalar.

## Ortak davranışı override etmek

```java
public enum Status {
    ACTIVE, SUSPENDED {
        @Override
        public String label() { return "Askida (" + super.label() + ")"; }
    };

    public String label() { return name().toLowerCase(); }
}
```

Sadece belirli sabitler farklı davranmalıysa, ortak metot enum gövdesinde normal (soyut olmayan) şekilde tanımlanır ve yalnızca istisnai sabit kendi gövdesini vererek `super.label()` ile temel davranışı da kullanabilir. Diğer sabitler ek kod yazmadan varsayılan uygulamayı miras alır.

## Dikkat edilmesi gerekenler

- Sabite özgü gövde tanımlayan bir enum, aslında her sabit için gizli bir alt sınıf üretir; bu yüzden `Operation.PLUS.getClass()` ile `Operation.class` aynı değildir.
- Soyut metot yaklaşımı, `switch` ifadesine göre daha fazla satır gerektirir ama yeni sabit eklenirken metodun unutulmasını derleme hatasına çevirir.
- Sabitlerin kendine has alanları da olabilir; constructor her sabit için ayrı ayrı çağrılabilir.

## Denemek için

```
jshell> enum Op { PLUS { public int apply(int a,int b){return a+b;} }, MINUS { public int apply(int a,int b){return a-b;} }; public abstract int apply(int a,int b); }
jshell> Op.PLUS.apply(3, 4)
jshell> Op.MINUS.apply(3, 4)
```

İlk sabit `7`, ikinci sabit `-1` döndürür; her ikisi de aynı `apply` imzasını taşıdığı halde farklı davranır çünkü her `enum` sabiti kendi override edilmiş gövdesini çalıştırır.

[Kaynak: https://docs.oracle.com/javase/specs/jls/se21/html/jls-8.html#jls-8.9.1]
