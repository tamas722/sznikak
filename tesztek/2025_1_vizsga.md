# Szoftvertechnológia vizsgasor

## 1. Igaz/Hamis állítások (5p)
*(Jó válasz: 1p, üres: 0p, rossz: -1p)*

- [H] A .NET beépített List osztálya szálbiztos (thread safe).
    - **Magyarázat:** A .NET beépített `List<T>` osztálya alapértelmezetten nem szálbiztos. Több szálról történő egyidejű módosításához (vagy olvasás/írás kombinációjához) külső szinkronizációra (pl. `lock`) van szükség.
- [H] C#-ban célszerű destruktort írni egy osztályban, amennyiben az osztály a konstruktorában egy nagyméretű tömböt foglal le (a new operátorral) és tárol el egy tagváltozójában.
    - **Magyarázat:** C#-ban egy nagyméretű tömb egy felügyelt (managed) erőforrás, amit a Garbage Collector (GC) automatikusan felszabadít. Destruktort (Finalize metódust) kizárólag nem felügyelt (unmanaged) erőforrások (pl. natív fájlleírók) takarítására szabad és célszerű írni.
- [I] A JIT compiler feladata, hogy az IL (köztes) kódot natív gépi kódra fordítsa le.
    - **Magyarázat:** A JIT (Just-In-Time) fordító feladata valóban az, hogy a platformfüggetlen IL (köztes) kódot a metódus legelső meghívásakor natív gépi kódra fordítsa le.
- [H] .NET környezetben egy folyamat (processz) a kilépéskor bevárja az összes háttér szálat.
    - **Magyarázat:** A folyamat (processz) kilépéskor nem várja be a háttérszálakat (background threads). Ha minden előtérszál befejeződik, a folyamat leáll, és a háttérszálak azonnal megszakadnak.
- [I] Egy adott szál rekurzívan többször is megszerezheti a zárat ugyanazon lock objektumra.
    - **Magyarázat:** Egy szál a .NET-ben rekurzívan (egymásba ágyazva) többször is megszerezheti ugyanazt a zárat (`lock` objektumot) anélkül, hogy holtpontot (deadlock) okozna.

## 2. Delegált típus (2p)
**Kérdés:** Adja meg, mely beépített .NET típussal lehet olyan függvényre hivatkozni, melyeknek egy `char` és egy `bool` paramétere van (ebben a sorrendben), és `void` a visszatérési értéke.
**Válasz:** `Action<char, bool>`
* **Magyarázat:** A .NET beépített generikus delegáltjai közül a `Func` értékkel tér vissza, míg az `Action` olyan metódusokra hivatkozik, amelyek visszatérési értéke `void`. A típusparaméterek határozzák meg a bemeneti paramétereket.

## 3. Using blokk (1p)
**Kérdés:** Mely művelet meghívását eredményezi az alábbi kód a blokkból való kilépés során az `a1` objektumon? `using (A a1 = new A) { .... }`
**Válasz:** `Dispose()`
* **Magyarázat:** A `using` blokk elhagyásakor (akár normál futással, akár kivétel miatt) a fordító garantáltan és determinisztikusan meghívja a hivatkozott objektum `Dispose` metódusát.

## 4. Interfész (1p)
**Kérdés:** Mely interfészt kell .NET környezetben egy osztályban megvalósítani, amennyiben az osztály nem felügyelt erőforrásra hivatkozik, és szeretnénk azt determinisztikusan felszabadítani?
**Válasz:** `IDisposable`
* **Magyarázat:** Ezt az interfészt kell megvalósítani ahhoz, hogy egy objektum `Dispose` metódusa hívható legyen, és az objektum fel tudjon szabadítani nem felügyelt erőforrásokat a GC megvárása nélkül.

## 5. Szálkezelés (2p)
**Kérdés:** .NET környezetben a célunk az, hogy amikor egy szál beolvas egy adatot, akkor úgy tudjon jelezni más erre várakozó szálaknak, hogy azok mind tovább futhassanak, és feldolgozhassák az adatot. Adja meg egy szóval, mit használna ehhez!
**Válasz:** `ManualResetEvent`
* **Magyarázat:** Míg az `AutoResetEvent` jelzéskor csak egyetlen várakozó szálat enged tovább (és automatikusan visszaáll blokkoló állapotba), addig a `ManualResetEvent` kinyitja a "kaput", és az összes várakozó szálat egyszerre továbbengedi.

## 6. Igaz/Hamis állítások (8p)
*(Jó válasz: 2p, üres: 0p, rossz: -1p)*

- [I] A Dependency Injection tervezési minta lényege, hogy az osztály a függőségeit konstruktor vagy művelet paraméterekben kapja meg (interfész típusként).
    - **Magyarázat:** A Dependency Injection (Függőséginjektálás) lényege, hogy a függőségeket az osztály nem maga példányosítja, hanem kívülről, többnyire interfészként kapja meg a konstruktorán (vagy property/metódus) keresztül.
- [I] A Singleton tervezési mintában az osztály egyetlen példányát az osztály egy statikus tagváltozója tárolja.
    - **Magyarázat:** A Singleton mintában az egyetlen globális példányt egy statikus (osztályszintű) tagváltozóban tároljuk, amelyet egy statikus metódus (vagy property) szolgáltat a hívóknak.
- [I] A Proxy tervezési mintában a kliens objektumban van egy mutató/hivatkozás egy proxy objektumra, a proxy objektumban pedig van egy mutató/hivatkozás az eredeti (subject) objektumra.
    - **Magyarázat:** A Proxy minta során a kliens egy proxy objektumon keresztül kommunikál, és a proxy tart fenn egy hivatkozást az eredeti, valós (subject) objektumra.
- [H] A Document-View architektúrában a dokumentumban külön tagváltozót vezetünk be az egyes nézetekre.
    - **Magyarázat:** A Document-View (Dokumentum-Nézet) architektúra az Observer (Megfigyelő) tervezési mintára épül. A dokumentum nem tart fenn külön-külön dedikált tagváltozókat a nézeteknek, hanem egy általános listát vezet a feliratkozott megfigyelőkről, így a nézetek száma dinamikusan változhat.

## 7. Igaz/Hamis állítások (2p)
*(Jó válasz: 1p, üres: 0p, rossz: -1p)*

- [H] Az adatforrás által vezérelt csővezeték architektúra szűrő/filter pszeudókódja egy „Read” művelet, mely beolvassa a bemenő csővezetéken az adatot, majd feldolgozza azt.
    - **Magyarázat:** Az adatforrás által vezérelt (source-driven) csővezeték/szűrő architektúrában az adatáramlást a forrás irányítja: a szűrő egy `Write` metóduson keresztül kapja meg az adatot az előző elemtől, majd a feldolgozás után ő hívja meg a következő elem `Write` metódusát. A `Read` metódus az adatnyelő által vezérelt megközelítésre jellemző.
- [I] Az üzleti logikai réteg a két- és háromrétegű architektúrának is része.
    - **Magyarázat:** Az üzleti logikai réteg alapvető része az alkalmazásoknak, amely a kétrétegű (kliens-szerver) és a háromrétegű (megjelenítés, üzleti logika, adatbázis) architektúrának is egyaránt funkcionális része, csupán a fizikai/logikai szétválasztásukban van eltérés.

## 8. Aszinkron programozás (2p)

* **a) Deklaráció:** `async Task<string> GetLongestAsync(string[] strings)`
    * **Magyarázat:** A függvény aszinkron végrehajtásához `async` módosító szükséges, visszatérési értékként pedig nem szimpla `string`-et, hanem `Task<string>`-et kell használni.
* **b) Hívás:** `Console.WriteLine(await GetLongestAsync(strings));`
    * **Magyarázat:** Az aszinkron függvény eredményét az `await` kulcsszóval kell megvárni a híváson belül.
* **c) Hívó függvény kulcsszava:** `async`
    * **Magyarázat:** Az `await` kulcsszó kizárólag olyan metódusokban használható, amelyek maguk is `async` módosítóval vannak ellátva.

## 9. ADO.NET (1p)
**Kérdés:** Milyen osztályt kell használni MSSQL szerver esetén egy több rekordból álló eredményhalmaz feldolgozására?
**Válasz:** `SqlDataReader`
* **Magyarázat:** MSSQL szerver esetén a `SqlCommand.ExecuteReader()` hívása egy `SqlDataReader` objektumot ad vissza, amellyel a több rekordból álló eredményhalmaz sorain lehet végigiterálni a `.Read()` metódussal.

## 10. Reflection (1p)
**Kérdés:** Adjon meg egy egysoros kódot, mely lekérdezi egy „s1” nevű lokális változó típusának nevét.
**Válasz:** `s1.GetType().Name` (vagy névtérrel együtt `s1.GetType().FullName`)
* **Magyarázat:** A `GetType()` az `Object` ősosztályból örökölt metódus, amely a futásidejű típust (egy `Type` objektumot) adja vissza, melynek `Name` vagy `FullName` tulajdonsága tartalmazza a kért nevet.