# 3. Szálkezelés – Írjon az alábbiaknak megfelelő C# nyelvű konzol alkalmazást.

Vezessen be egy `Decrement` nevű függvényt, mely egy **5 kezdőértékű** számláló (egy egyszerű egész szám) értékét csökkenti **eggyel** (vagyis visszafelé számlál). Ennek többszálú környezetben is jól kell működnie.

Vezessen be egy `WaitForZero` függvényt, melyet bármely, az esetleges későbbi fejlesztések során indított szál használhat (akár több is, és akár egyszerre is). A függvény mindaddig várakozzon (blokkolja a hívó szálat), míg a számláló értéke el nem éri a **0**-t, és csak akkor tér vissza (ez valamennyi várakozó szálra igaz). A várakozásnál **hatékony megoldásra** törekedjen, kerülje az aktív, valamint a felesleges várakozást. A várakozás végeztével a függvény írja ki a `"Zero reached"` szöveget a konzolra. Ezen felül a várakozás végeztével a várakozók közül **az egyik szál** (csak az egyik, akkor is, ha több is van!) a `WaitForZero` függvényben állítsa vissza a számláló értékét **5**-re.

Az alkalmazás **fő szála** (maga a `Main` függvény) indítson el **két ekvivalens munkaszálat** (közös a szálfüggvény definíció), melyek **végtelen ciklusban**, ciklusiterációnként **0..800 ms** között véletlenszerűen várakozva hívják a `Decrement` függvényt. Ezt követően a fő szál hívja meg a `WaitForZero` függvényt, mellyel bevárja, hogy a számláló elérje a 0-t. Ezt követően a főszál lépjen ki. Lényeges, hogy ekkor az egész alkalmazás fejezze be a futását, vagyis a munkaszálak **ne ragadjanak be a háttérben** (ezt a lehető legegyszerűbb technikával oldja meg).

A feladatban szereplő intervallumok esetében szabadon választhat inkluzív és exkluzív megközelítést.

**Segítség:** Véletlen egész számok generálásához egy `Random` osztálybeli objektumot kell létrehozni (ezt csak egyszer), és a `Next(maximum_érték)` művelettel az egyes számokat generálni.

```csharp
using System;
using System.Threading;

namespace SzalkezelesFeladat3
{
    class Program
    {
        // Közös erőforrások
        static int counter = 5; 
        static object lockObj = new object(); 
        static Random rnd = new Random(); 
        
        // Hatékony eseményalapú várakozás: kezdetben zárt (false), így a WaitOne() blokkol.
        // Amikor a számláló eléri a 0-t, a Set() kinyitja az ÖSSZES rajta várakozó szál számára.
        static ManualResetEvent mre = new ManualResetEvent(false);

        static void Main(string[] args)
        {
            // Két ekvivalens munkaszál létrehozása közös szálfüggvénnyel
            Thread t1 = new Thread(Worker);
            Thread t2 = new Thread(Worker);

            // A LEGEGYSZERŰBB TECHNIKA a beragadás ellen: Háttérszállá (Background Thread) alakítjuk őket.
            // Így ha a főszál (Foreground Thread) befejezi a futását, az OS/CLR azonnal lelövi őket is.
            t1.IsBackground = true; 
            t2.IsBackground = true;

            t1.Start();
            t2.Start();

            // A főszál meghívja a függvényt, amivel hatékonyan megvárja a 0 elérését
            WaitForZero();

            Console.WriteLine("A főszál befejezte a munkáját, az alkalmazás kilép.");
        }

        static void Worker()
        {
            // Végtelen ciklusban futó munkaszál
            while (true) 
            {
                // Véletlenszerű várakozás 0..800 ms között (inkluzív megközelítéssel 801-ig)
                Thread.Sleep(rnd.Next(0, 801)); 
                
                Decrement();
            }
        }

        static void Decrement()
        {
            // Kritikus szakasz a számláló szálbiztos módosításához
            lock (lockObj) 
            {
                if (counter > 0) 
                {
                    counter--;
                    Console.WriteLine($"Számláló csökkentve: {counter}");

                    if (counter == 0)
                    {
                        // Ha elérte a nullát, kinyitjuk a "sorompót", minden várakozó felébred
                        mre.Set(); 
                    }
                }
            }
        }

        static void WaitForZero()
        {
            // Hatékony, nem aktív várakozás. A szál itt blokkolódik (alszik), amíg a jelet meg nem kapja.
            mre.WaitOne();

            // Amint felébred a szál (vagy szálak), kiírja a kötelező szöveget
            Console.WriteLine("Zero reached");

            // Biztosítani kell, hogy a felébredő szálak közül CSAK AZ EGYIK állítsa vissza az értéket.
            // Ehhez kritikus szakaszt nyitunk: az elsőként bejutó szál fogja elvégezni a resetet.
            lock (lockObj)
            {
                if (counter == 0)
                {
                    counter = 5; // Visszaállítás 5-re
                    
                    // Visszazárjuk a ManualResetEvent-et (false-ra), hogy a következő körben 
                    // a szálak újra blokkolódjanak előtte, amíg újra le nem csökken 0-ra.
                    mre.Reset(); 
                    
                    Console.WriteLine("[Rendszer: Egy szál visszaállította a számlálót 5-re.]");
                }
                // Azok a szálak, amelyek később jutnak be ide a lock miatt, már azt fogják látni,
                // hogy a counter értéke 5, így ők már nem futnak rá erre a blokkra.
            }
        }
    }
}
```

## A megoldás legfontosabb logikai elemei:

- **`IsBackground = true`**: Alapértelmezett esetben a C#-ban elindított szálak ún. *Foreground* (előtér) szálak, ami azt jelenti, hogy a program addig nem áll le, amíg ezek futnak (még ha a `Main` véget is ért). Ha a tulajdonságukat `Background`-ra állítjuk, akkor a `Main` szál kilépése azonnal, erőszakosan leállítja őket a háttérben. Ez a feladat által kért **"lehető legegyszerűbb technika"**.

- **A `phase` változó trükkje**: Ha 3 szál egyszerre hívná a `WaitForZero`-t, mindhárom elalszik a `Monitor.Wait`-en. Amikor a `Decrement` nulláz, mind a hármat felébreszti a `PulseAll`.  
  Az első felébredő kiírja, hogy `"Zero reached"`, majd átállítja a számlálót 5-re.  
  A második felébredő kiírja, hogy `"Zero reached"`, majd visszaérve a `while` ciklus ellenőrzéséhez azt látná, hogy a számláló már **5 > 0**. Ha nem lenne a `phase` változó, a második szál tévesen visszaaludna ahelyett, hogy visszatérne a függvényből. A `phase` megváltozása garantálja, hogy a `while` ciklus azonnal megszakad mindenki számára, aki abban az iterációban feküdt le aludni.

- **`Monitor.Wait` és `PulseAll`**: Ez biztosítja a teljesen passzív, processzort 0%-on tartó (**hatékony**) várakozást, tökéletesen eleget téve a kiírásnak.
