# effectively-final-lambda-capture

Bir lambda ya da anonim sınıf, çevresindeki metottan yerel bir değişkeni kullandığında, o değişkeni kendi bünyesine kopyalar; metot çağrısı bittiğinde orijinal stack frame yok olsa da lambda hâlâ geçerli bir değer görür. Bu tutarlılığı garanti edebilmek için derleyici, yakalanan değişkenin `effectively final` olmasını, yani bir kez atandıktan sonra bir daha değiştirilmemesini zorunlu kılar.

## Derleme hatası veren durum

```java
int total = 0;
for (int i = 0; i < 5; i++) {
    total += i;
}

Runnable r = () -> System.out.println(total); // hata: total effectively final degil
total = 100;
```

Burada `total`, lambda tanımlandıktan sonra tekrar atama aldığı için `effectively final` sayılmaz ve derleyici `local variables referenced from a lambda expression must be final or effectively final` hatası verir. Değişken `final` anahtar kelimesiyle işaretlenmemiş olsa da, koddaki tek bir atamadan sonra bir daha değişmiyorsa derleyici onu örtük olarak final kabul eder ve yakalamaya izin verir.

## Yaygın tuzak: döngü sayacını biriktirme

```java
List<Runnable> tasks = new ArrayList<>();
for (int i = 0; i < 3; i++) {
    int current = i; // her iterasyonda yeni, kendi basina effectively final bir degisken
    tasks.add(() -> System.out.println("gorev " + current));
}
tasks.forEach(Runnable::run); // gorev 0, gorev 1, gorev 2
```

Döngü değişkeni `i` doğrudan yakalanamaz çünkü her iterasyonda değer değiştirir; bunun yerine döngü gövdesinde yeni bir `current` değişkeni oluşturulur. Bu değişken her iterasyonda yalnızca bir kez atandığından her lambda kendi anlık değerini güvenle saklar, aksi halde tüm görevler aynı paylaşılan sayacı görüp hepsi son değeri yazdırırdı.

## Dikkat edilmesi gerekenler

- Kısıtlama yalnızca yerel değişkenler ve metot parametreleri için geçerlidir; instance alanları ve static alanlar bu kurala tabi değildir çünkü onlar stack frame'e değil nesneye veya sınıfa bağlıdır.
- Mutasyona ihtiyaç varsa, yakalanan ilkel bir sayaç yerine `AtomicInteger` veya tek elemanlı bir dizi gibi referans tipi değişmeyen bir kap kullanılabilir; kabın içeriği değişse de referansın kendisi effectively final kalır.
- Aynı kural anonim sınıflar için de geçerlidir, çünkü Java bu kısıtlamayı lambda'larla birlikte anonim sınıflara da genişletmiştir (öncesinde sadece `final` zorunluydu).

## Denemek için

```
jshell> int[] counter = {0}
jshell> Runnable inc = () -> counter[0]++
jshell> inc.run(); inc.run(); System.out.println(counter[0])
```

Dizinin kendi referansı hiç değişmediği için `counter` effectively final kalır, ama dizinin içeriği lambda çalıştıkça serbestçe güncellenebilir.

[Kaynak: https://docs.oracle.com/javase/tutorial/java/javaOO/lambdaexpressions.html]
