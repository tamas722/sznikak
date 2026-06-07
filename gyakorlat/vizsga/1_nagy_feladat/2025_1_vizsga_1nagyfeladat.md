# 1. Nyelvi eszközök

A Kitten nevű osztály kiscicát reprezentál. A kiscicát meg lehet simogatni (Stroke művelet) és meg lehet etetni (Feed művelet). A Stroke művelet a kiscica boldogságához 3, a Feed 4 egységgel járul hozzá. Amennyiben egy simogatás során a cica boldogsága meghaladja a 12-t, a kiscica dorombolni kezd, amit a Purr nevű eseménnyel jelez, és paraméterben átadja a cica aktuális boldogságszámát. A cica aktuális boldogságszáma a Count tulajdonsággal (property) legyen lekérdezhető (beállítani az osztályon kívülről nem lehet).

Adja meg a Kitten osztály teljes C# nyelvű forráskódját és egy pár soros példaprogramot, mely létrehoz egy példányt belőle, feliratkozik az eseményére, és annak minden kiváltódásakor kiírja a cica boldogságszámát a konzolra. A megvalósítás során nem használhatja a .NET keretrendszer beépített eseménytípusait. (EventHandler, Func, Action és hasonlókat).
(20p)

```csharp
using System; // Beimportáljuk a System névteret, ez kell például a Console osztály használatához.

public delegate void PurrDelegate(int happinessCount); // Saját delegált típus létrehozása az eseményhez, amely egy int paramétert vár.

public class Kitten // A kiscicát reprezentáló osztály deklarálása.
{ 
    public int Count { get; private set; } // A boldogságszám: kívülről lekérdezhető (get), de csak ezen az osztályon belül módosítható (private set).

    public event PurrDelegate Purr; // A 'Purr' (dorombolás) esemény deklarálása a fent létrehozott saját delegált típusunkkal.

    public void Stroke() // A simogatás (Stroke) művelet metódusa.
    { 
        Count += 3; // Simogatáskor a boldogságszámot megnöveljük 3-mal.
        
        if (Count > 12) // Ellenőrizzük a feltételt: a boldogság meghaladta-e a 12-t.
        { 
            if (Purr != null) // Ellenőrizzük, hogy van-e feliratkozó az eseményre (nehogy hibára fusson a program, ha üres).
            { 
                Purr(Count); // Kiváltjuk (elsütjük) a Purr eseményt, és paraméterként átadjuk az aktuális boldogságszámot.
            } 
        } 
    } 

    public void Feed() // Az etetés (Feed) művelet metódusa.
    { 
        Count += 4; // Etetéskor a boldogságszámot megnöveljük 4-gyel.
    } 
} 

class Program // A főprogram osztálya, tartalmazza a belépési pontot.
{ 
    static void Main(string[] args) // A program fő belépési pontja (innen indul a futás).
    { 
        Kitten myKitten = new Kitten(); // Példányosítjuk a Kitten osztályt (létrehozunk egy kiscica objektumot a memóriában).
        
        myKitten.Purr += OnKittenPurr; // Feliratkozunk a kiscica Purr eseményére az általunk írt OnKittenPurr metódussal.
        
        myKitten.Feed(); // Megetetjük a cicát (boldogsága most 0 + 4 = 4).
        myKitten.Stroke(); // Megsimogatjuk a cicát (boldogsága most 4 + 3 = 7). Még nem dorombol.
        myKitten.Feed(); // Újra megetetjük (boldogsága most 7 + 4 = 11).
        myKitten.Stroke(); // Megsimogatjuk (boldogsága most 11 + 3 = 14). Mivel 14 > 12, kiváltódik az esemény!
    } 

    static void OnKittenPurr(int happinessCount) // A saját eseménykezelő metódusunk, ami akkor fut le, ha a cica dorombol.
    { 
        Console.WriteLine("A kiscica dorombol! Aktuális boldogságszáma: " + happinessCount); // Kiírja a konzolra az üzenetet és a megkapott számot.
    } 
}
´´´