# Tervezési minták (20p)

Jellemezze röviden a "Singleton" tervezési mintát! Mire ad megoldást a "Singleton" tervezési minta? Mutassa be általánosságban vagy a minta egy alkalmazásán keresztül (elég az egyik) a minta működését! Ezen belül rajzolja fel a minta osztálydiagramját, valamint röviden adja meg a mintában szereplő osztályok szerepét! Az osztálydiagramon a minta működésének szempontjából kritikus metódusok esetében pszeudokódot is adjon meg!

## 1. Mire ad megoldást a "Singleton" (Egyke) tervezési minta?

A **Singleton** egy **létrehozási (creational)** tervezési minta, amely két alapvető problémára ad megoldást egyszerre:

1. **Garantálja**, hogy egy osztályból pontosan egyetlen példány (instance) létezzen a program futása során.
2. Ehhez az egyetlen példányhoz **globális hozzáférési pontot** biztosít.

Olyan esetekben használjuk, amikor fogalmilag is csak egyetlen "valamiből" létezhet a rendszerben, és ha többet hoznánk létre belőle, az inkonzisztens állapothoz, erőforrás-ütközéshez vagy felesleges memóriafogyasztáshoz vezetne (például konfiguráció-kezelők, adatbázis-kapcsolatkészletek (connection pool), hardvereszköz-vezérlők vagy naplózók/loggerek esetén).

## 2. A minta működésének bemutatása (Naplózó / Logger példán keresztül)

Képzeljünk el egy nagyvállalati szoftvert, amely működés közben folyamatosan egy központi naplófájlba (log.txt) írja a hibákat és az eseményeket.

Ha a program különböző részei (pl. adatbázis modul, hálózati modul, felhasználói felület) mind saját maguk hoznának létre egy Logger objektumot (példányosítanák az osztályt), akkor mindegyik megpróbálná megnyitni ugyanazt a fájlt írásra. Ez operációs rendszer szintű fájlzárolási (lock) hibákhoz vezetne.

A megoldás, hogy a Logger osztály **Singletonként** van megvalósítva: megakadályozza, hogy bárki szabadon példányosítsa (letiltja a publikus konstruktort). Ehelyett ad egy `GetInstance()` nevű statikus metódust. Amikor a hálózati modul elkéri a Loggert, a metódus létrehozza az első és egyetlen példányt. Amikor később az adatbázis modul is kéri a Loggert, a `GetInstance()` felismeri, hogy már létrejött a példány, és ugyanazt adja vissza neki is. Így mindenki ugyanazt az egyetlen naplózót használja.

## 3. Az osztálydiagram

```
  +-----------------------------------+
  |             Singleton             |
  +-----------------------------------+
  | - instance: Singleton             |  <-- A statikus, egyetlen példányt tároló változó
  +-----------------------------------+
  | - Singleton()                     |  <-- PRIVÁT konstruktor!
  | + GetInstance(): Singleton        |  <-- Publikus, statikus lekérdező metódus
  | + DoSomething()                   |
  +-----------------------------------+
```

## 4. A mintában szereplő osztályok szerepe

Ennél a mintánál kivételesen csak egyetlen osztály szerepel:

- **Singleton (Egyke):** Felelős azért, hogy saját magából pontosan egy példányt hozzon létre, és azt karbantartsa.
  - **Privát konstruktorral** rendelkezik, így a külső osztályok (kliensek) nem tudják a `new` kulcsszóval példányosítani.
  - Rendelkezik egy **privát statikus változóval**, amelyben a saját, egyetlen példányát tárolja.
  - Nyújt egy **publikus statikus metódust** (általában `GetInstance()` vagy `Instance` property néven), amelyen keresztül a kliensek hozzáférhetnek az egyetlen példányhoz.

## 5. A minta működése és kritikus metódusok pszeudokódja

**Működés:** A kritikus rész a példányosítás szabályozása a `GetInstance()` metóduson belül. **Lusta inicializálást (lazy initialization)** alkalmazva a példány csak akkor jön létre, amikor valaki először meghívja a metódust. Fontos odafigyelni a **többszálú (multi-threaded)** környezetekre is: ha két szál egyszerre fut be a még üres (`null`) állapotba, véletlenül két példány jöhetne létre. Ezt szálbiztos (thread-safe) módon, pl. zárolással (`lock`) kell kivédeni.

**Pszeudokód a kritikus működésre:**

```csharp
public class Logger {
    // 1. Privát, statikus változó az egyetlen példány tárolására
    private static Logger _instance;
    
    // Zároló (lock) objektum a szálbiztos működéshez
    private static readonly object _lock = new object();

    // 2. PRIVÁT konstruktor: külső kód nem hívhatja meg!
    private Logger() {
        Console.WriteLine("Logger példányosítva (Ez csak egyszer fog megtörténni).");
    }

    // 3. A KRITIKUS METÓDUS: Globális, statikus hozzáférési pont
    public static Logger GetInstance() {
        // Ellenőrizzük, hogy létezik-e már a példány
        if (_instance == null) {
            
            // Zárolás: csak egy szál léphet be egyszerre (Thread-safety)
            lock (_lock) {
                
                // Dupla ellenőrzés (Double-checked locking) a biztonság kedvéért
                if (_instance == null) {
                    _instance = new Logger(); // Itt hívjuk a privát konstruktort
                }
            }
        }
        
        // Visszaadjuk a már garantáltan létező egyetlen példányt
        return _instance;
    }

    // Normál üzleti logika metódusai
    public void LogMessage(string message) {
        Console.WriteLine($"[LOG]: {message}");
    }
}
```