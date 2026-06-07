# 1. Nyelvi eszközök


Készítsen egy Craft nevű osztályt, mely egy önvezető vízi járművet reprezentál egy szimulátor alkalmazásban! Az osztály egy double típusú, velocity nevű tagváltozóban tárolja a jármű aktuális sebességét. A tagváltozó értéke egy Velocity nevű tulajdonságon (property) keresztül kérdezhető le és állítható be (0-nál kisebb érték esetén dobjon kivételt!).

Az osztály ezen felül tartalmaz egy ObjectDetected eseményt (C# event-et). Az esemény akkor váltódik ki, amikor a jármű a szimuláció léptetése során (Iterate művelet) egy akadályt detektál, és paraméterben átadja a detektált akadály méretét (int típusú). Az Iterate műveletben az akadály detektálása véletlenszerűen történik: az Iterate akkor tekinti úgy, hogy a jármű akadályt észlel, amikor a new Random().Next(100) utasítás 76-nál nagyobb értéket ad vissza, a detektált akadály mérete pedig maga a generált véletlenszám.

Adja meg a Craft osztály teljes C# nyelvű forráskódját és egy pár soros példaprogramot, mely létrehoz egy példányt belőle, feliratkozik az eseményére, és annak minden kiváltódásakor kiírja a konzolra az esemény paraméterét. A megvalósítás során nem használhatja a .NET keretrendszer beépített eseménytípusait. (15p)

```csharp
using System; // Beimportáljuk a System névteret, amely a Console és az Exception (kivétel) osztályokhoz szükséges.
using System.Threading; // Beimportáljuk a szálkezelés névterét, hogy a példaprogramban várakoztatni tudjuk a futást (a véletlenszám-generátor miatt).

public delegate void ObjectDetectedDelegate(int size); // Saját delegált típus létrehozása, amely egy int (méret) paramétert vár.

public class Craft // Az önvezető vízi járművet reprezentáló osztály deklarálása.
{
    private double velocity; // Privát tagváltozó a jármű aktuális sebességének tárolására.

    public double Velocity // A Velocity tulajdonság (property), amelyen keresztül a sebesség elérhető.
    {
        get // A get blokk fut le, amikor le akarjuk kérdezni a sebességet.
        {
            return velocity; // Visszaadja a privát velocity tagváltozó értékét.
        }
        set // A set blokk fut le, amikor új értéket akarunk adni a sebességnek.
        {
            if (value < 0) // Ellenőrizzük, hogy a beállítani kívánt érték (value) kisebb-e nullánál.
            {
                throw new ArgumentException("A sebesség nem lehet kisebb 0-nál!"); // Ha kisebb, kivételt (hibát) dobunk a megadott üzenettel.
            }
            velocity = value; // Ha az érték megfelelő (nem negatív), eltároljuk a privát tagváltozóban.
        }
    }

    public event ObjectDetectedDelegate ObjectDetected; // Az akadálydetektálást jelző esemény deklarálása a saját delegáltunkkal.

    public void Iterate() // A szimulációs lépést végrehajtó metódus.
    {
        int detectedSize = new Random().Next(100); // Legenerálunk egy véletlen számot 0 és 99 között, ahogy a feladat kérte.
        
        if (detectedSize > 76) // Megvizsgáljuk, hogy a generált szám nagyobb-e 76-nál (akadály észlelése).
        {
            if (ObjectDetected != null) // Ellenőrizzük, hogy feliratkozott-e valaki az eseményre (nehogy hibára fusson).
            {
                ObjectDetected(detectedSize); // Kiváltjuk az eseményt, és paraméterként átadjuk a generált véletlenszámot (az akadály méretét).
            }
        }
    }
}

class Program // A főprogram osztálya, tartalmazza a belépési pontot.
{
    static void Main(string[] args) // A program fő belépési pontja.
    {
        Craft myCraft = new Craft(); // Létrehozunk egy új Craft példányt (egy jármű objektumot) a memóriában.

        myCraft.ObjectDetected += OnObjectDetected; // Feliratkozunk a jármű ObjectDetected eseményére a saját metódusunkkal.

        myCraft.Velocity = 15.5; // Beállítjuk a jármű sebességét 15.5-re (mivel ez pozitív, nem dob kivételt).
        Console.WriteLine("Jármű sebessége beállítva: " + myCraft.Velocity); // Kiírjuk az aktuális sebességet ellenőrzésképp.

        Console.WriteLine("Szimuláció indítása..."); // Kiírunk egy tájékoztató szöveget.

        for (int i = 0; i < 20; i++) // Indítunk egy ciklust, ami 20-szor fog lefutni, hogy szimuláljunk 20 lépést.
        {
            myCraft.Iterate(); // Meghívjuk a jármű szimulációs lépését.
            
            Thread.Sleep(20); // Várunk 20 ezredmásodpercet. (Erre azért van szükség, mert a new Random() a rendszeridő alapján generál számot, és ha túl gyors a ciklus, mindig ugyanazt a számot kapnánk).
        }
    }

    static void OnObjectDetected(int size) // A saját eseménykezelő metódusunk, ami az esemény kiváltódásakor fut le.
    {
        Console.WriteLine("Figyelem! Akadály detektálva! Mérete: " + size); // Kiírjuk a konzolra az akadály észlelését és a kapott méretet.
    }
}
```