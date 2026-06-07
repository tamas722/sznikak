# 5. Előadás: Párhuzamos programozás alapjai

## 1. Alapfogalmak: Folyamat (Process) vs. Szál (Thread)
A vizsgán az egyik leggyakoribb beugró kérdés e kettő közötti különbség.

* **Folyamat (Process):**
    * A program egy betöltött, futó példánya.
    * Védelmi egység, amely elszigeteli a programokat egymástól és az operációs rendszertől (OS).
    * Saját virtuális memória címtartománnyal és saját rendszererőforrásokkal rendelkezik.
    * Legalább egy (fő) szál fut benne.
* **Szál (Thread):**
    * A tényleges futási/ütemezési egység.
    * Sokkal kisebb az erőforrásigénye, mint egy folyamatnak.
    * Egy folyamaton belül **megosztják a memóriát** (a globális, statikus és dinamikus változók közösek), így könnyen kommunikálnak.
    * Minden szálnak saját **"Stack"-je** (hívási verme) van, így a lokális változók privátok.



### A többszálúság előnyei és hátrányai
* **Előnyök:** * Jobb CPU kihasználás (I/O műveletek közbeni várakozás csökkentése).
    * Megakadályozza a grafikus felület (GUI) "befagyását" hosszú műveletek alatt.
    * Szervereknél rövidebb átlagos válaszidő (párhuzamos kérések).
* **Hátrányok:** * Megnövekedett komplexitás.
    * Nehezen kinyomozható, nem determinisztikus hibák (szinkronizációs problémák).
    * Túl sok szál esetén a "kontextusváltások" (context switch) terhelik a CPU-t.

---

## 2. Szálak indítása és kezelése a .NET-ben
A szálak ütemezését az operációs rendszer végzi. Modern rendszerekben ez **preemptív**: az ütemező elveszi a futási jogot a száltól, ha lejárt az időszelete, vagy egy nagyobb prioritású szál érkezik.

### Alapvető szálindítás (Kódpélda)
A .NET-ben a `System.Threading.Thread` osztályt használjuk.

```csharp
using System;
using System.Threading;
 
class Program
{
    static void Main()
    {
        // 1. Szál létrehozása és a szálfüggvény (ThreadStart delegate) megadása
        Thread t = new Thread(WriteY);
        
        // 2. Szál elindítása. E nélkül az OS nem ütemezi!
        t.Start();
 
        // A főszál is dolgozik párhuzamosan
        while (true) Console.Write("x");
    }
 
    // Ez a szálfüggvény, amit a háttérszál végrehajt
    static void WriteY()
    {
        while (true) Console.Write("y");
    }
}
```

*Magyarázat:* A kód futtatásakor az "x" és "y" karakterek keveredve jelennek meg a konzolon, ami bizonyítja a látszólagos párhuzamos futást az időszeletes ütemezés miatt. Paramétert is adhatunk át a szálnak a `ParameterizedThreadStart` delegate segítségével, ekkor a paramétert a `Start(param)` metódusnak kell átadni.

### Előtérszálak vs. Háttérszálak
* **Előtérszálak:** Alapértelmezésben minden létrehozott szál az. A program (folyamat) csak akkor lép ki, ha minden előtérszál befejezte a futását.
* **Háttérszálak:** Ha beállítjuk a `t.IsBackground = true` tulajdonságot, a szál háttérszál lesz. Ez azonnal terminálódik, amint a főszál (és az összes többi előtérszál) kilép.

---

## 3. Szinkronizáció és Kölcsönös Kizárás (Mutual Exclusion)
Ez a legfontosabb vizsgatéma! Mivel a szálak közös memórián osztoznak, ha többen egyszerre próbálnak módosítani egy erőforrást (pl. egy változót vagy listát), az inkonzisztens állapotot eredményezhet. 



**Probléma:** Például a `++` operátor vagy egy Lista elemének hozzáadása nem **atomi művelet**. Ha az OS pont a beolvasás és a visszaírás között veszi el a futási jogot, az adat elveszhet vagy felülíródhat. 
**Megoldás:** Kölcsönös kizárás (Mutual exclusion). Biztosítani kell, hogy egy adott időben csak egy szál férjen hozzá a megosztott erőforráshoz. Az aktív várakozás (pl. `while(flag)`) rossz módja a szinkronizációnak, mert feleslegesen lefoglalja a processzort.

### A lock kulcsszó használata (Kódpélda)
A .NET a `lock` kulcsszót (amely a `Monitor.Enter` és `Exit` utasításokra fordul) biztosítja a kritikus szakaszok védelmére.

```csharp
class ThreadSafeClass
{
    static bool done; // Közös, védendő erőforrás
    
    // Szinkronizációs objektum (zár). Csak referencia típus (osztály) lehet!
    static readonly object syncObject = new object(); 
 
    static void Main()
    {
        new Thread(Run).Start();
        new Thread(Run).Start();
    }
 
    static void Run()
    {
        // Kritikus szakasz kezdete. 
        // Ha valaki már bent van, a többi szál itt blokkolódik (nem fogyaszt CPU-t).
        lock (syncObject) 
        {
            if (!done) {
                done = true;
                Console.WriteLine("Winner");
            }
        } // Zár feloldása. A várakozó szálak egyike bemehet.
    }
}
```

### Fontos szabályok a zároláshoz
* **Kerüld a statikus zárat példányváltozókhoz:** Soha ne használj statikus zárat (`static object`) nem statikus (példány szintű) változók védelmére, mert az felesleges szűk keresztmetszetet okoz.
* **Típus szerinti védelem:** Statikus változókat statikus zárral, példányváltozókat példányzárral védünk.

### Atomitás és Volatile
* **Interlocked osztály:** Egyszerű matematikai műveletekhez (pl. növelés, összeadás) felesleges a `lock`, használhatók az operációs rendszer által támogatott, szupergyors, nem blokkoló atomi utasítások: `Interlocked.Increment(ref sum)`, `Interlocked.Exchange(...)`.
* **Volatile kulcsszó:** A fordító optimalizálásból felcserélheti az írási/olvasási sorrendet, vagy regiszterben cache-elheti a változókat, így más szálak elavult értéket láthatnak. A `volatile` rákényszeríti a rendszert, hogy az írás/olvasás azonnal, közvetlenül a memóriából történjen. 
    * *Megjegyzés:* `lock` blokk használata esetén ez felesleges, mert a `lock` ezt automatikusan kezeli.

---

## 4. Szálak közötti jelzések (Signaling)
Amikor egy szálnak arra kell várnia, hogy egy másik szál végezzen valamilyen feladattal, jelzéseket használunk. Erre szolgálnak a `WaitHandle` osztály leszármazottai.

### AutoResetEvent
Olyan, mint egy forgókapus beléptető:
* Ha a kapu zárva van (`WaitOne()`), a szálak sorban állnak és blokkolódnak.
* Ha valaki meghívja a `Set()` metódust, a kapu kinyílik, egyetlen szálat átenged, majd automatikusan visszacsukódik (nem jelzett állapotba kerül).



**AutoResetEvent (Kódpélda):**
```csharp
class SimpleEventDemo
{
    // false = kezdetben nem jelzett (zárt kapu)
    static AutoResetEvent wh = new AutoResetEvent(false); 
 
    static void Main()
    {
        new Thread(Run).Start();
        Thread.Sleep(1000); // Főszál "dolgozik"
        wh.Set(); // Ébresztőt küld a várakozó szálnak
    }
 
    static void Run()
    {
        Console.WriteLine("Várunk értesítésre ...");
        wh.WaitOne(); // A szál itt megáll (blokkol) CPU fogyasztás nélkül, amíg Set() nem jön
        Console.WriteLine("Az értesítés megérkezett.");
    }
}
```
### ManualResetEvent
Olyan, mint egy vasúti sorompó. Ha a `Set()` kinyitja, **minden** várakozó szál átmehet, és nyitva is marad addig, amíg valaki kézzel meg nem hívja rá a `Reset()` metódust.

---

## 5. Szálak kiléptetése és kivételek
* **Kivételkezelés:** Ha egy háttérszálban kezeletlen kivétel lép fel, a teljes alkalmazás összeomlik! A `try-catch` blokkot mindig a szálfüggvényen (a futó szálon) belül kell elhelyezni.
* **Kiléptetés:**
    * **SOHA NE HASZNÁLD:** `Thread.Abort()`. Ez egy brutális megoldás, ami nyitva hagyhat fájlokat és zárakat; a .NET kivezette.
    * **Helyes megoldások:**
        * **Polling:** Egy közös `volatile bool exit` jelzőváltozó időnkénti lekérdezése.
        * **Thread.Interrupt():** Ha a szál folyamatosan várakozik (pl. `Sleep` vagy `WaitOne`), ez kivételt dob a szálban, amit elkapva tisztán kiléphetünk.

---

## 6. Architekturális problémák és megoldások
* **Holtpont (Deadlock):** Két szál kölcsönösen egymásra vár. 
    * *Elkerülése:* A zárakat/erőforrásokat minden szálban szigorúan **azonos sorrendben** kell lefoglalni, vagy időkorlátot (`timeout`) kell alkalmazni.
* **Thread-pool (Szálkészlet):** A rendszer előre elindít egy "medencényi" szálat. A feladatok ezeket használják, így megspóroljuk a szálak létrehozásának költségeit. 
    * *Megjegyzés:* Hosszan blokkoló kódokat (pl. végtelen várakozást) tilos ide tenni!
* **Thread Safe Interface Pattern:** A publikus metódusokban végezzük a zárolást, és a tényleges logikát hívható privát metódusokba tesszük, amelyek már nem használnak `lock`-ot (feltételezik a védelmet).



---

## 7. Többszálú WinUI (GUI) alkalmazások
**A legfontosabb szabály:** A grafikus felület elemeit csak abból a szálból szabad elérni, amelyik létrehozta őket (a főszál). Ha háttérszálból módosítasz egy GUI elemet, a program kivételt dob.

* **Megoldás: DispatcherQueue:** A `TryEnqueue` metódusnak átadott kódrészletet a rendszer a megfelelő GUI szálon fogja végrehajtani.

**Kódpélda:**
```csharp
// Háttérszálban futó kód:
void HaterszalFuggveny()
{
    string eredmeny = "Kész a hosszú számítás!";
    
    // A GUI frissítését "átadjuk" a főszálnak a DispatcherQueue-val
    this.DispatcherQueue.TryEnqueue(() =>
    {
        // Ez a kód már a GUI szálon fut, biztonságos a módosítás
        eredmenyTextBox.Text = eredmeny; 
    });
}
```

2. További zárolási konstrukciók (Mutex, Semaphore, ReaderWriterLock)
Bár a lock a leggyakoribb, bizonyos esetekben másra van szükség.
•	Mutex: Működése hasonló a lock-hoz (kölcsönös kizárás), de sokkal lassabb . Cserébe folyamatok (process-ek) között is használható. Tipikus példa: megakadályozni, hogy egy programból egyszerre több példány induljon el. Lefoglalni a WaitOne(), elengedni a ReleaseMutex() metódussal lehet. 
•	Semaphore: A Mutexszel ellentétben nem egy, hanem maximum N darab hozzáférést engedélyez egyszerre az erőforráshoz . Ha a keret betelt, a többi szál blokkolódik. Lefoglalni a WaitOne(), elengedni a ReleaseSemaphore() metódussal lehet . 
•	ReaderWriterLock (és Slim): Olyan helyzetekre optimalizálták, ahol sok az olvasás, de ritka az írás. Egyszerre több olvasó szálat is beenged, de ha egy író szál érkezik, az mindenkit kizár (az olvasókat is). 
Kódpélda Semaphore használatára:
```csharp
class SemaphoreDemo
{
    // Egyszerre max 2 szál mehet be a kritikus szakaszba
    static Semaphore sem = new Semaphore(2, 2);

    static void Run()
    {
        Console.WriteLine("Várakozás a belépésre...");
        sem.WaitOne(); // Blokkol, ha már 2 szál bent van
        
        Console.WriteLine("Bent vagyok!");
        Thread.Sleep(2000); // Erőforrás használata
        
        sem.Release(); // Kilépés, hely felszabadítása a várakozóknak
    }
}
```

## 3. Több esemény bevárása: WaitAny és WaitAll
Ha nem egyetlen eseményre (pl. `AutoResetEvent` vagy `ManualResetEvent`) vársz, hanem többre, a `WaitHandle` ősosztály statikus metódusait használhatod. Be kell nekik adni a várt események tömbjét.

* **WaitAll:** Akkor enged tovább (nem blokkol tovább), ha a tömbben lévő **összes** leíró jelzett (szabad) állapotba került.
* **WaitAny:** Akkor enged tovább, ha a tömbben lévő események közül **bármelyik** (legalább egy) jelzett állapotba került.



**Kódpélda:**
```csharp
AutoResetEvent event1 = new AutoResetEvent(false);
AutoResetEvent event2 = new AutoResetEvent(false);
 
WaitHandle[] handles = new WaitHandle[] { event1, event2 };
 
// A szál itt megáll, és csak akkor indul el, ha MINDKÉT event Set()-et kap:
WaitHandle.WaitAll(handles);
```

## 4. Szálak állapotai (Thread States)
Egy szál életciklusa során különböző állapotokon megy keresztül:

* **Unstarted:** A szál létrejött (`new Thread()`), de még nem hívták meg rá a `Start()` metódust.
* **Running:** A szál fut, az operációs rendszer ütemezi.
* **WaitSleepJoin:** A szál blokkolt állapotban van (pl. `Sleep()`, `Join()`, `lock`, `WaitOne()` miatt). Nem terheli a processzort. Kiléphet ebből, ha a feltétel teljesül, lejár a timeout, vagy megszakítják (`Interrupt`).
* **Stopped:** A szálfüggvény befejeződött, a szál végleg leállt.



---

## 5. Prioritások és Ütemezés
A rendszer az eredő prioritást két dolog alapján számolja ki (0-31 közötti érték): a folyamat prioritása (`ProcessPriorityClass`) és a szál prioritása (`ThreadPriority`) alapján. Mindkettő alapértelmezett értéke a `Normal`.

**Veszélyes játék a RealTime prioritás használata!** Ezzel arra kéred az OS-t, hogy soha ne vegye el a futási jogot a száltól. Ha egy ilyen szál végtelen ciklusba kerül, az egész gép "befagyhat" (az egér is megállhat, a lemez nem tud írni), és csak a reset gomb segíthet.

**Példa a kódra:**
```csharp
// Folyamat prioritásának megemelése
Process.GetCurrentProcess().PriorityClass = ProcessPriorityClass.High;
 
// Aktuális szál prioritásának maximálisra állítása
Thread.CurrentThread.Priority = ThreadPriority.Highest;
```