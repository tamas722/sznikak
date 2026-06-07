# 3. Szálkezelés – Írjon az alábbiaknak megfelelő C# nyelvű konzol alkalmazást.

Vezessen be egy `Decrement` nevű függvényt, mely egy **5 kezdőértékű** számláló (egy egyszerű egész szám) értékét csökkenti **eggyel** (vagyis visszafelé számlál). Ennek többszálú környezetben is jól kell működnie.

Vezessen be egy `WaitForZero` függvényt, melyet bármely, az esetleges későbbi fejlesztések során indított szál használhat (akár több is, és akár egyszerre is). A függvény mindaddig várakozzon (blokkolja a hívó szálat), míg a számláló értéke el nem éri a **0**-t, és csak akkor tér vissza (ez valamennyi várakozó szálra igaz). A várakozásnál **hatékony megoldásra** törekedjen, kerülje az aktív, valamint a felesleges várakozást. A várakozás végeztével a függvény írja ki a `"Zero reached"` szöveget a konzolra. Ezen felül a várakozás végeztével a várakozók közül **az egyik szál** (csak az egyik, akkor is, ha több is van!) a `WaitForZero` függvényben állítsa vissza a számláló értékét **5**-re.

Az alkalmazás **fő szála** (maga a `Main` függvény) indítson el **két ekvivalens munkaszálat** (közös a szálfüggvény definíció), melyek **végtelen ciklusban**, ciklusiterációnként **0..800 ms** között véletlenszerűen várakozva hívják a `Decrement` függvényt. Ezt követően a fő szál hívja meg a `WaitForZero` függvényt, mellyel bevárja, hogy a számláló elérje a 0-t. Ezt követően a főszál lépjen ki. Lényeges, hogy ekkor az egész alkalmazás fejezze be a futását, vagyis a munkaszálak **ne ragadjanak be a háttérben** (ezt a lehető legegyszerűbb technikával oldja meg).

A feladatban szereplő intervallumok esetében szabadon választhat inkluzív és exkluzív megközelítést.

**Segítség:** Véletlen egész számok generálásához egy `Random` osztálybeli objektumot kell létrehozni (ezt csak egyszer), és a `Next(maximum_érték)` művelettel az egyes számokat generálni.

```csharp
using System; // Alapvető IO műveletekhez (Console)
using System.Threading; // Szálkezelés eszközeihez (Thread, Monitor)

namespace SzalkezelesFeladat3 // Névtér a programnak
{
    class Program
    {
        // Közös erőforrások (Shared resources)
        static int counter = 5; // A számláló, 5-ös kezdőértékkel
        static object lockObj = new object(); // Szinkronizációs objektum (zár) a szálakhoz
        
        // Egy "fázis" változó, ami segít a WaitForZero-ban, hogy egy szál se aludjon vissza 
        // miután a számláló 5-re lett állítva, hanem mindenki sikeresen visszatérjen.
        static int phase = 0; 
        
        static Random rnd = new Random(); // Véletlenszám-generátor, csak egyszer létrehozva

        static void Main(string[] args) // A főszál belépési pontja
        {
            Thread t1 = new Thread(Worker); // Első munkaszál példányosítása
            Thread t2 = new Thread(Worker); // Második munkaszál példányosítása

            // A legegyszerűbb technika a "beragadó" szálak elkerülésére:
            // Háttérszálként (Background) indítjuk őket, így a főszál kilépésekor azonnal,
            // automatikusan megszakad a futásuk, az egész program leáll.
            t1.IsBackground = true; 
            t2.IsBackground = true;

            t1.Start(); // Első munkaszál indítása
            t2.Start(); // Második munkaszál indítása

            // A főszál bevárja, amíg a számláló eléri a 0-t
            WaitForZero();

            // Amikor a WaitForZero visszatér, a főszál befejezi futását.
            Console.WriteLine("A főszál kilép, az alkalmazás befejeződik.");
        }

        // A szálak által futtatott végtelen ciklus
        static void Worker()
        {
            while (true) // Végtelen ciklus, ahogy a feladat kérte
            {
                // Véletlenszerű várakozás 0 és 800 ms között. 
                // A Next 2. paramétere exkluzív, ezért 801-et adunk meg, hogy a 800 is benne legyen (inkluzív megközelítés).
                Thread.Sleep(rnd.Next(0, 801)); 
                
                Decrement(); // Számláló csökkentése
            }
        }

        // A számláló csökkentését végző szálbiztos függvény
        static void Decrement()
        {
            lock (lockObj) // Lefoglaljuk a zárat, hogy egyszerre csak egy szál módosítsa a számlálót
            {
                if (counter > 0) // Biztosítjuk, hogy ne menjen 0 alá a számláló, amíg a másik szál újra nem indítja
                {
                    counter--; // Érték csökkentése eggyel
                    Console.WriteLine($"Számláló csökkentve: {counter}"); // Segédkiírás a működés követéséhez

                    if (counter == 0) // Ha elértük a nullát...
                    {
                        phase++; // ...megnöveljük a fázist, jelezve a várakozóknak a ciklus végét...
                        Monitor.PulseAll(lockObj); // ...és felébresztjük AZ ÖSSZES (esetlegesen) várakozó szálat.
                    }
                }
            } // Zár elengedése
        }

        // A várakozást végző szálbiztos függvény (akár több szál is hívhatja)
        static void WaitForZero()
        {
            lock (lockObj) // Lefoglaljuk a zárat az ellenőrzéshez és várakozáshoz
            {
                int myPhase = phase; // Eltároljuk, hogy melyik fázisban kezdtünk el várakozni

                // Amíg a számláló > 0 ÉS a fázis nem változott meg (spurious wakeup ellen)
                while (counter > 0 && phase == myPhase) 
                {
                    Monitor.Wait(lockObj); // Várakozó állapotba lépünk (hatékony, nincs aktív várakozás)
                }

                // Ezt a pontot csak akkor érjük el, ha a számláló 0 lett (és a fázis megnőtt).
                // Mivel a feladat kérte, hogy minden várakozó szál írja ki a szöveget:
                Console.WriteLine("Zero reached"); 

                // Mivel csak EGYETLEN szál állíthatja vissza 5-re, ez a feltétel gondoskodik róla.
                // A Monitor.Wait-ből elsőként felébredő szál látni fogja, hogy counter == 0.
                if (counter == 0) 
                {
                    counter = 5; // Visszaállítja a számlálót 5-re.
                    Console.WriteLine("[Rendszer: A számláló vissza lett állítva 5-re.]");
                    // A többi szál, ami ezután ébred fel a Wait-ből, már azt látja, hogy a counter 5,
                    // de a "phase == myPhase" feltétel miatt nem fognak visszaaludni a while ciklusban!
                }
            } // Zár elengedése
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