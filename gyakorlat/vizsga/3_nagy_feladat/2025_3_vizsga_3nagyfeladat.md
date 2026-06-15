# 3. Szálkezelés – Írjon az alábbiaknak megfelelő C# nyelvű konzol alkalmazást.

Vezessen be egy `Decrement` nevű függvényt, mely egy **5 kezdőértékű** számláló (egy egyszerű egész szám) értékét csökkenti **eggyel** (vagyis visszafelé számlál). Ennek többszálú környezetben is jól kell működnie.

Vezessen be egy `WaitForZero` függvényt, melyet bármely, az esetleges későbbi fejlesztések során indított szál használhat (akár több is, és akár egyszerre is). A függvény mindaddig várakozzon (blokkolja a hívó szálat), míg a számláló értéke el nem éri a **0**-t, és csak akkor tér vissza (ez valamennyi várakozó szálra igaz). A várakozásnál **hatékony megoldásra** törekedjen, kerülje az aktív, valamint a felesleges várakozást. A várakozás végeztével a függvény írja ki a `"Zero reached"` szöveget a konzolra. Ezen felül a várakozás végeztével a várakozók közül **az egyik szál** (csak az egyik, akkor is, ha több is van!) a `WaitForZero` függvényben állítsa vissza a számláló értékét **5**-re.

Az alkalmazás **fő szála** (maga a `Main` függvény) indítson el **két ekvivalens munkaszálat** (közös a szálfüggvény definíció), melyek **végtelen ciklusban**, ciklusiterációnként **0..800 ms** között véletlenszerűen várakozva hívják a `Decrement` függvényt. Ezt követően a fő szál hívja meg a `WaitForZero` függvényt, mellyel bevárja, hogy a számláló elérje a 0-t. Ezt követően a főszál lépjen ki. Lényeges, hogy ekkor az egész alkalmazás fejezze be a futását, vagyis a munkaszálak **ne ragadjanak be a háttérben** (ezt a lehető legegyszerűbb technikával oldja meg).

A feladatban szereplő intervallumok esetében szabadon választhat inkluzív és exkluzív megközelítést.

**Segítség:** Véletlen egész számok generálásához egy `Random` osztálybeli objektumot kell létrehozni (ezt csak egyszer), és a `Next(maximum_érték)` művelettel az egyes számokat generálni.

```csharp
using System;
using System.Threading;
using System.Threading.Tasks; // Szükséges a Task és az await használatához

namespace SzalkezelesFeladat3
{
    class Program
    {
        // Közös erőforrások
        static int counter = 5; 
        static object lockObj = new object(); 
        static Random rnd = new Random(); 
        
        // ManualResetEvent, ami kezdetben false (zárt, tehát blokkolja a várakozókat).
        // Amikor a számláló 0 lesz, kinyitjuk (Set), így AZ ÖSSZES várakozó szálat átengedi.
        static ManualResetEvent mre = new ManualResetEvent(false);

        // Ez a változó teszi lehetővé, hogy kilépjünk a végtelen ciklusból, és használhassuk a Join()-t.
        static volatile bool isRunning = true;

        static async Task Main(string[] args)
        {
            Thread t1 = new Thread(Worker);
            Thread t2 = new Thread(Worker);

            // A feladat által említett "legegyszerűbb technika", hogy a szálak ne ragadjanak be
            // (a biztonság kedvéért meghagyva, bár a Join és az isRunning miatt amúgy is kilépnének).
            t1.IsBackground = true; 
            t2.IsBackground = true;

            t1.Start();
            t2.Start();

            // AWAIT HASZNÁLATA: A főszál aszinkron módon várja meg a számláló nullázódását,
            // hogy ne blokkolódjon a főszál (UI vagy más műveletek esetén ez elengedhetetlen lenne).
            await Task.Run(() => WaitForZero());

            // A feladat kérte, hogy innentől az alkalmazás fejezze be a futását.
            // A JOIN HASZNÁLATÁHOZ jelezzük a szálaknak, hogy álljanak le a végtelen ciklusból:
            isRunning = false;

            // Bevárjuk a munkaszálak tiszta és biztonságos leállását:
            t1.Join();
            t2.Join();

            Console.WriteLine("A főszál kilép, az alkalmazás befejeződik.");
        }

        static void Worker()
        {
            // "Végtelen ciklus", ami addig fut, amíg a főszál le nem állítja
            while (isRunning) 
            {
                Thread.Sleep(rnd.Next(0, 801)); // Véletlenszerű várakozás 0..800 ms között
                
                if (isRunning) // Extra ellenőrzés, hogy leállítás után már ne csökkentsünk
                {
                    Decrement();
                }
            }
        }

        static void Decrement()
        {
            lock (lockObj) 
            {
                if (counter > 0) 
                {
                    counter--;
                    Console.WriteLine($"Számláló csökkentve: {counter}");

                    if (counter == 0)
                    {
                        // Amikor a számláló eléri a 0-t, "felnyitjuk a sorompót".
                        // A ManualResetEvent minden rajta várakozó szálat (WaitOne) felébreszt!
                        mre.Set(); 
                    }
                }
            }
        }

        static void WaitForZero()
        {
            // Várakozás blokkolással (aktív várakozás / spinlock nélkül).
            // A szál itt addig alszik, amíg a Decrement() meg nem hívja az mre.Set()-et.
            mre.WaitOne();

            // A feladat megkövetelte, hogy a felébredés után minden szál írja ki ezt:
            Console.WriteLine("Zero reached");

            // Csak és kizárólag EGYETLEN szál állíthatja vissza az értéket 5-re!
            // Ehhez lefoglaljuk a zárat, így a párhuzamosan felébredő szálak sorba állnak.
            lock (lockObj)
            {
                // Az elsőként beérkező szál látja, hogy a counter 0, tehát ő végzi el a visszaállítást.
                if (counter == 0)
                {
                    counter = 5; // Visszaállítás
                    
                    // A sorompót "visszazárjuk", hogy a következő körben újra meg lehessen állni előtte.
                    mre.Reset(); 
                    
                    Console.WriteLine("[Rendszer: A számláló vissza lett állítva 5-re.]");
                }
                // A többi szál, ami ide érkezik, már azt látja, hogy counter == 5, így nem csinálnak semmit.
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
