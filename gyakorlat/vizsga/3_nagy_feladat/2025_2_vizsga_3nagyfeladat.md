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
using System; // Az alapvető rendszerkönyvtárak importálása (pl. Console használatához)
using System.Collections.Generic; // A generikus gyűjtemények (mint a List<T>) használatához szükséges
using System.Threading; // A szálkezelő osztályok (Thread, Monitor) elérését biztosítja

namespace SzalkezelesFeladat2 // A program névtere
{
    class Program // A fő programosztály
    {
        // Közös erőforrások (Shared resources)
        static List<int> numbers = new List<int>(); // A közös lista, amelyből a munkaszálak olvasnak
        static int sum = 0; // A közös változó, amelyben a páratlan számok összegét tároljuk
        
        static object listLock = new object(); // Szinkronizációs objektum (zár) a listához és az összeghez
        
        static int readyCount = 0; // Számláló, ami jelzi, hány munkaszál írta ki, hogy "Munkára kész"
        static object readyLock = new object(); // Külön szinkronizációs objektum a "készenléti" állapothoz
        
        static Random rnd = new Random(); // Véletlenszám-generátor, a feladat kérésének megfelelően csak egyszer létrehozva

        static void Main(string[] args) // A program belépési pontja, a főszál futtatja
        {
            // 1. lépés: Lista feltöltése
            for (int i = 0; i < 1000; i++) // 1000 darab számot generálunk
            {
                numbers.Add(rnd.Next(0, 11)); // 0..10 közötti (inkluzív) véletlen szám hozzáadása a listához
            }

            // 2. lépés: Munkaszálak létrehozása és indítása
            Thread t1 = new Thread(Worker); // Első munkaszál példányosítása a Worker metódussal
            Thread t2 = new Thread(Worker); // Második munkaszál példányosítása ugyanazzal a metódussal

            t1.Start(); // Első munkaszál elindítása
            t2.Start(); // Második munkaszál elindítása

            // 3. lépés: Várakozás, amíg mindkét szál munkára kész (aktív várakozás nélkül!)
            lock (readyLock) // Lefoglaljuk a "készenléti" zárat, hogy biztonságosan olvassuk a readyCount változót
            {
                while (readyCount < 2) // Amíg nem áll készen mind a két szál...
                {
                    Monitor.Wait(readyLock); // ...a főszál elengedi a zárat és bealszik (így nincs felesleges processzorhasználat)
                }
            } // A zár elengedése
            
            // Ez a sor csak azután fut le, hogy mindkét munkaszál jelezte a készenlétet
            Console.WriteLine("Mindenki munkára kész"); 

            // 4. lépés: Várakozás a munkaszálak befejeződésére
            t1.Join(); // A főszál megvárja (blokkolódik), amíg a t1 szál befejezi a futását és kilép
            t2.Join(); // A főszál megvárja, amíg a t2 szál is kilép

            // 5. lépés: Eredmény kiírása (miután a szálak már végeztek)
            Console.WriteLine($"A páratlan számok összege: {sum}"); // Kiírjuk a végső, közösen kiszámolt összeget
        }

        static void Worker() // A munkaszálak által végrehajtott közös metódus
        {
            // 1. Feladat: Jelezni a főszálnak, hogy a szál munkára kész
            Console.WriteLine("Munkára kész"); // Kiírjuk a konzolra a kért szöveget
            
            lock (readyLock) // Lefoglaljuk a "készenléti" zárat, hogy biztonságosan módosítsuk a számlálót
            {
                readyCount++; // Megnöveljük a készenlétben lévő szálak számát
                Monitor.Pulse(readyLock); // Szólunk (felébresztjük) a várakozó főszálnak, hogy változott a számláló
            } // Zár elengedése

            // 2. Feladat: Számok feldolgozása a listából amíg ki nem ürül
            while (true) // Végtelen ciklus, amiből majd kilépünk, ha üres a lista
            {
                lock (listLock) // Lefoglaljuk a listát és az összeget védő zárat (hogy a két szál ne zavarja egymást)
                {
                    if (numbers.Count == 0) // Ellenőrizzük, hogy üres-e a lista
                    {
                        break; // Ha üres, kilépünk a while ciklusból (és ezáltal a Worker metódus is befejeződik)
                    }

                    // A feladat által megadott kódrészlet a lista utolsó elemének kiolvasására és eltávolítására:
                    int last = numbers[numbers.Count - 1]; // Kiolvassuk az utolsó elemet
                    numbers.RemoveAt(numbers.Count - 1);   // Eltávolítjuk az utolsó elemet a listából

                    // Ellenőrizzük, hogy a szám páratlan-e
                    if (last % 2 != 0) // Ha a maradék 2-vel osztva nem nulla, akkor páratlan
                    {
                        sum += last; // Hozzáadjuk a kivett számot a közös összeghez
                    }
                } // Itt engedjük el a zárat, hogy a másik szál is tudjon elemet kivenni
            }
            // Miután a break miatt kilép a ciklusból, a metódus lefutott, a munkaszál természetes úton "meghal" (kilép).
        }
    }
}
```