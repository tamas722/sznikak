# 10. Előadás: Relációs adatmodell és SQL alapok

## 1. Relációs Adatmodell Alapfogalmak
A relációs adatmodell alapja a **reláció**, amely a gyakorlatban egy kétdimenziós táblázatnak felel meg.

* **Attribútum:** A táblázat oszlopa, meghatározott értelmezési tartománnyal.
* **Reláció elem (rekord):** A táblázat egy sora.
* **Relációs séma:** A tábla struktúrájának leírása (pl. `Személy(Név, Nem, Szül.Év)`).
* **Kulcs:** Attribútumok olyan halmaza, amely egyedileg és minimálisan meghatároz egy sort.
    * **Elsődleges kulcs (PK):** A kiválasztott elsődleges azonosító.
    * **Idegen kulcs (FK):** Más tábla PK-jára hivatkozik, biztosítva az adatintegritást.
* **Funkcionális függőség:** Attribútum B függ A-tól, ha A értékeiből egyértelműen következik B értéke.
* **Normalizáció:** A redundancia és az anomáliák (módosítási, törlési, beszúrási) elkerülése érdekében végzett dekompozíció (gyakran 3NF vagy BCNF formára).



---

## 2. SQL Nyelv Alapjai
Az SQL (Structured Query Language) a relációs adatbázisok kezelésére szolgáló nyelv.

### Nyelvi elemek:
* **DDL (Data Definition Language):** Adatdefiníció (pl. `CREATE TABLE`).
* **DML (Data Manipulation Language):** Adatkezelés (pl. `INSERT`, `UPDATE`, `DELETE`, `SELECT`).

### SQL Példa: Tábla létrehozás
```sql
CREATE TABLE Termek (
  TermekID int NOT NULL PRIMARY KEY,
  Nev nvarchar(50) NOT NULL,
  Ar int NOT NULL,
  GyartoID int NULL REFERENCES Gyarto(GyartoID)
)
```
### Magyarázat a táblalétrehozáshoz
A `CREATE TABLE` utasítással létrehozzuk a táblát, meghatározzuk az oszlopok típusát, beállítjuk az elsődleges kulcsot (`PRIMARY KEY`), valamint egy idegen kulcsot (`REFERENCES`) a `Gyarto` táblára, amely garantálja a hivatkozási integritást.

---

### Lekérdezés illesztéssel (JOIN)
Ha több táblából szeretnénk adatokat kinyerni, `JOIN`-t használunk az idegen kulcs és az elsődleges kulcs összekapcsolásával.



```sql
SELECT 
  Termek.*, 
  Gyarto.Nev AS GyartoNev
FROM Termek
JOIN Gyarto
  ON Termek.GyartoID = Gyarto.GyartoID
```

### Magyarázat a JOIN művelethez
A `JOIN` két táblát kapcsol össze a megadott feltétel alapján, és csak azokat a sorokat adja vissza, amelyeknél a feltétel igaz (ez az úgynevezett `INNER JOIN`).

---

## 3. Objektumrelációs Leképezés (ORM)
Az ORM célja a fogalmi modell (pl. UML osztálydiagram) automatikus vagy félautomata leképezése relációs sémára.

* **Osztály -> Tábla:** Minden osztályhoz tábla tartozik, minden tagváltozóhoz pedig egy oszlop.
* **Egy-több kapcsolat:** A „több" oldali táblában egy idegen kulcsot (FK) hozunk létre, amely a „egy" oldali tábla elsődleges kulcsára mutat.
* **Több-több kapcsolat:** Egy külön **kapcsolótáblát** (asszociációs táblát) hozunk létre, amely a két tábla elsődleges kulcsait idegen kulcsként tartalmazza.



---

## 4. ADO.NET
Az ADO.NET a .NET keretrendszer alapszintű adatbázis-elérési technológiája.

### Főbb objektumok:
* **Connection:** Az adatbázis-kapcsolatot kezeli.
* **Command:** SQL parancsok végrehajtására szolgál.
* **DataReader:** A `SELECT` lekérdezések eredményének gyors, csak olvasható, továbbhaladó (forward-only) olvasására szolgál.

### Kapcsolatalapú hozzáférés példa (C#)
```csharp
// 1. Kapcsolat létrehozása
using (SqlConnection conn = new SqlConnection("Data Source=(local);Initial Catalog=SzoftechDB;Integrated security=true"))
{
    // 2. Parancs létrehozása
    SqlCommand command = new SqlCommand("SELECT Nev, Ar FROM Termek", conn);
    conn.Open(); // Kapcsolat nyitása
 
    // 3. Eredmény olvasása
    using (SqlDataReader reader = command.ExecuteReader())
    {
        while (reader.Read())
        {
            Console.WriteLine($"{reader["Nev"]} - {reader["Ar"]}");
        }
    }
} // A 'using' blokk végén automatikusan bezárul a kapcsolat
```

### Magyarázat a kapcsolatkezeléshez
A nyitott adatbázis-kapcsolat drága erőforrás, amely jelentősen terhelheti a szervert, ezért mielőbb le kell zárni. A `using` blokk garantálja a `Close()` (vagy `Dispose()`) hívást még akkor is, ha a kód futása közben hiba (kivétel) történik.

---

## SQL Injection és védekezés
Soha ne fűzd össze a felhasználói bemenetet közvetlenül az SQL parancsszöveggel! Ez egy súlyos biztonsági rés, amelyen keresztül a felhasználó rosszindulatú SQL parancsokat (pl. `DROP TABLE`) injektálhat az adatbázisodba.

* **Megoldás:** Használj mindig `SqlParameter`-eket. Ezzel a rendszer különválasztja a parancsszöveget és az adatot, így az injektált kód nem hajtódik végre.



### Kódpélda: Paraméterek használata (biztonságos beszúrás)
```csharp
// Biztonságos parancs létrehozása paraméterekkel
SqlCommand command = new SqlCommand("INSERT INTO Termek (Nev, Ar) VALUES (@Nev, @Ar)", conn);

// Paraméterek hozzáadása (a rendszer itt validál és szanitál)
command.Parameters.Add(new SqlParameter("@Nev", nev));
command.Parameters.Add(new SqlParameter("@Ar", ar));

// Módosító művelet (nem ad vissza eredményhalmazt)
command.ExecuteNonQuery();
```

## Kapcsolatnélküli (Disconnected) hozzáférés
A folyamatos adatbázis-kapcsolat helyett (ami drága erőforrás) egy hatékonyabb, szakaszos szemléletet alkalmazunk:

1. **Adatok lekérdezése:** Megnyitjuk a kapcsolatot, beolvassuk a szükséges adatokat egy lokális tárolóba (pl. `DataSet` vagy `DataTable`), majd **bezárjuk a kapcsolatot**.
2. **Offline szerkesztés:** A felhasználó a memóriában lévő adatokkal dolgozik, miközben az adatbázis-kapcsolat nem él.
3. **Szinkronizálás (Mentés):** Amikor a felhasználó végez, újra megnyitjuk a kapcsolatot, és a lokális változásokat (INSERT, UPDATE, DELETE) visszaküldjük az adatbázisba egyetlen műveletsorban.



**Előny:** Jelentősen csökkenti az adatbázis-szerver terhelését, mivel a kapcsolatok csak a minimális ideig élnek.