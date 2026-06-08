# Tervezési minták (20p)

Jellemezze röviden az "Observer" tervezési mintát! Mire ad megoldást az "Observer" tervezési minta? Mutassa be általánosságban vagy a minta egy alkalmazásán keresztül (elég az egyik) a minta működését! Ezen belül rajzolja fel a minta osztálydiagramját, valamint röviden adja meg a mintában szereplő osztályok szerepét! Az osztálydiagramon a minta működésének szempontjából kritikus metódusok esetében pszeudokódot is adjon meg!

## 1. Mire ad megoldást az "Observer" (Megfigyelő) tervezési minta?

Az **Observer** egy **viselkedési (behavioral)** tervezési minta, amely egy **egy-a-többhöz (one-to-many)** függőséget hoz létre objektumok között. Fő célja, hogy ha egy objektum (a megfigyelt) állapota megváltozik, akkor erről az összes tőle függő objektum (a megfigyelők) automatikusan értesüljön és frissüljön.

Megoldást nyújt arra a problémára, amikor a rendszer különböző részeinek reagálnia kell egy adott komponens változásaira, de el akarjuk kerülni a **szoros csatolást (tight coupling)**. A megfigyelt objektumnak nem kell ismernie a megfigyelők konkrét osztályait, csupán egy közös interfészen keresztül kommunikál velük, így a megfigyelők listája futási időben dinamikusan változhat (bővíthető, szűkíthető).

## 2. A minta működésének bemutatása (YouTube csatorna példán keresztül)

A legkézenfekvőbb példa az Observer mintára egy hírlevél feliratkozás vagy egy YouTube csatorna.

Legyen a **YouTube Csatorna** a megfigyelt objektum (**Subject**). A csatornának vannak **Feliratkozói (Observers)**. A csatorna nem tudja pontosan, kik a feliratkozók (nem ismeri a konkrét életüket vagy osztályukat), csak azt tudja róluk, hogy van egy elérhetőségük (értesítési csatornájuk), ahová üzenetet küldhet.

Amikor a csatornára feltöltenek egy új videót (állapotváltozás), a rendszer végigmegy a feliratkozók listáján, és mindegyiküknek küld egy automatikus értesítést ("Hé, új videó került fel!"). Bárki bármikor feliratkozhat (**Attach**) és leiratkozhat (**Detach**).

## 3. Az osztálydiagram

```
  +-----------------------+                    +-----------------------+
  |      <<interface>>    |                    |      <<interface>>    |
  |        ISubject       |                    |       IObserver       |
  +-----------------------+                    +-----------------------+
  | + Attach(IObserver)   | o----------------> | + Update()            |
  | + Detach(IObserver)   |       értesít      +-----------------------+
  | + Notify()            |                              ^
  +-----------------------+                              | megvalósítja
             ^                                           |
             | megvalósítja                              |
  +-----------------------+                    +-----------------------+
  |    ConcreteSubject    |                    |   ConcreteObserver    |
  +-----------------------+                    +-----------------------+
  | - observers: List     | <----------------- | - subject: ISubject   |
  | - state               |      lekérdez      +-----------------------+
  +-----------------------+                    | + Update()            |
  | + GetState()          |                    +-----------------------+
  | + SetState()          |
  +-----------------------+
```

## 4. A mintában szereplő osztályok szerepe

- **ISubject (Megfigyelt interfész):** Deklarálja a metódusokat a megfigyelők feliratkoztatására (`Attach`), leiratkoztatására (`Detach`), valamint az értesítésükre (`Notify`).

- **IObserver (Megfigyelő interfész):** Általában egyetlen metódust (pl. `Update`) definiál, amelyet a Subject meghív, amikor állapotváltozás történik.

- **ConcreteSubject (Konkrét Megfigyelt):** Tárolja a megfigyelők listáját és a saját belső állapotát. Ha ez az állapot megváltozik, meghívja az ősétől/interfészből örökölt `Notify()` metódust. Szükség esetén metódusokat biztosít az állapot lekérdezésére (`GetState`).

- **ConcreteObserver (Konkrét Megfigyelő):** Megvalósítja az `IObserver` interfészt. Gyakran fenntart egy hivatkozást a konkrét Subject-re, hogy az értesítés (`Update`) megérkezése után le tudja kérdezni a megváltozott, friss állapotot.

## 5. A minta működése és kritikus metódusok pszeudokódja

**Működés:** A minta "lelke" a `ConcreteSubject` osztályban található `Notify()` metódus, amely egy iterátor vagy foreach ciklus segítségével végigmegy a beregisztrált `IObserver`-ek listáján, és mindegyiken meghívja az `Update()` metódust. Ez biztosítja az automatikus szinkronizációt.

**Pszeudokód a kritikus működésre:**

```csharp
// 1. Megfigyelő (Observer) interfész
public interface IObserver {
    void Update(); // Ezt hívja a Subject
}

// 2. Megfigyelt (Subject) interfész
public interface ISubject {
    void Attach(IObserver observer);
    void Detach(IObserver observer);
    void Notify();
}

// 3. Konkrét Megfigyelt (YouTube Csatorna)
public class YouTubeChannel : ISubject {
    // Feliratkozók listája
    private List<IObserver> _subscribers = new List<IObserver>();
    public string LatestVideoTitle { get; private set; }

    public void Attach(IObserver observer) {
        _subscribers.Add(observer);
    }

    public void Detach(IObserver observer) {
        _subscribers.Remove(observer);
    }

    // A KRITIKUS METÓDUS: Értesítés kiküldése minden feliratkozónak
    public void Notify() {
        foreach (IObserver subscriber in _subscribers) {
            subscriber.Update(); // Polimorfikus hívás
        }
    }

    // Állapotváltozást előidéző metódus
    public void UploadVideo(string title) {
        LatestVideoTitle = title;
        Console.WriteLine($"\nCsatorna: Új videó feltöltve -> '{title}'");
        Notify(); // Állapotváltozás történt, értesítünk mindenkit!
    }
}

// 4. Konkrét Megfigyelő (Feliratkozó Felhasználó)
public class Subscriber : IObserver {
    private string _name;
    private YouTubeChannel _channel;

    public Subscriber(string name, YouTubeChannel channel) {
        _name = name;
        _channel = channel;
    }

    // Amikor a Subject meghívja, lekérdezzük az új állapotot
    public void Update() {
        Console.WriteLine($"[{_name}] Értesítést kapott! Az új videó címe: {_channel.LatestVideoTitle}");
    }
}
```