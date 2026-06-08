# 3. Szálkezelés – Írjon az alábbiaknak megfelelő C# nyelvű konzol alkalmazást. (20p)

Az alkalmazás **fő szála** (ez maga a `Main` függvényt hívó szál) indítson el **két azonos módon viselkedő** (lásd következő pont) munkaszálat.

Egy munkaszál az indítás után egy ciklusban generáljon véletlen számokat **0..100** között, és ezeket írja bele egy **közös stack** (típusa `Stack<int>`) gyűjteménybe a `Push` művelet felhasználásával. A munkaszál a ciklusból akkor lépjen ki és fejezze be a futását, amikor a generált véletlen szám értéke **99**.

A **fő szál** a munkaszálak indítása után egy ciklusban folyamatosan olvassa ki a munkaszálak által a közös stackbe írt véletlen számokat (`Pop` művelet), és ezek értékét írja ki a konzolra. Tartsa szem előtt, hogy ekkor a munkaszálak még futhatnak! A fő szál a munkaszálak által stackbe írt számokat a **lehető leghamarabb** írja ki (hatékony megoldásra törekedjen, kerülje az aktív, valamint a felesleges várakozásokat)! Azt, hogy a stack üres-e, a `Count` property/tulajdonság segítségével tudja ellenőrizni.

A fő szál akkor fejezze be az adatok feldolgozását, amikor a munkaszálak **kiléptek** és a stack **már kiürült**. Némiképpen kevesebb pontszámért olyan megoldást is választhat, mely nem gondoskodik arról, hogy a munkaszálak kilépését követően a stackben levő adatok feldolgozásra kerüljenek.

A feladatban szereplő intervallumok esetében szabadon választhat inkluzív és exkluzív megközelítést.

**Segítség:** Véletlen egész számok generálásához egy `Random` osztálybeli objektumot kell létrehozni (ezt csak egyszer), és a `Next(maximum_érték)` művelettel az egyes számokat generálni.

```csharp
using System; // Az alapvető rendszerkönyvtárak importálása (pl. a Console osztályhoz)
using System.Collections.Generic; // A generikus gyűjtemények (mint a Stack<T>) használatához szükséges
using System.Threading; // A szálkezelő osztályok (Thread, Monitor) elérését biztosítja

namespace SzalkezelesFeladat // A program névtere, ami összefogja az osztályokat
{
    class Program // A fő programosztály
    {
        // Közös erőforrások (Shared resources)
        static Stack<int> stack = new Stack<int>(); // A közös LIFO (utoljára be, először ki) adatszerkezet a számok tárolására
        static object lockObj = new object(); // Szinkronizációs objektum, ez szolgál "zárként" (lock) a szálak között
        static Random rnd = new Random(); // Véletlenszám-generátor, a feladat szerint csak egyszer hozzuk létre
        static int activeWorkers = 2; // Nyilvántartja az aktív (még futó) munkaszálak számát

        static void Main(string[] args) // A program belépési pontja, ez fut a főszálon
        {
            Thread t1 = new Thread(Worker); // Létrehozzuk az első munkaszálat, ami a Worker metódust fogja futtatni
            Thread t2 = new Thread(Worker); // Létrehozzuk a második munkaszálat, ami szintén a Worker metódust futtatja

            t1.Start(); // Elindítjuk az első munkaszálat
            t2.Start(); // Elindítjuk a második munkaszálat

            while (true) // A főszál végtelenített ciklusa az adatok folyamatos olvasásához
            {
                lock (lockObj) // Lefoglaljuk a zárat, hogy csak a főszál férjen hozzá a közös változókhoz (stack, activeWorkers)
                {
                    // Amíg a stack üres ÉS még vannak futó munkaszálak, a főszál várakozik
                    while (stack.Count == 0 && activeWorkers > 0) 
                    {
                        Monitor.Wait(lockObj); // A főszál elengedi a zárat és várakozó (alvó) állapotba kerül (megelőzi az aktív várakozást)
                    }

                    // Ha a felébredés után (vagy eleve) van adat a stack-ben...
                    if (stack.Count > 0) 
                    {
                        int number = stack.Pop(); // ...kivesszük a legfelső elemet (és egyben töröljük is onnan a Pop művelettel)
                        Console.WriteLine($"Kiolvasva a stackből: {number}"); // Kiírjuk a kivett számot a konzolra
                    }
                    // Ha a stack már teljesen üres ÉS az összes munkaszál végleg kilépett (activeWorkers == 0)...
                    else if (activeWorkers == 0) 
                    {
                        break; // ...kilépünk a főszál végtelen ciklusából, befejezve az adatfeldolgozást
                    }
                } // Itt engedi el a lefoglalt zárat a főszál, hogy a munkaszálak ismét hozzáférjenek a stackhez
            }

            // Ez a sor csak akkor fut le, ha a főszál kilépett a ciklusból (munkaszálak leálltak, stack kiürült)
            Console.WriteLine("A program sikeresen befejeződött, minden adat feldolgozásra került."); 
        }

        static void Worker() // A munkaszálak által végrehajtott metódus
        {
            while (true) // Végtelen ciklus a véletlen számok folyamatos generálásához
            {
                int number; // Változó a generált szám ideiglenes tárolására
                
                lock (lockObj) // Lefoglaljuk a zárat a véletlenszám-generáláshoz és a stack-be íráshoz, hogy ne legyen ütközés
                {
                    number = rnd.Next(0, 101); // Generálunk egy véletlen számot 0 és 100 között (inkluzív, ezért 101 a felső exkluzív határ)
                    stack.Push(number); // Belerakjuk a generált számot a közös stack-be
                    
                    Monitor.Pulse(lockObj); // Felébresztjük a várakozó főszálat, jelezve, hogy új adat érhető el a stack-ben
                } // Itt a munkaszál elengedi a zárat

                if (number == 99) // Ellenőrizzük a kilépési feltételt: ha a generált szám 99...
                {
                    lock (lockObj) // ...akkor újra lefoglaljuk a zárat az adminisztrációhoz (a számláló módosításához)
                    {
                        activeWorkers--; // Csökkentjük az aktív munkaszálak számát, jelezve a leállást
                        Monitor.Pulse(lockObj); // Felébresztjük a főszálat, hogy észlelje a munkaszál leállását (fontos, ha a stack épp üres lenne)
                    } // Zár elengedése
                    break; // Kilépünk a munkaszál ciklusából, így a munkaszál futása végleg befejeződik
                }
            }
        }
    }
}
```
