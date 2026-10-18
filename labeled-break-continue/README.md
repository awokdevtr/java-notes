# Etiketli break ve continue

İç içe döngülerde sıradan bir `break` veya `continue`, sadece kendisini çevreleyen en yakın döngüyü etkiler. Dış döngüyü doğrudan durdurmak veya atlamak istediğinizde genelde bir `boolean` bayrak değişkeni eklenir, ama Java bunun için doğrudan bir dil özelliği sunar: etiketli (labeled) `break` ve `continue`.

## Etiketli break ile iç içe döngüden çıkma

```java
int target = 5;
boolean found = false;

search:
for (int row = 0; row < matrix.length; row++) {
    for (int col = 0; col < matrix[row].length; col++) {
        if (matrix[row][col] == target) {
            found = true;
            break search;
        }
    }
}
```

`search:` etiketi dıştaki `for` döngüsüne bağlanır. İçteki döngüde çalışan `break search;`, sadece içteki döngüyü değil, etiketlenmiş dış döngüyü de tek adımda sonlandırır. Etiket, hemen üzerinde durduğu döngü ifadesine (for, while, do-while) bağlanmak zorundadır; aradaki boşlukta başka bir ifade olamaz. Bayrak değişkeni ve ardından dış döngüde tekrar kontrol etmek yerine, niyeti doğrudan ifade eder.

## Etiketli continue ile dış döngüyü bir adım ilerletme

```java
outer:
for (int i = 0; i < rows; i++) {
    for (int j = 0; j < cols; j++) {
        if (grid[i][j] < 0) {
            continue outer;
        }
        process(grid[i][j]);
    }
}
```

`continue outer;` çalıştığında içteki döngünün kalan iterasyonları tamamen atlanır ve kontrol doğrudan dış döngünün bir sonraki iterasyonuna geçer. Etiketsiz bir `continue` burada yalnızca içteki döngüyü bir adım ilerletirdi; etiket, hangi döngünün ilerletileceğini açıkça seçmenizi sağlar.

## Dikkat edilmesi gerekenler

- Etiket yalnızca bir döngü (`for`, `while`, `do-while`) veya herhangi bir bloğun (`{}`) hemen önüne konabilir; `break` için bloklar da hedef olabilir, ama `continue` sadece döngüleri hedefleyebilir.
- Aşırı kullanım okunabilirliği düşürür; genelde ikiden fazla iç içe döngüde etiket yerine ayrı bir metoda çıkarıp `return` kullanmak daha açık olur.
- Etiket adları herhangi bir tanımlayıcı olabilir ama değişken adlarıyla karışmaması için genelde `camelCase` ve döngünün amacını anlatan bir isim seçilir.

## Denemek için

```
jshell> int[][] m = {{1,2},{3,4}}
jshell> search: for (int[] row : m) { for (int v : row) { if (v == 3) { System.out.println("bulundu"); break search; } } }
```

[Kaynak: https://docs.oracle.com/javase/specs/jls/se21/html/jls-14.html#jls-14.7]
