# 3. Előadás

## 1. Lokális típuskövetkeztetés (var kulcsszó)

A C#-ban lokális változók deklarálásakor a konkrét típus megadása helyett használhatjuk a `var` kulcsszót.

### Szabályok és feltételek:
* **Kötelező inicializálás:** A `var` csak akkor használható, ha a változót a deklarációval egy időben inicializáljuk is. A fordító az értékadás jobb oldalán álló kifejezésből következteti ki a valódi típust fordítási időben.
* **Kizárólag lokális:** Csak metódusokon belüli lokális változók esetén alkalmazható.
* **Tilos használni:** Nem használható osztályok tagváltozóira (field) és függvények paramétereire sem.
* **Ekvivalencia:** A `var` használatától a kód **nem lesz gyengén típusos!** A fordító a háttérben pontosan behelyettesíti a típust, így a futási teljesítmény teljesen megegyezik a hagyományos deklarációval.

### Kódpélda:

```csharp
// Így használható:
var list = new Complex(10, 20); // A fordító kikövetkezteti, hogy a 'list' típusa 'Complex'

// Ez pontosan megegyezik a következő hagyományos kóddal:
Complex list = new Complex(10, 20); 

class HibasPelda 
{
    // Tilos! Tagváltozó nem lehet var:
    // private var adat = 10; 
    
    // Tilos! Függvényparaméter nem lehet var:
    // public void Muvelet(var parameter) { } 
}
```

## 2. Generikus típusok (Generics)
Generikus típusokat akkor használunk, ha olyan osztályt vagy függvényt akarunk írni, amelynek működési logikája független attól, hogy milyen típusú adatokkal dolgozik. A konkrét típust a felhasználó csak a példányosításkor/híváskor adja meg. 

### Az object alapú működés problémái (Miért kell a Generics?)
A generikus típusok előtt mindent a közös ős, az object típus segítségével tároltak általános gyűjteményekben (pl. ArrayList). Ennek komoly hátrányai vannak: 

* Típus conversion (Castolás) kényszere: Az elemek kivételekor explicit módon castolni kell, ami felesleges plusz kódírást jelent. 
* Futásidejű hibák: Mivel az object mindent elfogad, a típusbeli tévedésekre (hibákra) nem derül fény fordítási időben, hanem a program futása közben fog crashelni (InvalidCastException). 
* Keveredő típusok: Nincs kikényszerítve, hogy egy gyűjteménybe csak azonos típusú elemek kerüljenek (pl. egy Person listába simán be lehet tenni egy int-et). 
* Be- és kidobozolás (Boxing/Unboxing) teljesítményproblémája: Érték típusok (pl. int, float, struct) object-ként való tárolásakor a .NET runtime kénytelen az értéket egy heap-en allokált objektumba csomagolni (boxing), kivételkor pedig kibontani (unboxing). Ez nagyságrendekkel lassabb és helypazarlóbb, mint a memóriában egymás mellett elhelyezkedő tiszta értékek tömbje. 

### C++ template-ek vs. .NET Generics (ZH-ra fontos elmélet!)
* C++ Sablonok: Önmagukban le sem fordulnak a build során, csak a felhasználásuk helyén fejtődnek ki. Minden egyes típusvariációra külön gépi kód generálódik (kódburjánzás / code bloat). Ha nem használunk egy sablont, ki sem derülnek a benne lévő szintaktikai hibák. A forráskódot közzé kell tenni a felhasználáshoz (nincs kódvédelem). 
* .NET Generics: A generikus típus lefordul a build során egy köztes IL (köztes) kódra. Bár az érték típusokhoz külön kód generálódik a futás során, a referencia típusok (osztályok) esetén egyetlen közös kód generálódik a memóriában, így nincs kódburjánzás. A forráskód rejtve maradhat. 

### Mi lehet generikus .NET-ben?
Osztály, struktúra, tagfüggvény (művelet), interfész és delegate. 

### Kódpélda:
**Generikus Osztály és Metódus megvalósítása**
Alább látható egy típusbiztos, generikus Verem (Stack<T>) tároló osztály és egy generikus csere (Swap) metódus. 

```csharp
using System;
using System.Collections.Generic;

// Generikus osztály, ahol a T a helyettesítő típusparaméter
public class Stack<T> 
{
    private readonly int size; 
    private int current = 0; 
    private T[] items; // T típusú tömb az elemeknek
    
    public Stack(int size) {
        this.size = size; 
        items = new T[size]; // Helyfoglalás a konkrét T típusú elemeknek
    }
    
    public void Push(T item) { 
        items[current++] = item; 
    }
    
    public T Pop() { 
        return items[--current]; 
    }
}

public class Program
{
    // Generikus tagfüggvény (metódus) két érték megcserélésére
    // A 'ref' kulcsszó segítségével cím szerint adjuk át a paramétereket (pointer-szerűen),
    // így az eredeti változók értéke módosul a metóduson kívül is.
    public static void Swap<T>(ref T lhs, ref T rhs) { 
        T temp = lhs; 
        lhs = rhs; 
        rhs = temp; 
    }
 
    public static void Main() {
        // Típusbiztos osztálypéldányosítás int típussal
        Stack<int> intStack = new Stack<int>(); 
        intStack.Push(1); 
        // intStack.Push("ss"); -> Ez a sor fordítási hibát adna! Típusbiztonság! 
        int i = intStack.Pop(); // Nincs szükség castolásra!
 
        // Generikus metódus meghívása:
        int a = 2, b = 3; 
        Swap<int>(ref a, ref b); // Explicit típusmegadással 
        Swap(ref a, ref b);      // Implicit módon: a fordító kikövetkezteti a paraméterekből 
    }
}
```

### 3. Generikus kényszerek (Constraints)

Ha egy generikus osztályban a T típusparaméteren olyan műveletet akarunk végezni, ami nem értelmezhető minden létező típusra (pl. összehasonlítás, példányosítás), a fordító hibát jelez. Alapértelmezetten a fordító csak az object ősosztály metódusait (pl. ToString()) engedi meghívni T-n. A megoldás a kényszerek alkalmazása a `where` kulcsszóval, amivel leszűkítjük a használható típusok körét.

**Legfontosabb kényszertípusok:**

* **where T : IComparable** $\rightarrow$ T csak olyan típus lehet, ami megvalósítja az adott interfészt (így biztosan lesz CompareTo metódusa).
* **where T : class** $\rightarrow$ T csak referencia típus (osztály) lehet.
* **where T : struct** $\rightarrow$ T csak érték típus (egyszerű típusok, struktúrák) lehet.
* **where T : new()** $\rightarrow$ T-nek kötelezően rendelkeznie kell egy paraméter nélküli alapértelmezett konstruktorral (lehetővé teszi a `new T()` hívást a kódon belül).

**Kódpélda összetett kényszerek magyarázatával:**

```csharp
using System;
 
// Egy fiktív interfész
public interface IKeyProvider<K> { }
public interface IPersistable { }
 
// Összetett generikus osztály kényszerekkel felvértezve
public class CustomDictionary<K, V> 
    where K : IComparable<K> // A K-nak összehasonlíthatónak kell lennie önmagával
    where V : class, IKeyProvider<K>, IPersistable, new() 
    // A V jelentése: csak referencia típus lehet, implementálnia kell a két megadott interfészt,
    // és kötelező, hogy legyen paraméter nélküli konstruktora.
{
    public void AddElement(K kulcs, V ertek) {
        // Mivel V-re van new() kényszer, ezért legálisan megtehetjük ezt:
        V ujElem = new V(); 
        
        // Mivel K-ra van IComparable kényszer, bátran használhatjuk a CompareTo-t:
        if(kulcs.CompareTo(kulcs) == 0) { /* ... */ }
    }
}
```

### 4. Lambda kifejezések (Lambda Expressions)

A lambda kifejezések lehetővé teszik a névtelen (anonymous) függvények helyben történő, rendkívül tömör definiálását. C#-ban a lambda kifejezések deklarációs operátora a `=>`.

#### .NET vs. Java Lambda különbségek (ZH elmélet!)
* **Java:** Az implementáció interfészekre épül (Functional Interface), az operátora pedig a `->`.
* **.NET/C#:** Az implementáció a `delegate` típusokra épül, az operátora pedig a `=>`.

A lambda típusát mindig egy vele kompatibilis (megegyező paraméterekkel és visszatérési értékkel bíró) `delegate` típus határozza meg.

#### Két fő szintaktikai változata:
* **Utasítás (Statement) lambda:** A jobb oldalon kapcsos zárójelek `{}` között utasítások állnak. Explicit módon kell kiírni a `return` kulcsszót és a pontosvesszőket.
    * *Szintaxis:* `(paraméterek) => { <utasítások>; return érték; }`

* **Kifejezés (Expression) lambda:** A jobb oldalon nincsenek kapcsos zárójelek, csak egyetlen kifejezés szerepel. Nem kell és nem is szabad kitenni a `return` kulcsszót és a pontosvesszőt, automatikusan a kifejezés értékével tér vissza.
    * *Szintaxis:* `(paraméterek) => kifejezés`

#### Paraméter-megadási szabályok:
* Ha nincs paraméter, kötelező az üres zárójel: `() => Console.WriteLine();`
* Ha egy paraméter van, a zárójel elhagyható: `x => x * x`
* Ha több paraméter van, kötelező a kerek zárójel és a vesszővel elválasztás: `(x, y) => x + y`
* A fordító szinte mindig képes kikövetkeztetni a paraméterek típusait, így azokat nem szükséges kiírni, de ha szükséges, explicit is megadható: `(int x, string s) => s.Length > x.`

**Kódpélda a használatukra:**

```csharp
using System;

public class LambdaBemutato
{
    // Saját delegate típus definiálása
    public delegate bool MuveletDelegate(int a, int b);

    public static void Main()
    {
        // 1. Utasítás lambda eltárolása delegate változóban
        MuveletDelegate utasitasLambda = (int x, int y) => {
            int osszeg = x + y;
            return osszeg > 10;
        };

        // 2. Kifejezés lambda (sokkal tömörebb, a fordító a típusokat is kitalálja)
        MuveletDelegate kifejezesLambda = (x, y) => x + y > 10;
 
        // Meghívásuk pontosan úgy történik, mint egy normál függvénynél:
        bool eredmeny = kifejezesLambda(5, 8); // true
    }
}
```
### 5. Beépített Delegate típusok (Func és Action)

A C#-ban a leggyakoribb esetekre nem kell saját delegate típusokat létrehoznunk, mert a .NET biztosít előre definiált, generikus delegate-eket.

#### 1. Func<...> generikus delegate
Olyan függvények hivatkozására szolgál, amelyeknek **van visszatérési értékük**.

* A generikus paraméterlistában mindig a legutolsó paraméter jelöli a visszatérési típust, az azt megelőzőek pedig a bemenő paraméterek típusait.
* `Func<int>` $\rightarrow$ Nincs paramétere, `int` értékkel tér vissza.
* `Func<string, int, bool>` $\rightarrow$ Két paramétere van (`string` és `int`), és `bool` értékkel tér vissza.

#### 2. Action<...> generikus delegate
Olyan függvények hivatkozására szolgál, amelyeknek **nincs visszatérési értékük** (`void`).

* `Action` $\rightarrow$ Paraméter nélküli, `void` metódus.
* `Action<string, double>` $\rightarrow$ Egy `string` és egy `double` paramétert fogad, visszatérése `void`.

**Kódpélda magyarázattal:**

```csharp
using System;
 
public class BeepitettDelegatePelda
{
    public static void Main()
    {
        // Func példa: Két int-et kap, bool-t ad vissza.
        // A lambda megvizsgálja, hogy az első osztható-e a másodikkal.
        Func<int, int, bool> oszthatoE = (x, y) => x % y == 0;
        
        bool res = oszthatoE(10, 2); // true-val fog visszatérni
 
        // Action példa: Egy stringet kap, kiírja a konzolra, nincs visszatérése (void)
        Action<string> udvozles = nev => Console.WriteLine($"Szia {nev}!");
        
        udvozles("Péter"); // Konzolon megjelenik: Szia Péter!
    }
}
```

### 6. Változóbefogás és Lezárás (Variable Capturing, Closure)

A normál függvények csak a paramétereikkel és a saját lokális változóikkal tudnak dolgozni. A lambda függvények viszont képesek látni és használni a definíciójuk környezetében (kontextusában) lévő lokális változókat és tagváltozókat is. Ezt a jelenséget nevezzük *variable capturing*-nek vagy *closure*-nek.

#### Hogyan működik a háttérben? (ZH Elmélet!)
A működés meglepő tulajdonsága, hogy a lambda akkor is eléri a befogott lokális változót, ha a befogadó függvény futása már rég befejeződött (és a lokális változóknak a stack-ből elvileg már meg kellett volna semmisülniük). 

A háttérben a fordító egy rejtett anonim osztályt generál, a befogott változókat átmásolja ennek az osztálynak a tagváltozóiba (a heap-re), a lambda kifejezést pedig ennek az osztálynak a metódusává alakítja. Így a változók élettartama meghosszabbodik.

**Kódpélda részletes magyarázattal:**

```csharp
using System;

public class IncrementFactory
{
    // A metódus egy olyan függvénnyel tér vissza, ami egy int-et vár és int-et ad vissza
    public Func<int, int> GetIncrementFunc(int delta) 
    {
        // A lambda kifejezés befogja a 'delta' paramétert (ami lokális változónak számít)
        return n => n + delta; 
    }
}

public class Program
{
    public static void Main()
    {
        IncrementFactory factory = new IncrementFactory();
        
        // Kérünk egy függvényt, ami a mindenkori paraméterét 20-szal növeli meg
        Func<int, int> hozzaadoFuggveny = factory.GetIncrementFunc(20);
        
        // Ezen a ponton a GetIncrementFunc már lefutott, a delta=20 változó elvileg megszűnt 
        // A closure miatt azonban a háttérben az érték megőrződött 
        int eredmeny = hozzaadoFuggveny(100); // 100 + 20 = 120
        Console.WriteLine(eredmeny);
    }
}
```

### 7. LINQ (Language-Integrated Query)

A LINQ gyűjtemények (pl. `List`, tömbök) kezelését (szűrés, rendezés, transzformáció) radikálisan leegyszerűsítő eszköz. Két szintaxisa létezik, a tárgy keretében kizárólag a **fluent szintaxist** (egymás után láncolt metódushívások pont operátorral) kell tudni.

#### Alapvető LINQ metódusok (mindegyik lambdát vár paraméterként):
* **.Where(...) (Szűrés):** Egy `Func<T, bool>` típusú predikátumot vár. Csak azokat az elemeket engedi tovább, amelyekre a lambda kifejezés `true` értéket ad vissza.
* **.OrderBy(...) (Rendezés):** A megadott tulajdonság (kifejezés) alapján növekvő sorrendbe rendezi a gyűjtemény elemeit.
* **.Select(...) (Projekció / Transzformáció):** A gyűjtemény minden egyes elemét átalakítja egy új formátumú/típusú objektummá a lambda kifejezés alapján.

#### Halasztott végrehajtás (Deferred Execution - Fontos elmélet!)
A LINQ metódusok nem hajtódnak végre azonnal, hanem egy `IEnumerable<T>` típusú iterátort (lekérdezési tervet) adnak vissza. A tényleges elemfeldolgozás csak akkor történik meg, amikor elkezdjük bejárni a gyűjteményt (pl. egy `foreach` ciklussal). Ha azonnal kézzelfogható listát vagy tömböt akarunk kapni, meg kell hívni a `.ToList()` vagy `.ToArray()` lezáró metódusokat.

**Kódpélda láncolt LINQ műveletekre:**

```csharp
using System;
using System.Collections.Generic;
using System.Linq; // A LINQ kiterjesztő metódusok miatt kötelező beimportálni!

public class LinqPelda
{
    public static void Main()
    {
        List<string> gyumolcsok = new List<string>() { "apple", "grape", "peach", "banana", "pineapple" };
 
        // Komplett láncolt lekérdezés fluent szintaxissal:
        List<string> eredmeny = gyumolcsok
            .Where(f => f.Length <= 5)      // 1. Csak a legfeljebb 5 betűsek maradnak
            .OrderBy(f => f.Length)         // 2. Sorbarendezés hossz szerint 
            .Select(f => f.ToUpper())       // 3. Transzformáció: Minden szót csupa nagybetűssé alakítunk 
            .ToList();                      // 4. Azonnali végrehajtás kikényszerítése új listába
 
        // Az 'eredmeny' tartalma ekkor: "APPLE", "GRAPE", "PEACH"
        foreach (var g in eredmeny) {
            Console.WriteLine(g);
        }
    }
}
```
### 8. Részleges osztályok (partial class)

A `partial` kulcsszó lehetővé teszi, hogy egyetlen osztály definícióját több különálló `.cs` forrásfájlra bontsuk szét.

#### Szabályok:
* **Kötelező kulcsszó:** Mindegyik fájlban, ahol az osztály egy darabja van, ki kell tenni a `partial class` jelölést, különben fordítási hibát kapunk.
* **Összefésülés:** A fordítás során a compiler automatikusan egyetlen egységes osztállyá fésüli össze a darabkákat. Bármelyik fájlban deklarált privát változót a másik fájlban lévő metódusok láthatják és módosíthatják.
* **Fő alkalmazási területe:** A generált kód és a kézzel írt kód tiszta különválasztása. Például a grafikus felvezérlők tervezőprogramjai (Designer) által automatikusan generált kód külön fájlba megy, így a mi kézzel írt logikánk egy másik fájlban nem fog felülíródni.

**Kódpélda:**

```csharp
// Fájl 1: AutoGeneralt_Adatok.cs
public partial class User { 
    private string name; 
    private int age;
}

// Fájl 2: KezzelIrt_Logika.cs
public partial class User { 
    public void DisplayUser() {
        // Teljesen legális: eléri a másik fájlban definiált privát 'name' változót! 
        Console.WriteLine($"Felhasználó: {name}"); 
    }
}
```

### 9. Kifejezéstörzsű tagok (Expression-Bodied Members)

Ha egy függvény vagy egy tulajdonság (`Property`) törzse egyetlen egy kifejezésből áll, a hagyományos kapcsos zárójeles `{}` + `return` szintaxis helyett használhatjuk a `=>` operátort a kód tömörítésére.

#### ZH Buktató / Fontos elmélet!
Bár itt is a `=>` szimbólumot használjuk, ennek **semmi köze sincs a lambda kifejezésekhez!** Ez pusztán egy szintaktikai édesítőszer (szintaktikai csavar), ahol a C# nyelv ugyanazt a tokent egy teljesen más célra használja újra.

#### Kifejezéstörzsű tulajdonságok variációi:
* Sima `getter` esetén elhagyható a `{ get { return ... } }` kód.
* Ha egy tulajdonságnak csak `getter`-je van (read-only property), még a `get` kulcsszó is lehagyható. Ezt az különbözteti meg a metódustól, hogy a neve mögé nem kell kitenni a kerek zárójeleket `()`.

**Kódpélda magyarázattal:**

```csharp
using System;
 
public class Kutya
{
    private string nev;
    private int szuletesiEv;
 
    public Kutya(string nev, int ev) {
        this.nev = nev;
        this.szuletesiEv = ev;
    }
 
    // 1. Kifejezéstörzsű METÓDUS (void, nincs return)
    public void Ugat() => Console.WriteLine($"{nev} mondja: VAU!");
 
    // 2. Kifejezéstörzsű METÓDUS (értékkel tér vissza, elmarad a {} és a return)
    public int KorSzamitas() => DateTime.Now.Year - szuletesiEv;
 
    // 3. Kifejezéstörzsű Read-Only TULAJDONSÁG (Property)
    // Nincs () zárójel, nincs 'get' kulcsszó, tiszta és tömör
    public bool IdosE => KorSzamitas() > 10; 
}
```

### 10. Objektuminicializáló (Object Initializer)

Az objektuminicializáló szintaxis lehetővé teszi, hogy egy objektum létrehozásakor (a konstruktorhívás után közvetlenül) kapcsos zárójelek között értéket adjunk az objektum publikus tulajdonságainak vagy tagváltozóinak.

#### Előnyei és szabályai:
* **Konstruktor után fut:** A kapcsos zárójelbe tett értékadások szigorúan a konstruktor lefutása után történnek meg.
* **Egyetlen kifejezésnek számít:** Az egész folyamat (példányosítás + adatok feltöltése) egyetlen utasításnak minősül.
* **Nincs szükség segédváltozóra:** Mivel egyetlen kifejezés, így egy metódus paramétereként közvetlenül is átadhatjuk az újonnan létrehozott és azonnal inicializált objektumot, anélkül, hogy előtte egy lokális változóba el kellene mentenünk.
* **Vessző szabály:** Az utolsó tulajdonság megadása után kitett vessző opcionális, a fordítót nem zavarja, ha ott hagyjuk.

**Kódpélda részletes magyarázattal:**

```csharp
using System;
 
public class Person {
    public string Name { get; set; } // Publikus property-k
    public int Age { get; set; }
}
 
public class Program {
    public static void PrintPersonInfo(Person p) { 
        Console.WriteLine($"{p.Name} is {p.Age} years old.");
    }
 
    public static void Main() {
        // Hagyományos, hosszadalmas megoldás segédváltozóval:
        Person p1 = new Person();
        p1.Name = "Luke";
        p1.Age = 17;
        PrintPersonInfo(p1);
 
        // Modern megoldás Objektuminicializálóval, közvetlenül argumentumként átadva!
        // Nincs felesleges lokális változó, a beállítás a háttérben a konstruktor után történik.
        PrintPersonInfo(new Person() { Name = "Luke", Age = 17 });
    }
}
```