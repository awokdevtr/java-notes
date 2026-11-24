# synchronized-collection-wrappers

`Collections.synchronizedList`/`synchronizedMap` çağrısı, hazır bir koleksiyonu alıp her tekil metodu `synchronized` bloğuna sarılmış bir sarmalayıcıya (wrapper) dönüştürür. Bu, tek tek `get`/`put`/`add` çağrılarını thread-safe yapar, ama bir iterator üzerinden gezinme hâlâ birden çok metot çağrısından oluşan tek bir mantıksal işlemdir; sarmalayıcı bu bütünü korumaz ve bu yüzden `for-each` döngüsü sessizce `ConcurrentModificationException` riskiyle baş başa kalır.

## Tekil erişim güvenli, iterasyon değil

```java
List<Integer> list = Collections.synchronizedList(new ArrayList<>());
list.add(1);
list.add(2);

for (Integer i : list) {        // içeride iterator kullanır
    System.out.println(i);      // başka bir thread aynı anda list.add(3) çağırırsa
}                                // ConcurrentModificationException fırlayabilir
```

`for-each` döngüsü kapak altında `list.iterator()` çağırır ve her `next()` çağrısı ayrı, kendi başına kilitlenen bir metot çağrısıdır; döngünün tamamı tek bir kilit altında değildir. Bu yüzden iterasyon sırasında başka bir thread listeye yazarsa, fail-fast iterator değişikliği algılar ve istisna fırlatır — tam da senkronize wrapper'ın önlemesi beklenen türden bir hata.

## Doğru kullanım: manuel senkronizasyon

```java
List<Integer> list = Collections.synchronizedList(new ArrayList<>());

synchronized (list) {            // Javadoc'un açıkça istediği kilit
    for (Integer i : list) {
        System.out.println(i);
    }
}
```

`Collections.synchronizedList` Javadoc'u, kullanıcıdan döndürülen listenin kendisini kilit nesnesi olarak kullanıp iterasyonu manuel bir `synchronized` bloğuna almasını açıkça ister. Bu adım atlanırsa sarmalayıcı sadece yanlış bir güven duygusu verir; `CopyOnWriteArrayList` veya `ConcurrentHashMap` gibi gerçek eşzamanlı koleksiyonlar bu manuel adıma gerek duymaz.

## Dikkat edilmesi gerekenler

- Döndürülen nesne orijinal koleksiyonun kendisi değil, ona delege eden ayrı bir sarmalayıcıdır; orijinal referans üzerinden senkronize edilmeden erişmek kilidi tamamen atlar.
- `keySet()`, `values()`, `entrySet()` gibi görünümler de aynı kuralın kapsamındadır: üzerlerinde gezinirken yine ana koleksiyonun kilidine ihtiyaç vardır.
- Yüksek eşzamanlılık gerektiren durumlarda bu sarmalayıcılar yerine `java.util.concurrent` paketindeki koleksiyonlar tercih edilmelidir; onlar kaba kilitleme yerine ince taneli veya kilitsiz stratejiler kullanır.

## Denemek için

```
jshell> var list = java.util.Collections.synchronizedList(new java.util.ArrayList<Integer>())
jshell> list.add(1); list.add(2); list.add(3)
jshell> synchronized (list) { for (var i : list) System.out.println(i); }
```

[Kaynak: https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/Collections.html#synchronizedList(java.util.List)]
