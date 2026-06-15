# 3. Szálkezelés – Írjon az alábbiaknak megfelelő C# nyelvű konzol alkalmazást. (20p)

Az alkalmazás **fő szála** (ez maga a `Main` függvényt hívó szál) indítson el **két azonos módon viselkedő** (lásd következő pont) munkaszálat.

Egy munkaszál az indítás után egy ciklusban generáljon véletlen számokat **0..100** között, és ezeket írja bele egy **közös stack** (típusa `Stack<int>`) gyűjteménybe a `Push` művelet felhasználásával. A munkaszál a ciklusból akkor lépjen ki és fejezze be a futását, amikor a generált véletlen szám értéke **99**.

A **fő szál** a munkaszálak indítása után egy ciklusban folyamatosan olvassa ki a munkaszálak által a közös stackbe írt véletlen számokat (`Pop` művelet), és ezek értékét írja ki a konzolra. Tartsa szem előtt, hogy ekkor a munkaszálak még futhatnak! A fő szál a munkaszálak által stackbe írt számokat a **lehető leghamarabb** írja ki (hatékony megoldásra törekedjen, kerülje az aktív, valamint a felesleges várakozásokat)! Azt, hogy a stack üres-e, a `Count` property/tulajdonság segítségével tudja ellenőrizni.

A fő szál akkor fejezze be az adatok feldolgozását, amikor a munkaszálak **kiléptek** és a stack **már kiürült**. Némiképpen kevesebb pontszámért olyan megoldást is választhat, mely nem gondoskodik arról, hogy a munkaszálak kilépését követően a stackben levő adatok feldolgozásra kerüljenek.

A feladatban szereplő intervallumok esetében szabadon választhat inkluzív és exkluzív megközelítést.

**Segítség:** Véletlen egész számok generálásához egy `Random` osztálybeli objektumot kell létrehozni (ezt csak egyszer), és a `Next(maximum_érték)` művelettel az egyes számokat generálni.

```csharp
using System;
using System.Collections.Generic;
using System.Threading;

namespace SzalkezelesFeladat
{
    class Program
    {
        // Közös erőforrások a szálak között
        static Stack<int> stack = new Stack<int>();
        static object lockObj = new object();
        static Random rnd = new Random();
        static int activeWorkers = 2;

        // AutoResetEvent: kezdetben nem jelzett (false), így a WaitOne() blokkolni fog
        static AutoResetEvent autoEvent = new AutoResetEvent(false);

        static void Main(string[] args)
        {
            // Két azonos módon viselkedő munkaszál létrehozása és indítása
            Thread t1 = new Thread(Worker);
            Thread t2 = new Thread(Worker);

            t1.Start();
            t2.Start();

            // A főszál folyamatosan olvassa ki és dolgozza fel az adatokat
            while (true)
            {
                // Szinkron várakozás aktív polling nélkül. 
                // Ha a stack üres, a szál elalszik, amíg egy munkaszál nem jelez (Set).
                autoEvent.WaitOne();

                lock (lockObj)
                {
                    // Mivel a szálak gyorsak, több elem is bekerülhetett egyetlen jelzés alatt.
                    // Ezért a felébredés után az összes éppen elérhető elemet kiolvassuk.
                    while (stack.Count > 0)
                    {
                        int number = stack.Pop();
                        Console.WriteLine($"Főszál kiolvasta a stackből: {number}");
                    }

                    // A főszál akkor fejezi be a futását, ha a munkaszálak már leálltak 
                    // ÉS az összes adatot feldolgoztuk (a stack már kiürült).
                    if (activeWorkers == 0 && stack.Count == 0)
                    {
                        break; // Kilépés a feldolgozó ciklusból
                    }
                }
            }

            // A biztonság kedvéért a főszál megvárja a munkaszálak fizikai leállását is
            t1.Join();
            t2.Join();

            Console.WriteLine("A program sikeresen befejeződött, minden adat feldolgozásra került.");
        }

        static void Worker()
        {
            while (true)
            {
                int number;
                
                // Számgenerálás és Push művelet szinkronizált védelme
                lock (lockObj)
                {
                    number = rnd.Next(0, 101); // 0..100 közötti véletlen egész szám
                    stack.Push(number);
                }
                
                // Jelzés a főszálnak, hogy új adat érkezett a stackbe
                autoEvent.Set();

                // Ha a generált szám 99, a munkaszál befejezi a futását
                if (number == 99)
                {
                    lock (lockObj)
                    {
                        activeWorkers--; // Aktív munkaszálak számának csökkentése
                    }
                    
                    // Utolsó jelzés leadása, hogy ha a főszál már aludna, 
                    // mindenképpen ébredjen fel ellenőrizni az activeWorkers állapotát.
                    autoEvent.Set(); 
                    break;
                }
            }
        }
    }
}
```
