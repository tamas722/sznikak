# 1. Nyelvi eszközök


Készítsen egy Aggregate nevű osztályt, mely egy összegzőt valósít meg!

Az osztály egy int típusú, sum nevű tagváltozóban tárol egy aktuális összértéket. A tagváltozó értéke egy Sum nevű tulajdonságon (property) keresztül állítható be (6000-nél nagyobb érték esetén dobjon kivételt!), a lekérdezésére nincs lehetőség!

Az osztálynak van egy IncludeValue nevű művelete, mely egy int paraméterrel rendelkezik, és ezt hozzáadja a sum tagváltozó értékéhez.

Amennyiben az IncludeValue művelet hívásakor a sum értéke páratlan szám lesz, akkor az osztály süssön el egy IsOdd nevű eseményt, paraméterben megadva, hogy az adott pillanatig összesen hányszor került IncludeValue hívásra az adott objektumon (beleértve az adott esetet).

Adja meg az Aggregate osztály teljes C# nyelvű forráskódját és egy pár soros példaprogramot, mely létrehoz egy példányt belőle, feliratkozik az eseményére, és annak minden kiváltódásakor kiírja a konzolra az esemény paraméterét. A megvalósítás során nem használhatja a .NET keretrendszer beépített eseménytípusait. (EventHandler, Func, Action és hasonlókat). (15)

```csharp
using System; // Beimportáljuk a System névteret, amely a Console és az Exception (kivétel) osztályok használatához szükséges.

public delegate void OddSumDelegate(int callCount); // Saját delegált típus deklarálása az eseményhez. Egy int paramétert vár, ami a hívások számát jelöli majd.

public class Aggregate // Az összegzőt megvalósító osztály deklarálása.
{
    private int sum; // Privát tagváltozó az aktuális összérték tárolására. Kívülről nem elérhető közvetlenül.
    
    private int callCount = 0; // Privát segédváltozó, amely azt számolja, hányszor hívták meg az IncludeValue metódust (kezdetben 0).

    public int Sum // A Sum nevű tulajdonság (property) deklarálása.
    {
        // Nem írunk 'get' blokkot, mert a feladat kikötötte, hogy a lekérdezésére nincs lehetőség (write-only property).
        set // A set blokk fut le, amikor az osztályon kívülről értéket akarunk adni a Sum-nak.
        {
            if (value > 6000) // Ellenőrizzük, hogy a beállítani kívánt érték (value) nagyobb-e 6000-nél.
            {
                throw new ArgumentException("Az összeg nem lehet nagyobb 6000-nél!"); // Ha nagyobb, kivételt dobunk.
            }
            sum = value; // Ha az érték érvényes, beállítjuk a privát sum tagváltozó értékét.
        }
    }

    public event OddSumDelegate IsOdd; // Az IsOdd (Páratlan) esemény deklarálása a saját delegált típusunk felhasználásával.

    public void IncludeValue(int value) // Az IncludeValue metódus, amely egy int paramétert vár.
    {
        sum += value; // Hozzáadjuk a kapott paraméter értékét az aktuális sum értékéhez.
        
        callCount++; // Megnöveljük a hívásszámlálót 1-gyel, mivel most hívták meg a metódust.

        if (sum % 2 != 0) // Ellenőrizzük, hogy a sum új értéke páratlan-e (a 2-vel való osztás maradéka nem egyenlő 0-val).
        {
            if (IsOdd != null) // Ellenőrizzük, hogy van-e feliratkozó az eseményre (nehogy NullReferenceException-t kapjunk).
            {
                IsOdd(callCount); // Ha páratlan és van feliratkozó, elsütjük az eseményt, átadva az eddigi hívások számát.
            }
        }
    }
}

class Program // A főprogram osztálya.
{
    static void Main(string[] args) // A belépési pont.
    {
        Aggregate myAgg = new Aggregate(); // Létrehozunk egy új példányt az Aggregate osztályból.

        myAgg.IsOdd += OnIsOddEvent; // Feliratkozunk az IsOdd eseményre az általunk írt OnIsOddEvent metódussal.

        myAgg.Sum = 10; // Beállítjuk a kezdőértéket 10-re (ez nem dob kivételt, mert kisebb 6000-nél).
        
        Console.WriteLine("Első hívás (IncludeValue(3))..."); // Tájékoztató szöveg a konzolra.
        myAgg.IncludeValue(3); // A sum 13 lesz (páratlan). Ez az 1. hívás, így az esemény kiváltódik.

        Console.WriteLine("Második hívás (IncludeValue(1))..."); // Tájékoztató szöveg a konzolra.
        myAgg.IncludeValue(1); // A sum 14 lesz (páros). Ez a 2. hívás, az esemény NEM váltódik ki.

        Console.WriteLine("Harmadik hívás (IncludeValue(5))..."); // Tájékoztató szöveg a konzolra.
        myAgg.IncludeValue(5); // A sum 19 lesz (páratlan). Ez a 3. hívás, így az esemény ismét kiváltódik.
    }

    static void OnIsOddEvent(int callCount) // A saját eseménykezelő metódusunk.
    {
        Console.WriteLine("Az összeg páratlan lett! Az IncludeValue hívások száma eddig: " + callCount); // Kiírja a konzolra az esemény paraméterét.
    }
}
```