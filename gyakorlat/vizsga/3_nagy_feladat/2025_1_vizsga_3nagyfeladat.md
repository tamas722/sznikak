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
using System.Threading.Tasks; // Szükséges a Task és az await használatához

namespace SzalkezelesFeladat
{
    class Program
    {
        // Közös erőforrások
        static Stack<int> stack = new Stack<int>();
        static object lockObj = new object();
        static Random rnd = new Random();
        static int activeWorkers = 2;

        // Az AutoResetEvent váltja le a Monitor.Wait / Monitor.Pulse használatát.
        // false (nem jelzett) állapottal indul, tehát az első WaitOne várakozni fog.
        static AutoResetEvent autoEvent = new AutoResetEvent(false);

        // A Main metódus aszinkron lett (async Task), hogy lehessen használni az await kulcsszót.
        static async Task Main(string[] args)
        {
            Thread t1 = new Thread(Worker);
            Thread t2 = new Thread(Worker);

            t1.Start();
            t2.Start();

            // A főszál ciklusa folyamatosan olvassa az adatokat
            while (true)
            {
                // AWAIT HASZNÁLATA: Mivel a WaitOne() szinkron blokkolna, 
                // háttérszálra küldjük a várakozást, és aszinkron várjuk be az eredményt.
                // Ez megelőzi a felesleges aktív várakozást (polling).
                await Task.Run(() => autoEvent.WaitOne());

                lock (lockObj)
                {
                    // Az AutoResetEvent jelzései "összecsúszhatnak", ha a munkaszálak nagyon gyorsak.
                    // Ezért ha felébredtünk, az összes éppen elérhető elemet ki kell szedni a stackből.
                    while (stack.Count > 0)
                    {
                        int number = stack.Pop();
                        Console.WriteLine($"Kiolvasva a stackből: {number}");
                    }

                    // Ha a stack már üres ÉS az összes munkaszál kilépett
                    if (activeWorkers == 0 && stack.Count == 0)
                    {
                        break; // Kilépés a végtelen ciklusból
                    }
                }
            }

            // JOIN HASZNÁLATA: Ahogy a feladat kérte, a feldolgozás befejeztével 
            // a főszál szinkron módon be is várja a munkaszálak tényleges (fizikai) leállását.
            t1.Join();
            t2.Join();

            Console.WriteLine("A program sikeresen befejeződött, minden adat feldolgozásra került.");
        }

        static void Worker()
        {
            while (true)
            {
                int number;
                
                lock (lockObj)
                {
                    number = rnd.Next(0, 101); // Véletlen szám 0..100 (inkluzív)
                    stack.Push(number);
                }
                
                // Jelzés (Pulse helyett): Az AutoResetEvent nyit egy pillanatra, 
                // felébresztve a feladatban várakozó főszálat.
                autoEvent.Set();

                if (number == 99)
                {
                    lock (lockObj)
                    {
                        activeWorkers--; // Jelezzük a leállást
                    }
                    
                    // Extra jelzés a főszálnak, nehogy alvó állapotban ragadjon, 
                    // miután az utolsó szál is végzett.
                    autoEvent.Set(); 
                    break;
                }
            }
        }
    }
}
```
