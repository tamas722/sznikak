# 13. Előadás: .NET Aszinkron programozás

## 1. Alapfogalmak és Motiváció
* **Szinkron végrehajtás:** A hívó fél blokkolva van, amíg a művelet le nem fut. UI-ban ez a felület "fagyását" okozza.
* **Aszinkron végrehajtás:** A hívó szál nem blokkolódik, így más feladatokat is végezhet.
* **Miért aszinkron?**
    * **UI:** A felület reszponzív marad.
    * **Szerver:** Több kérés egyidejű, skálázható kiszolgálása (a szálak felszabadulnak).

---

## 2. Megvalósítási módok: CPU vs. I/O Bound

| Művelet típus | Jellemzők | Megoldás |
| :--- | :--- | :--- |
| **CPU-bound** | Számításigényes (pl. rendezés, képfeldolgozás) | `Task.Run()`: Külön szálon (ThreadPool) futtatjuk |
| **I/O-bound** | I/O műveletek (fájl, hálózat, adatbázis) | Beépített aszinkron metódusok (hardveres támogatás, nem igényel külön szálat) |



---

## 3. A Task és az Async/Await mechanizmus
A modern .NET a **Task-alapú Aszinkron Mintát (TAP)** használja.

* **Task<T>:** Aszinkron művelet, amely `T` típusú eredményt ad vissza.
* **Task:** Aszinkron művelet, amelynek nincs visszatérési értéke (void-szerű).
* **async kulcsszó:** A függvény deklarációjánál kell megadni, lehetővé teszi az `await` használatát.
* **await kulcsszó:** Aszinkron módon bevárja a `Task` befejeződését. 
    * *Működése:* Ha a Task még nem végzett, a metódus futása felfüggesztődik, a vezérlés visszatér a hívóhoz. Amikor a Task elkészül, a vezérlés "visszaugrik" a metódusba, és folytatódik a futás.

---

## 4. Szabályok és Gyakorlati példa

### Szabályok összefoglalása
1. Az aszinkron metódusok `Task` vagy `Task<T>`-vel térnek vissza.
2. Konvenció: a metódus neve `Async` végződésű.
3. Az `await` kulcsszót használjuk a beváráshoz (sose használd a `.Result`-ot, mert az **blokkol**!).
4. Ha `await` van a metódusban, kötelező az `async` kulcsszó.

### Kódpélda (I/O-bound):
```csharp
// 1. Aszinkron metódus definiálása
public async Task<int> GetUrlContentLengthAsync(string url)
{
    // HttpClient GetStringAsync-ja egy beépített aszinkron I/O művelet
    string contents = await _httpClient.GetStringAsync(url);
    
    // Szimulált lassú művelet (ne Thread.Sleep-et használj!)
    await Task.Delay(2000); 
    
    return contents.Length;
}
// 2. Fogyasztás párhuzamosítással
private async void GetContentLengthButton_Click(object sender, RoutedEventArgs e)
{
    // Elindítjuk a feladatokat egyszerre (nem awaiteljük még)
    Task<int> t1 = GetUrlContentLengthAsync(url1);
    Task<int> t2 = GetUrlContentLengthAsync(url2);
    Task<int> t3 = GetUrlContentLengthAsync(url3);

    // Most bevárjuk őket (awaitelünk az eredményre)
    int len1 = await t1;
    int len2 = await t2;
    int len3 = await t3;

    // Az eredmények itt már elérhetőek
    contentLengthTextBlock.Text = $"{len1 + len2 + len3} bytes";
}
```

### Magyarázat a fenti példához
* **Felfüggesztés blokkolás nélkül:** A `GetUrlContentLengthAsync` metódusban az `await _httpClient.GetStringAsync(url)` sorban a metódus felfüggeszti a saját futását a letöltés idejére, de **nem blokkolja a UI szálat**, így a felhasználói felület reszponzív marad.
* **Párhuzamos végrehajtás:** Ha egyszerre indítjuk el a három `Task`-ot (ahelyett, hogy sorban `await`-elnénk őket), azok **párhuzamosan** kezdenek el futni. Ha mindegyik sor elé `await`-et tennénk, akkor a műveletek szekvenciálisan (egymás után) futnának le, ami jelentősen lassabb lenne.

---

### CPU-bound példa (rövid emlékeztető)
Ha a feladatunk számításigényes (a CPU-t terheli), ne `Task.Delay`-t vagy I/O műveletet használjunk. Ehelyett a `Task.Run` segítségével kényszerítsük a munkát egy külön szálra a `ThreadPool`-ból:



**Kódpélda:**
```csharp
// Task.Run segítségével külön szálon futtatjuk a nehéz számítást
Task<long> task = Task.Run(() => {
    // Itt történik a számítás, ami a CPU-t terheli
    return CalculateSum(); 
});

// A UI szál tovább futhat, amíg a számítás zajlik
long result = await task; // Itt várjuk be az eredményt, ha szükség van rá
```
### További fontos tudnivalók
* **Async Main:** A `Main` metódus is lehet `async Task`, ami megkönnyíti az aszinkron konzolos alkalmazások írását (a C# 7.1-től kezdve támogatott).
* **Az `async` kulcsszó korlátja:** Fontos megjegyezni, hogy az `async` kulcsszó önmagában **nem tesz semmit** (nem indít új szálat, nem teszi automatikusan aszinkronná a metódust). Csupán arra szolgál, hogy "engedélyezze" az `await` kulcsszó használatát a metóduson belül.