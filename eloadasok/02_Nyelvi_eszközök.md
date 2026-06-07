## 1. A Típusrendszer és Memóriakezelés (A legfontosabb alapok)

A C# típusrendszere a .NET CTS-re (Common Type System) épül. Ez azt jelenti, hogy a C# beépített típusai (pl. `int`, `string`) valójában .NET osztályok/típusok rövidítései (pl. az `int` pontosan ugyanaz, mint a `System.Int32`). Minden objektum (még az egyszerű számok is) a `System.Object`-ből származik.

A típusokat két nagy csoportra osztjuk, aminek a megértése kritikus a memóriakezelés szempontjából:

### Érték típusok (Value types)
* **Mik ezek?** `int`, `double`, `bool`, `char`, valamint a magunk által definiált `struct` és `enum`.
* **Hol tárolódnak?** A memóriában a vermen (**stack**) jönnek létre, és automatikusan törlődnek, amint kikerülnek a hatókörből (nem kell hozzájuk Garbage Collector).
* **Hogyan viselkednek?** Értékadáskor vagy függvénynek történő átadáskor mindig másolat készül róluk.

### Referencia típusok (Reference types)
* **Mik ezek?** `class`, tömbök, `string`, `delegate`, `interface`.
* **Hol tárolódnak?** Maga az adat a heap-en (**kupac**) jön létre (ehhez kell a `new` kulcsszó), a változó pedig csak egy memóriacímet (referenciát) tárol, ami erre az adatra mutat.
* **Hogyan viselkednek?** A Garbage Collector (GC) takarítja el őket, ha már nem mutat rájuk referencia. Átadáskor csak a "mutató" másolódik, nem maga az adat.

### Boxing és Unboxing (Be- és kidobozolás)
Mivel minden típus az `object`-ből származik, egy érték típust (pl. `int`) beletehetünk egy referencia típusba (`object`).

* **Boxing:** Az érték típushoz a rendszer létrehoz egy "doboz" objektumot a heap-en, és abba másolja az értéket.
* **Unboxing:** Az objektumból visszanyerjük az érték típust (castolással).
* **Fontos:** A dobozolás lassít, mert memóriát foglal a heap-en!

---

## 2. Osztályok, Struktúrák és Adattagok

### Osztály (Class) vs. Struktúra (Struct)
* **Struct:** Érték típus. Nem lehet belőle örökölni, és ő sem örökölhet (csak interfészt valósíthat meg). Kisméretű adatokhoz jó (pl. 3D koordináta), hogy megspóroljuk a heap allokációt.
* **Class:** Referencia típus. Lehet örököltetni, gazdag objektumokhoz használjuk.

### Property (Tulajdonság)
A tagváltozókat (mezőket) illik privátra állítani, hogy védjük az objektum állapotát (pl. egy életkor ne lehessen negatív). A Property egy elegáns nyelvi elem a privát adatok lekérdezésére (`get`) és beállítására (`set`).

**Kódpélda magyarázattal:**

```csharp
class Person
{
    private int yearOfBirth; // Rejtett tagváltozó

    // Ez itt a Property:
    public int YearOfBirth
    {
        get { return yearOfBirth; } // Amikor lekérdezik az értéket
        set 
        { 
            // Amikor beállítják az értéket (a 'value' a bejövő adat)
            if (value < 1800) throw new Exception("Túl régi év!");
            yearOfBirth = value; 
        }
    }

    // Auto-implementált property (a fordító maga csinál mögé egy rejtett változót)
    // Megadhatunk kezdőértéket is, és korlátozhatjuk a settert!
    public string Name { get; private set; } = "Anonymous";
}
```

*Magyarázat:* Ha a kódodban azt írod, hogy `p1.YearOfBirth = 1995;`, akkor a háttérben a `set` ág fut le. Ha kihagyod a `set` ágat (vagy priváttá teszed, mint a `Name` esetén), a tulajdonság a külvilág számára csak olvasható (`readonly`) lesz. Az auto-implementált propertynél nem kell megírni a logikát, ha nincs rá szükség.

### Const vs. Readonly
* **const:** Fordítási időben dől el az értéke, kötelezően statikus jellegű (osztályszintű), és csak primitív típusokra (számok, string) használható.
* **readonly:** Futási időben értékeli ki a gép, lehet példányszintű is (objektumhoz kötött), és értékét konstruktorban is be lehet állítani.

---

## 3. Paraméterátadás: ref és out

Alapértelmezetten a paraméterek érték szerint adódnak át. Ha a függvény módosítja a paramétert, az eredeti változó nem változik meg. Ezt kerüli meg a `ref` és az `out`.

* **ref:** Referencia szerint adja át a változót (mint a C-ben a mutatók). A függvény meg tudja változtatni a kinti eredeti értéket. **Szabály:** a hívás előtt a változónak kötelező értéket adni!
* **out:** Hasonló a `ref`-hez, de az adat csak "kifelé" jön a függvényből (pl. visszatérési érték helyett használjuk, ha több adatot akarunk visszaadni). **Szabály:** nem kötelező a hívás előtt inicializálni.

**Kódpélda:**

```csharp
void Increment(ref int n) { n += 10; }

int value = 5; // Kötelező értéket adni!
Increment(ref value); 
// A 'value' most már 15 lesz, mert a függvény az eredetit módosította.
```
## 4. Delegátok és Események (Kiemelten fontos!)

### Delegate (Delegát)
A delegát egy objektumorientált, típusos "függvénymutató". Segítségével futási időben dönthetjük el, hogy egy vagy több metódus közül melyiket hívjuk meg. Egyszerre több függvényre is mutathat (ez a **multicast** tulajdonság), ha a `+=` operátorral fűzzük hozzájuk.

**Kódpélda:**

```csharp
// 1. Definiáljuk a delegát TÍPUST (milyen függvényekre mutathat?)
delegate void MyDelegate(string msg);

class Program 
{
    static void PrintConsole(string msg) { Console.WriteLine(msg); }

    static void Main() 
    {
        // 2. Létrehozunk egy delegát változót, és ráállítjuk a függvényre
        MyDelegate del = PrintConsole; 
        
        // 3. Meghívjuk. Ez automatikusan meghívja a PrintConsole-t.
        del("Hello!"); 
    }
}
```
### Event (Esemény)
A modern alkalmazások eseményvezéreltek (pl. gombnyomás). Az esemény (Publisher/Subscriber minta) a háttérben delegátokra épül. 

**Különbség a sima delegáthoz képest:** Az `event` kulcsszó megvédi a delegátot! Kívülről senki sem teheti egyenlővé egy másik függvénnyel (`=`), csak feliratkozni (`+=`) és leiratkozni (`-=`) lehet róla. Valamint csak az az osztály sütheti el, amelyik definiálta.

**Kódpélda:**

```csharp
// 1. Delegát típus a szignatúrához
public delegate void LogHandler(string msg);

class Logger
{
    // 2. Az esemény definiálása
    public event LogHandler Log; 

    public void DoWork() {
        // 3. Esemény elsütése (Biztonságos hívás: ?.Invoke)
        // Csak akkor sül el, ha van legalább egy feliratkozó (nem null)
        Log?.Invoke("A munka elkészült!"); 
    }
}

class App {
    public App() {
        Logger logger = new Logger();
        // 4. Feliratkozás az eseményre (a WriteConsole fog lefutni)
        logger.Log += WriteConsole; 
        logger.DoWork();
    }

    void WriteConsole(string msg) { Console.WriteLine(msg); }
}
```
## 5. Egyéb fontos nyelvi eszközök

### Stringek és hatékonyság
A `string` C#-ban **immutable** (megváltoztathatatlan). Ha összefűzöl stringeket a `+` jellel (pl. `"A" + "B"`), akkor mindig egy teljesen új string objektum jön létre, a régit pedig a Garbage Collectornak kell eltakarítania. Ha ciklusban csinálod, nagyon lelassítja a gépet.
* **Megoldás:** Használj `StringBuilder`-t, vagy string interpolációt: `$"Név: {name}"`.

### Operátorok és Castolás
* **Logikai operátorok:** A `&&` és `||` operátorok "rövidrezáróak" (short-circuit): ha az első tagból kiderül a végeredmény, a másodikat már ki sem értékeli a gép. Ezzel szemben a `&` és `|` logikai használata esetén mindkét oldal lefut.
* **is operátor:** Megvizsgálja a típust, és egy `bool`-t ad vissza (`true`, ha sikeres a cast).
* **as operátor:** Megpróbál castolni. Ha nem sikerül, kivétel dobása helyett `null`-t ad vissza (nagyon biztonságos).

### Tömbök (Arrays)
* **Szintaktika:** Egydimenziós: `int[]`, Kétdimenziós (mátrix): `int[,]`, Tömbök tömbje (jagged): `int[][]`.
* **Memóriakezelés:** Érték típusú tömbnél (pl. `struct` elemek) a memória egyben lefoglalódik. Referencia típusú tömbnél (pl. `class` elemek) a tömb létrejöttekor csak `null` mutatók jönnek létre, egy ciklusban minden elemre külön rá kell hívni a `new`-t.

### Attribútumok (Attributes)
Deklaratív metaadatok, amiket hozzárendelhetünk osztályokhoz, metódusokhoz (hasonló a Java annotációkhoz). Például a `[Serializable]` jelzi a keretrendszernek, hogy az adott osztályt el lehet menteni binárisan fájlba. Futásidőben a **Reflection** segítségével olvashatók ki ezek az adatok.

---

## 6. Vezérlési szerkezetek és a foreach működése

A hagyományos vezérlési szerkezetek (`if-else`, `switch`, `while`, `do-while`, `for`, `break`, `continue`) a C/C++ és Java nyelvekhez hasonlóan működnek.

### A foreach működése
A `foreach` ciklus egy egyszerűsített iteráció, amely bármilyen olyan kollekción vagy tömbön használható, amely megvalósítja az `IEnumerable` interfészt (azaz rendelkezik egy `GetEnumerator()` metódussal).

* A háttérben a `GetEnumerator()` visszaad egy `IEnumerator` objektumot.
* Ez a `MoveNext()` metódussal lépked előre, a `Reset()` metódussal újraindul.
* A `Current` tulajdonságon keresztül biztosítja az aktuális elem elérését.

**Kódpélda:**

```csharp
string[] napok = { "Hétfő", "Kedd" }; 

foreach (string nap in napok) 
{
    Console.WriteLine(nap);
}
```
### 2. Felsorolt típusok: Enum és Flags (Bitmezők)

Az `enum` nevesített konstansok használatát teszi lehetővé számértékek helyett az olvashatóság növelése érdekében, miközben erősen típusos és megőrzi az egész típusok (`int`) hatékonyságát.

A `[Flags]` attribútummal ellátott enumok (bitmezők) lehetővé teszik az értékek bitműveletekkel (`|`, `&`) való kombinálását. Az értékeknek kötelezően kettő hatványainak kell lenniük, hogy pontosan egy-egy bitet reprezentáljanak.

**Kódpélda:**

```csharp
[Flags] // Jelzi, hogy kombinálható bitmezőről van szó
enum Mód : byte { 
    Olvasás = 0x1, 
    Írás = 0x2, 
    Végrehajtás = 0x4 
}

// Kombinálás bitenkénti VAGY (|) operátorral
Mód jogok = Mód.Olvasás | Mód.Írás;

// Ellenőrzés bitenkénti ÉS (&) operátorral
if ((jogok & Mód.Írás) != 0) 
{
    Console.WriteLine("Van írási jog.");
}
```
*Magyarázat: A [Flags] attribútumnak köszönhetően a jogok.ToString() nem egy számot vagy egyetlen elemet fog kiírni, hanem a kombinált elemek listáját szövegesen **("Olvasás, Írás").

3. OOP Kulcsszavak: virtual, override, abstract, new
•	C#-ban a függvények és tulajdonságok alapértelmezetten nem virtuálisak. 
•	virtual: Csak az ezzel a kulcsszóval megjelölt metódusok alakíthatók át a leszármazott osztályokban. 
•	override: Kötelező kiírni a leszármazott osztályban, ha felülírjuk az ős virtuális metódusát. 
•	abstract: Az absztrakt osztály nem példányosítható. Az absztrakt metódusnak nincs törzse, implementálása a leszármazottakban kötelező az override használatával . 
•	new: Lehetővé teszi, hogy a leszármazottban elfedjünk egy ősosztálybeli metódust, megszakítva a virtuális függvény láncolatát (ritkán használt eszköz). 
Kódpélda:
```csharp
[cite_start]abstract class Alakzat [cite: 375, 376, 386]
{
    public abstract void Rajzol(); // Nincs törzse, kötelező felülírni [cite: 378, 379]
}

[cite_start]class Kor : Alakzat [cite: 388]
{
    [cite_start]public override void Rajzol() // Kötelező az override kulcsszó [cite: 373, 379]
    {
        // Kör rajzolás logikája...
    }
}
```
### 4. Láthatósági szintek (Visibility)

A C++ modellt (`public`, `protected`, `private`) kibővíti a .NET környezet sajátosságaival:

* **private:** Csak az adott osztályon belül érhető el.
* **protected:** Az adott osztályban és annak leszármazottaiban érhető el.
* **public:** Bárhonnan elérhető.
* **sealed:** Lezárt osztály, nem lehet belőle tovább örökölni.
* **internal:** Csak az adott szerelvényen (assembly-n, lefordított projekten) belül publikus.
* **protected internal:** Elérhető a szerelvényen belül bárhonnan, illetve azon kívül a leszármazott osztályokban.

### 5. Indexerek

Az indexerek konzisztens módot biztosítanak arra, hogy egy konténer vagy gyűjtemény elemeit a tömbökhöz hasonlóan, a `[]` operátor segítségével érjük el.

Szintaktikája a tulajdonságokhoz (`property`) hasonlít (`get`/`set` blokkok), de definíciókor a `this[...]` kulcsszót használja. Az index típusa bármi lehet (nem csak egész szám, akár string is).

**Kódpélda:**

```csharp
class AdatTarolo
{
    private string[] adatok = new string[10];

    // Indexer definíciója int típusú indexre
    public string this[int index]
    {
        get { return adatok[index]; }
        set { adatok[index] = value; }
    }
}
```
### 6. Operátorok felüldefiniálása (Operator Overloading)

A legtöbb aritmetikai, relációs és logikai operátort felüldefiniálhatjuk a saját osztályainkra a `public static` módosítókkal és az `operator` kulcsszóval.

* **Nem felüldefiniálható:** Az értékadás (`=`), valamint a speciális operátorok, mint a `sizeof`, `new`, `is`, `typeof`.

**Kódpélda:**

```csharp
class Pont
{
    public int X;

    public static Pont operator +(Pont p1, Pont p2) // + operátor felülírása
    {
        Pont uj = new Pont();
        uj.X = p1.X + p2.X;
        return uj;
    }
}
```
### 7. Statikus konstruktorok

A statikus konstruktorok a statikus változók (osztályszintű változók) inicializálására szolgálnak.

* **Tulajdonságok:** Nem mi hívjuk meg őket, és nincs láthatósági módosítójuk (pl. nincsenek `public` vagy `private` jelzők).
* **Mikor fut le?** Automatikusan az osztály legelső példányának létrejötte, vagy a legelső statikus függvényhívás / változóelérés előtt pontosan egyszer.

**Kódpélda:**

```csharp
class CharInfo
{
    static bool[] isAlpha;

    static CharInfo() // Statikus konstruktor 
    {
        isAlpha = new bool[256]; // Egyszer fut le a program során
    }
}
```
### 8. Kivételkezelés (Exception Handling)

* **try:** Ide kerül a futtatni kívánt, potenciálisan hibát okozó kód.
* **catch:** Elkapja és kezeli a megadott típusú futásidejű hibát (kivételt).
* **finally:** Olyan kódblokk, ami minden esetben lefut (akkor is, ha történt hiba, és akkor is, ha nem) – általában erőforrások takarítására, lezárására használjuk.

A beépített `Exception` osztály legfontosabb tulajdonságai: `Message` (szöveges hibaüzenet), `StackTrace` (a hívási lánc, ahol a hiba történt) és az `InnerException` (beágyazott hiba).



---

### 9. Névterek (Namespaces) és Basic IO

* **Névterek (namespace):** A kódok szemantikai csoportosítására és a névütközések elkerülésére szolgálnak. C# 10-től kezdve használható a fájlszintű névterezés is (pl. `namespace Game;`), így nem kell a teljes fájl tartalmát kapcsos zárójelek közé tenni. Más névterek behozatalára a `using` kulcsszó szolgál.

* **Alapvető I/O (System.IO):**
    * Fájl- és könyvtárkezelésre a `File`, `Directory`, `Path` statikus segédosztályok, valamint a `FileInfo` használható.
    * Az adatfolyamok őse a `Stream` (pl. `FileStream`).
    * Bináris adatok írására/olvasására a `BinaryWriter`/`BinaryReader` szolgál.
    * Szöveges fájlok kezelésére a `StreamWriter`/`StreamReader` (melyek a `TextWriter`/`TextReader` leszármazottai) szolgálnak.