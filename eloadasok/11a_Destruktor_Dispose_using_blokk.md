# 9. Előadás: Memóriakezelés és az IDisposable minta

## 1. A destruktor szerepe és működése
A destruktor (`~Osztálynév()`) egy speciális metódus, amelynek célja az objektumhoz tartozó erőforrások felszabadítása.

* **.NET (C#) sajátosság:** A C#-ban nincs `delete` operátor. A már nem hivatkozott objektumokat a **Szemétgyűjtő (Garbage Collector – GC)** távolítja el. A destruktor definíciója (`~MyClass`) a `Finalize` metódus felülírásával ekvivalens.
* **Jellemzők:**
    * **Nem determinisztikus:** Nem tudjuk, mikor fut le.
    * **Teljesítmény:** Lassítja a szemétgyűjtést, mert az objektumok "finalizer queue"-ba kerülnek.
    * **Veszélyforrás:** Ne hivatkozzunk a destruktorból más, felügyelt (.NET) objektumokra, mert azok már el lehetnek távolítva.

---

## 2. Mikor írjunk destruktort?
Destruktort **csak akkor írjunk**, ha az osztály "nem felügyelt" (unmanaged) erőforrást használ, amiről a .NET nem tud.
* **Nem felügyelt erőforrások:** Fájlleírók, adatbázis-kapcsolatok, mutexek, natív Windows ablakok.
* **Ne használjuk:** Felügyelt memória (saját .NET objektumok) felszabadítására a destruktor nem való.

---

## 3. Determinisztikus felszabadítás: A Dispose minta
Mivel a destruktor lefutása kiszámíthatatlan, a drága erőforrásokat azonnal fel kell szabadítani az `IDisposable` interfész és a `Dispose()` metódus segítségével.
* **Implementáció:** Ha az osztály nem felügyelt erőforrást tartalmaz, implementáljuk az `IDisposable`-t.
* **Dispose() metódus:** Itt szabadítjuk fel a nem felügyelt erőforrásokat, és hívjuk a tartalmazott (szintén `IDisposable`) objektumok `Dispose()` metódusát.



---

## 4. A using blokk
A `using` blokk a legbiztonságosabb módja az `IDisposable` objektumok használatának. Garantálja a `Dispose()` meghívását akkor is, ha kivétel (hiba) történik a kód futása közben.

**Példa:**
```csharp
// Amikor kilépünk a blokkból, automatikusan meghívódik a Dispose()
using (StreamReader sr = new StreamReader("adat.txt"))
{
    string sor = sr.ReadLine();
    // ... használat ...
}
```

5. Kombinált megoldás (Dispose + Destruktor)
A profi megvalósítás az, ha a Dispose metódust és a destruktort kombináljuk. Ez garantálja, hogy ha a fejlesztő elfelejti hívni a Dispose-t, a destruktor "biztonsági hálóként" lezárja az erőforrást. 
Példa a kombinált mintára (leegyszerűsítve):
C#
class MyResource : IDisposable
{
    private bool isDisposed = false;

    public void Dispose()
    {
        Dispose(true);
        // Megmondjuk a GC-nek, hogy a destruktort már nem kell futtatni
        GC.SuppressFinalize(this); 
    }

    protected virtual void Dispose(bool disposing)
    {
        if (!isDisposed)
        {
            if (disposing) 
            {
                // Itt szabadítjuk fel a felügyelt erőforrásokat
            }
            // Itt szabadítjuk fel a nem felügyelt erőforrásokat
            isDisposed = true;
        }
    }

    ~MyResource() // Destruktor: a biztonsági háló
    {
        Dispose(false);
    }
}


### Összehasonlító táblázat: Destruktor vs. Dispose()

| Funkció | Destruktor (~) | Dispose() |
| :--- | :--- | :--- |
| **Hívás** | GC hívja automatikusan | Fejlesztő hívja (vagy `using`) |
| **Időzítés** | Nem determinisztikus (bizonytalan) | Determinisztikus (azonnali) |
| **Cél** | "Biztonsági háló" (erőforrás-szivárgás ellen) | Erőforrások szabályos, gyors lezárása |
| **Kötelező?** | Csak nem felügyelt erőforrásnál | Igen, ha az osztály erőforrást kezel |