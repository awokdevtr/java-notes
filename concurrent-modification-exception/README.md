# ConcurrentModificationException

`ArrayList`, `HashMap` gibi `java.util` koleksiyonları üzerinde `for-each` ile gezinirken koleksiyonun yapısını (eleman ekleme/çıkarma) doğrudan değiştirmek, çoğu zaman anında fark edilmeyen ama çalışma zamanında patlayan bir hataya yol açar. Bunun nedeni koleksiyonların *fail-fast* iterator kullanmasıdır: iterator, koleksiyonun her yapısal değişiklikte artan bir `modCount` sayacını kendi başlangıç değeriyle karşılaştırır ve uyuşmazlık gördüğü an `ConcurrentModificationException` fırlatır.

## Hatanın oluşumu

```java
List<String> names = new ArrayList<>(List.of("ali", "veli", "ayse"));

for (String name : names) {
    if (name.equals("veli")) {
        names.remove(name); // ConcurrentModificationException
    }
}
```

Burada `for-each` arka planda bir `Iterator` kullanır; `names.remove(name)` çağrısı koleksiyonu iterator'dan bağımsız olarak değiştirir ve `modCount`'u artırır. Bir sonraki `next()` çağrısında iterator bu tutarsızlığı fark eder ve istisna fırlatır. Önemli olan nokta, bu davranışın garanti edilmediği, sadece *best-effort* bir tespit olduğudur; bazı durumlarda hata sessizce atlanabilir.

## Doğru kaldırma yöntemleri

```java
Iterator<String> it = names.iterator();
while (it.hasNext()) {
    if (it.next().equals("veli")) {
        it.remove(); // guvenli: modCount'u iterator kendisi gunceller
    }
}

names.removeIf(name -> name.equals("ayse")); // tek satirlik alternatif
```

`Iterator.remove()`, koleksiyonu değiştirdiği anda iterator'ın kendi beklenen `modCount` değerini de günceller, bu yüzden tutarsızlık oluşmaz. `Collection.removeIf()` ise aynı güvenli mekanizmayı kullanan, Java 8 ile gelen daha okunaklı bir kısayoldur.

## Dikkat edilmesi gerekenler

- Hata sadece eleman ekleme/çıkarmada değil, bazı koleksiyonlarda `clear()` gibi diğer yapısal değişikliklerde de tetiklenir.
- `CopyOnWriteArrayList` gibi eşzamanlılığa özel koleksiyonlar bu istisnayı fırlatmaz çünkü iterator, değişiklik anındaki dizinin bir kopyası üzerinde çalışır.
- Aynı anda birden fazla thread bir koleksiyonu değiştiriyorsa, tek thread'li kodda bile görülmeyen bu istisna gerçek eşzamanlılık hatalarını maskeleyebilir; kalıcı çözüm için `java.util.concurrent` koleksiyonlarına bakılmalıdır.

## Denemek için

```
jshell> var list = new java.util.ArrayList<>(java.util.List.of("a","b","c"))
jshell> for (var s : list) { if (s.equals("b")) list.remove(s); }
```

İkinci satırda `ConcurrentModificationException` fırlatıldığını görürsünüz. Aynı denemeyi `it.remove()` kullanan bir `while` döngüsüyle tekrarlarsanız hatasız tamamlandığını doğrulayabilirsiniz.

[Kaynak: https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/ConcurrentModificationException.html]
