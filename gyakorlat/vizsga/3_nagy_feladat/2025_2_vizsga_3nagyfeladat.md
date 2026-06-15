# 3. Szálkezelés – Írjon az alábbiaknak megfelelő C# nyelvű konzol alkalmazást.

Készítsen az alábbiaknak megfelelő C# nyelvű konzol alkalmazást.

Az alkalmazás **fő szála** (ez maga a `Main` függvényt hívó szál) először egy listába rakjon be **1000 darab 0..10 közötti** véletlen számot, majd indítson el **két munkaszálat**. A munkaszálak indítást követően írják ki a `"Munkára kész"` szöveget. Miután **mindkét** munkaszál kiírta ezt a szöveget (de csak ezután!) a fő szál írja ki a `"Mindenki munkára kész"` szöveget.

A két munkaszál feladata, hogy a fő szál által előállított listában levő **páratlan elemek összegét** "közös erőfeszítéssel", párhuzamosan futva kiszámolják. Ezt úgy oldják meg, hogy mindaddig, amíg a lista ki nem ürül, folyamatosan kiolvassák és eltávolítják a lista **utolsó elemét**, és ezt hozzáadják egy közös összeghez (ha a szám páratlan). Az alábbi kódrészlet (feltéve, hogy a `numbers` egy `List<int>` objektum) kiolvassa és eltávolítja a lista utolsó elemét, ezt a kódrészletet felhasználhatja a megoldás során:

```csharp
int last = numbers[numbers.Count - 1];
numbers.RemoveAt(numbers.Count - 1);
```

A munkaszálak, miután végeztek, fejezzék be a futásukat. Miután a munkaszálak befejezték a futásukat (miután kiléptek!), a fő szál írja ki az eredményt.

A megoldásában a várakozások esetén hatékony megoldásra törekedjen, kerülje az aktív, valamint a felesleges várakozásokat. Amennyiben több szálfüggvényt is írna, és a második az elsőtől csak kismértékben különbözik, elég az egyiket megírnia, a másodikra pedig azt adja meg, miben különbözik az elsőtől.

A feladatban szereplő esetleges intervallumok esetében szabadon választhat inkluzív és exkluzív megközelítést.

Segítség: Véletlen egész számok generálásához egy Random osztálybeli objektumot kell létrehozni (ezt csak egyszer), és a Next(maximum_érték) művelettel az egyes számokat generálni.

```csharp
using System;
using System.Collections.Generic;
using System.Threading;

namespace SzalkezelesFeladat2
{
    class Program
    {
        // Közös erőforrások
        static List<int> numbers = new List<int>();
        static int sum = 0;
        
        // Külön zároló objektumok a hatékonyság érdekében
        static object listLock = new object();
        static object sumLock = new object(); 
        
        static Random rnd = new Random();

        // A TE JAVASLATOD: Statikus ManualResetEvent tömb a két szálnak
        static ManualResetEvent[] readyEvents = new ManualResetEvent[]
        {
            new ManualResetEvent(false),
            new ManualResetEvent(false)
        };

        static void Main(string[] args)
        {
            // 1. lépés: Lista feltöltése
            for (int i = 0; i < 1000; i++)
            {
                numbers.Add(rnd.Next(0, 11)); // 0..10 közötti véletlen számok
            }

            // 2. lépés: Munkaszálak létrehozása és indítása
            Thread t1 = new Thread(Worker);
            Thread t2 = new Thread(Worker);

            // Csak egy sorszámot (indexet) adunk át paraméterként
            t1.Start(0);
            t2.Start(1);

            // 3. lépés: VÁRAKOZÁS
            // Elegáns várakozás a tömb összes elemére (WaitAll)
            WaitHandle.WaitAll(readyEvents);
            
            // Ez csak a két szál jelzése után fut le
            Console.WriteLine("Mindenki munkára kész");

            // 4. lépés: Szálak tényleges befejezésének megvárása
            t1.Join();
            t2.Join();

            // 5. lépés: Eredmény kiírása
            Console.WriteLine($"A páratlan számok összege: {sum}");
        }

        static void Worker(object idParam)
        {
            // Lekérjük a szál saját azonosítóját (0 vagy 1)
            int threadId = (int)idParam;

            // 1. Feladat: Készenlét jelzése
            Console.WriteLine("Munkára kész");
            
            // Értesítjük a főszálat a SAJÁT eseményünkön keresztül
            readyEvents[threadId].Set();

            // 2. Feladat: Számok feldolgozása
            while (true)
            {
                int last;

                // 1. Kritikus szakasz: csak a lista kezelése
                lock (listLock) 
                {
                    if (numbers.Count == 0)
                    {
                        break; // Ha üres a lista, kilépünk
                    }

                    // A feladatban megadott kódrészlet
                    last = numbers[numbers.Count - 1];
                    numbers.RemoveAt(numbers.Count - 1);
                } 

                // A vizsgálat a lock-on kívül történik (párhuzamosítás)
                if (last % 2 != 0)
                {
                    // 2. Kritikus szakasz: csak az összegzés
                    lock (sumLock)
                    {
                        sum += last;
                    }
                }
            }
        }
    }
}
```
