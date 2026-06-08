# 1. Nyelvi eszközök


Készítsen egy Craft nevű osztályt, mely egy önvezető vízi járművet reprezentál egy szimulátor alkalmazásban! Az osztály egy double típusú, velocity nevű tagváltozóban tárolja a jármű aktuális sebességét. A tagváltozó értéke egy Velocity nevű tulajdonságon (property) keresztül kérdezhető le és állítható be (0-nál kisebb érték esetén dobjon kivételt!).

Az osztály ezen felül tartalmaz egy ObjectDetected eseményt (C# event-et). Az esemény akkor váltódik ki, amikor a jármű a szimuláció léptetése során (Iterate művelet) egy akadályt detektál, és paraméterben átadja a detektált akadály méretét (int típusú). Az Iterate műveletben az akadály detektálása véletlenszerűen történik: az Iterate akkor tekinti úgy, hogy a jármű akadályt észlel, amikor a new Random().Next(100) utasítás 76-nál nagyobb értéket ad vissza, a detektált akadály mérete pedig maga a generált véletlenszám.

Adja meg a Craft osztály teljes C# nyelvű forráskódját és egy pár soros példaprogramot, mely létrehoz egy példányt belőle, feliratkozik az eseményére, és annak minden kiváltódásakor kiírja a konzolra az esemény paraméterét. A megvalósítás során nem használhatja a .NET keretrendszer beépített eseménytípusait. (15p)

```csharp
using System;

namespace SimulatorApp
{
    // 1. Saját delegált típus deklarálása (nem .NET beépített)
    public delegate void ObjectDetectedEventHandler(int size);

    public class Craft
    {
        // Tagváltozó a sebesség tárolására
        private double velocity;

        // Tulajdonság (Property) lekérdezésre és beállításra
        public double Velocity
        {
            get { return velocity; }
            set
            {
                if (value < 0)
                {
                    throw new ArgumentOutOfRangeException(nameof(value), "A sebesség nem lehet negatív!");
                }
                velocity = value;
            }
        }

        // 2. Az esemény (event) definiálása a saját delegálttal
        public event ObjectDetectedEventHandler ObjectDetected;

        // A szimuláció léptetését végző művelet
        public void Iterate()
        {
            // A feladat által pontosan kért utasítás
            int detectedSize = new Random().Next(100);

            if (detectedSize > 76)
            {
                // 3. Esemény kiváltása, ha van feliratkozó
                ObjectDetected?.Invoke(detectedSize);
            }
        }
    }

    class Program
    {
        static void Main(string[] args)
        {
            // Példányosítás
            Craft autonomousBoat = new Craft();

            // 4. Feliratkozás az eseményre
            autonomousBoat.ObjectDetected += OnObstacleDetected;

            // Szimuláció futtatása néhányszor
            for (int i = 0; i < 10; i++)
            {
                autonomousBoat.Iterate();
            }

            Console.ReadLine();
        }

        // 5. Az eseménykezelő metódus, ami kiírja a paramétert
        static void OnObstacleDetected(int size)
        {
            Console.WriteLine($"Akadály detektálva! A mérete: {size}");
        }
    }
}
```
