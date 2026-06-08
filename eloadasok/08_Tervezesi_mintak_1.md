# 6. Előadás: Tervezési minták és a Template Method

## I. Alapfogalmak
A tervezési minta (design pattern) egy gyakran előforduló programtervezési problémát, annak környezetét és a megoldás magját írja le. A legismertebb mintákat a "Gang of Four" (GoF) katalogizálta.

### A minta leírásának négy alapeleme:
1. **Név (Pattern Name):** Azonosítja a mintát, segíti a fejlesztők közötti kommunikációt.
2. **Probléma:** Bemutatja a környezetet és a megoldandó feladatot.
3. **Megoldás:** Az elemeket, kapcsolatokat és felelősségeket írja le (absztrakt módon).
4. **Következmények:** Tapasztalatok és hatások, segítve a megfelelő minta kiválasztását.

### Kategóriák:
* **Hatókör szerint:** Architekturális minták (globális felépítés), Tervezési minták (rendszeren belüli megoldások), Idiómák (nyelvspecifikus megoldások).
* **GoF kategóriák:**
    * **Létrehozási (Creational):** Factory Method, Abstract Factory, Singleton.
    * **Strukturális (Structural):** Adapter, Composite, Façade, Proxy.
    * **Viselkedési (Behavioral):** Template Method, Strategy, Observer, Command, Memento.

---

## II. Tervezési célok és SOLID alapelvek
A minták segítenek a szoftverfejlesztés három fő céljának elérésében:
1. **Változtathatóság/Bővíthetőség:** Külön választjuk a változó és változatlan részeket.
2. **Újrafelhasználhatóság:** Lazán csatolt rendszerelemek létrehozása.
3. **Unit tesztelhetőség:** A függőségek interfészeken keresztüli kezelése (mock-olhatóság).

### SOLID alapelvek:
* **Open/Closed Principle (O):** Az osztály legyen kiterjeszthető (Open), de forráskódja ne legyen módosítható (Closed).
* **Single Responsibility Principle (S):** Egy osztály pontosan egy dologért legyen felelős.

---

## III. Template Method (Sablonmetódus)
Egy viselkedési minta, amely egy műveleten belül definiálja az algoritmus vázát, a lépések konkrét implementálását pedig a leszármazottakra bízza.



### Jellemzői és következményei:
* **DRY elv (Kódduplikáció elkerülése):** A közös részek ősosztályba kerülnek.
* **Hook függvények:** Kiterjesztési pontok, amiket a leszármazott felülírhat.
* **Hátrány:** Rugalmatlanság (fordítási időben dől el a viselkedés) és "kombinatorikus robbanás" veszélye (túl sok kombináció esetén túl sok osztály kell).

### Példa: Adatfeldolgozás
```csharp
// 1. Az ősosztály definiálja az algoritmus vázát
abstract class DataProcessorBase
{
   // Ez a Template Method! A struktúra (a váz) fix.
   public void Run() 
   {
       InitCompression(); 
       byte[] inputData = new byte[] { 1, 2, 3 }; 
       byte[] compressed = CompressData(inputData); 
       CloseCompression(); 
   }

   protected abstract void InitCompression();
   protected abstract byte[] CompressData(byte[] data);
   protected abstract void CloseCompression();
}
 
// 2. A leszármazott osztály megadja a specifikus viselkedést
class ZipDataProcessor : DataProcessorBase
{
   protected override void InitCompression() => Console.WriteLine("ZIP inicializálása...");
   protected override byte[] CompressData(byte[] data) 
   {
       Console.WriteLine("Tömörítés ZIP algoritmussal...");
       return data; 
   }
   protected override void CloseCompression() => Console.WriteLine("ZIP lezárása.");
}
```

## IV. Strategy (Stratégia) Minta
A Strategy egy viselkedési minta, amelynek célja algoritmusok vagy viselkedések egy csoportjának egységbe zárása, és egymással való szabad cserélhetősége. A kliens számára a használt stratégia részletei láthatatlanok maradnak.

[Image of Strategy design pattern class diagram]

### Jellemzői és következményei:
* **Delegáció leszármazás helyett:** A kiterjesztést nem ősosztályból való származtatással, hanem egy interfészen keresztül hivatkozott külső osztálynak való átadással (delegálással) érjük el.
* **Futásidejű csere:** A stratégiák akár futás közben is szabadon módosíthatók.
* **Nincs kombinatorikus robbanás:** Nem kell minden lehetséges keresztkombinációhoz osztályt létrehozni; a stratégiákat paraméterként adjuk át.
* **Segíti a Unit tesztelhetőséget:** Teszteléskor "dummy" (üres vagy teszt) stratégiákat adhatunk át a fő osztálynak, így izoláltan vizsgálhatjuk a működést.

---

### Példa: Rendezési stratégia
A kliens (Client) nem akarja tudni, hogyan zajlik a rendezés, csak hívni akar egy `Sort` metódust. A konkrét rendezési logikát külső osztályokba szervezzük.

```csharp
// 1. Létrehozzuk a Stratégia Interfészt
interface ISortStrategy
{
    void Sort(string[] items);
}
 
// 2. Megírjuk a Konkrét Stratégiákat
class QuickSort : ISortStrategy
{
    public void Sort(string[] items)
    {
        Console.WriteLine("Rendezés QuickSort segítségével...");
    }
}
 
class HeapSort : ISortStrategy
{
    public void Sort(string[] items)
    {
        Console.WriteLine("Rendezés HeapSort segítségével...");
    }
}
 
// 3. A Kliens, ami használja a Stratégiát
class Client
{
    private ISortStrategy sortStrategy; // Interfész típusú hivatkozás
 
    // Lehetőség a stratégia futásidejű beállítására
    public void SetSortStrategy(ISortStrategy strategy)
    {
        this.sortStrategy = strategy;
    }
 
    public void Sort(string[] items)
    {
        sortStrategy.Sort(items); // A munka delegálása a stratégiának
    }
}
 
// Használat a gyakorlatban:
class Program
{
    static void Main()
    {
        Client client = new Client();
        string[] items = { "körte", "alma", "szilva" };
 
        client.SetSortStrategy(new QuickSort()); // Stratégia beállítása
        client.Sort(items); // QuickSort fog lefutni
 
        client.SetSortStrategy(new HeapSort()); // Stratégia cseréje futás közben!
        client.Sort(items); // Most már a HeapSort fut le
    }
}
```

## V. Modern Kiterjeszthetőség (C# Lambda és Delegate)
Bár a `Strategy` minta a klasszikus GoF megoldás, a modernebb nyelvekben (mint a C#) a kiterjesztést gyakran delegáltakkal (`Func`, `Action`) és lambda kifejezésekkel oldják meg. Ennek lényege, hogy komplett osztályok és interfészek írása helyett egyszerűen egy függvényt adunk át paraméterként, amely tartalmazza a kívánt viselkedést.



### Modern C# alternatíva (a Strategy minta helyett)
```csharp
class DataProcessor
{
    // A Func<bool> egy metódusreferencia, ami visszaad egy bool értéket
    private Func<bool> isCancelled;
 
    public DataProcessor(Func<bool> isCancelled)
    {
        this.isCancelled = isCancelled;
    }
 
    public void Run()
    {
        // Ha adtak át ellenőrző függvényt, meghívjuk azt
        if (isCancelled != null && isCancelled())
        {
            Console.WriteLine("Folyamat megszakítva.");
            return;
        }
    }
}
 
// Használat lambda kifejezéssel:
// Itt adjuk át a logikát: "() => false" jelentése, hogy sosem szakítjuk meg
var processor = new DataProcessor(() => false);
processor.Run();
```
