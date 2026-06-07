# 9. Előadás: Létrehozási minták és Dependency Injection

Bár a Dependency Injection (DI) nem klasszikus GoF minta, logikailag a létrehozási mintákhoz tartozik, mivel a célja az objektumok létrehozásának rugalmasabbá tétele.

## 1. Dependency Injection (Függőséginjektálás)

### A probléma: Merev függőségek
Képzelj el egy `SecurityService` osztályt, ami a felhasználók jelszavát kezeli, és az adatokat egy adatbázisba menti. Ha a kódba "beleégetjük" (hardcode), hogy az osztály maga hozza létre a `UserRepository` példányt a `new` kulcsszóval, akkor:
* A szolgáltatás rugalmatlan lesz (csak egy konkrét adatbázissal tud működni).
* Lehetetlen lesz izoláltan egységtesztelni (`unit test`), mert a teszt során is mindig az éles adatbázishoz akar majd kapcsolódni.

### A megoldás lépései:
1. **Strategy minta alkalmazása:** Hozzunk létre egy `IUserRepository` interfészt, és készítsünk hozzá több implementációt (pl. `DbUserRepository` az éles működéshez, `MemUserRepository` a gyors teszteléshez).
2. **Dependency Injection:** A `SecurityService` ne hozza létre a repository-t, hanem kapja meg kívülről (injektáljuk), leggyakrabban a **konstruktorán keresztül**.



### Kódpélda és magyarázat:

```csharp
// 1. Az interfész és az implementációk
public interface IUserRepository {
    string GetUserPassword(int userId);
    void UpdateUserPassword(int userId, string password);
}

public class DbUserRepository : IUserRepository { /* Éles adatbázis műveletek */ }
public class MemUserRepository : IUserRepository { /* Memória alapú műveletek teszthez */ }

// 2. A szolgáltatás, ami DI-t használ
public class SecurityService {
    private IUserRepository userRepository; // Interfészként hivatkozik rá

    // A függőséget kívülről kapja meg! Nem itt van a "new" operátor.
    public SecurityService(IUserRepository userRepository) {
        this.userRepository = userRepository;
    }

    public bool ChangePassword(int userId, string newPassword) {
        // A kapott repository-t használja anélkül, hogy tudná, melyik konkrét típus az.
        var oldPassword = userRepository.GetUserPassword(userId);
        // ... validáció ...
        userRepository.UpdateUserPassword(userId, newPassword);
        return true;
    }
}
```

### Hogyan használjuk a Dependency Injection-t?
* **Éles kódban:** `new SecurityService(new DbUserRepository());`
* **Teszteléskor:** `new SecurityService(new MemUserRepository());`

Ezáltal a `SecurityService` osztályt sosem kell módosítani, ha a perzisztenciát (adatkezelést) lecseréljük.

* **Megjegyzés:** Ha futás közben kell implementációt cserélni, a konstruktor helyett setter metódust (pl. `SetUserRepository`) is használhatunk.

---

## 2. Singleton (Egyke)
Célja: Biztosítja, hogy egy adott osztályból csakis egyetlen egy példány jöhessen létre a program futása során, és ehhez az egy példányhoz globális hozzáférést ad. Ilyen lehet például egy központi ablakkezelő (`WindowManager`).

### Megvalósítás:
* Az osztály maga felelős a saját példányáért, amit egy privát, statikus változóban tárol.
* A konstruktora védett (`private` vagy `protected`), így kívülről senki nem tudja a `new` operátorral hívni.
* Egy statikus metódus vagy property adja vissza a példányt.



### Kódpélda:
```csharp
public class WindowManager {
    // Statikus változó inicializálása (ez C#-ban automatikusan szálbiztos is!)
    private static WindowManager instance = new WindowManager();
 
    // Védett konstruktor - senki más nem hozhat létre újat!
    protected WindowManager() { }
 
    // Globális hozzáférési pont (Property)
    public static WindowManager Instance {
        get { return instance; }
    }
 
    public void DoSomething() { /* ... */ }
}
 
// Használat:
WindowManager.Instance.DoSomething();
```
### Vizsga tipp / Anti-pattern: A Singleton
A Singleton használatát gyakran "anti-pattern"-nek tekintik. 
* **Miért?** A `GetInstance()` (vagy `Instance`) mindig egy konkrét osztálytípushoz kötődik. Ez megnehezíti a unit tesztelést, mert a tesztek során nem tudunk a helyére könnyedén egy mock/fake objektumot tenni. 
* **Alternatíva:** A legtöbb esetben a **Dependency Injection** egy rugalmasabb és tesztelhetőbb megoldás.

---

## 3. Abstract Factory (Absztrakt gyár)
**Célja:** Interfészt biztosít ahhoz, hogy egymással összefüggő objektumok egész családjait hozzuk létre anélkül, hogy a konkrét osztályaikat megadnánk.

### A probléma: Témaváltás (Skins)
Képzelj el egy GUI-t, aminek támogatnia kell a Windows és az OSX megjelenést is. Ha a kódba beégetjük, hogy `new Win10Button()`, akkor a téma váltásakor az egész kódot át kell írni.

### A megoldás:
* **GUIFactory interfész:** Megadjuk, miket lehet gyártani (pl. `CreateWindow()`, `CreateButton()`), amik általános interfészekkel (`Window`, `Button`) térnek vissza.
* **Konkrét gyárak:** Létrehozunk gyárakat az egyes témákhoz (pl. `Win10GUIFactory`), amik konkrét termékeket adnak vissza.
* **Kliens:** Csak az absztrakt `GUIFactory`-t ismeri, így független a konkrét témától.



### Kódpélda:
```csharp
// 1. Az absztrakt termékek és a gyár interfésze
public interface Window { void Paint(); }
public interface Button { void Paint(); }
 
public interface GUIFactory {
    Window CreateWindow();
    Button CreateButton();
}
 
// 2. Konkrét gyár egy adott termékcsaládhoz (Win10)
public class Win10GUIFactory : GUIFactory {
    public Window CreateWindow() { return new Win10Window(); }
    public Button CreateButton() { return new Win10Button(); }
}
 
// 3. A Kliens, ami független a konkrét témától
public class Client {
    private Window wnd;
    private Button button;
    private GUIFactory factory;
 
    // A kliens megkapja, hogy épp melyik gyárat kell használnia
    public void SetFactory(GUIFactory fy) { this.factory = fy; }
 
    public void InitGUIElements() {
        // A new helyett a gyárat használja!
        wnd = factory.CreateWindow();
        button = factory.CreateButton();
    }
}
```

### Abstract Factory: Értékelés
* **Előnyök:** Nagyon könnyű kicserélni egy teljes termékcsaládot (csak a gyárat kell máshogy beállítani), és garantálja a konzisztenciát (nem keverednek az elemek, pl. OSX ablakhoz nem kerül Win10 gomb).
* **Hátrányok:** Ha be akarunk vezetni egy új elemet (pl. `Checkbox`), akkor a `GUIFactory` interfészt, és az **összes** létező konkrét gyárat módosítani kell.

---

## 4. Factory Method (Gyártó metódus)
**Célja:** Interfészt definiál objektumok létrehozására, de a pontos példányosítást a leszármazott osztályra bízza. Gyakran hívják "virtuális konstruktornak" is. 
*Valójában a `Template Method` minta alkalmazása objektumok létrehozására.*

### A probléma: Keretrendszer fejlesztése
Írunk egy általános keretrendszert (pl. dokumentumkezelőt), ami tudja, hogy a "Fájl/Megnyitás" esetén létre kell hozni egy új dokumentumot. De a keretrendszer írója még nem tudhatja, hogy a használó fejlesztő szöveges (`TextDocument`) vagy képi dokumentumot fog-e használni, tehát nem írhatja be a kódba a `new TextDocument()` hívást.

### A megoldás:
* Az `Application` ősosztályba teszünk egy absztrakt/virtuális metódust: `CreateDocument()`.
* A keretrendszer az általános logikában ezt hívja meg.
* A fejlesztő leszármazik az `Application`-ből, és felülírja a `CreateDocument()` metódust, ahol immár a `new` operátorral létrehozza a konkrét dokumentumot.



### Kódpélda:
```csharp
// --- KERETRENDSZER (Nem ismeri a konkrét dokumentum típust) ---
public abstract class Document { public abstract void Load(); }

public abstract class Application {
    private List<Document> documents = new List<Document>();

    // A Factory Method! Absztrakt, a leszármazott fogja megírni.
    protected abstract Document CreateDocument();

    // Közös logika, ami használja a Factory Methodot
    public void OpenDocument() {
        Document doc = CreateDocument(); // new operátor helyett!
        documents.Add(doc);
        doc.Load();
    }
}

// --- KONKRÉT ALKALMAZÁS (Ismeri a konkrét típusokat) ---
public class TextDocument : Document {
    public override void Load() { /* szöveg betöltése */ }
}

public class TextEditorApplication : Application {
    // Itt történik a konkrét példányosítás!
    protected override Document CreateDocument()
    {
        return new TextDocument();
    }
}
```

## 5. Observer (Megfigyelő) - Viselkedési minta

**Célja:** Egy az egyhez (vagy egy a többhöz) kapcsolatot alakít ki objektumok között. Ha a megfigyelt objektum (Subject) állapota megváltozik, a tőle függők (Observers) automatikusan értesülnek róla, anélkül, hogy erősen össze lennének csatolva ("drótozva").

### A probléma: Nézetek szinkronizálása
Van egy táblázatkezelőnk, amelynek több nézete van (táblázat, tortadiagram, oszlopdiagram). Ha az adatot módosítjuk, minden nézetnek frissülnie kell. Ha a nézetek közvetlenül egymást hívogatják, az "áttekinthetetlen káoszt" eredményez, és új nézet hozzáadásakor az összes régit módosítani kell.

### A megoldás (Document-View architektúra):
* **Subject (Document):** Az adatok "hiteles forrása". Tartalmaz egy listát a feliratkozott `Observer`-ekről. Amikor változás történik, meghív egy értesítő metódust (`Notify`/`UpdateAllViews`), ami végigmegy a listán. Csak az általános `Observer` interfészt ismeri.
* **Observer (View):** Rendelkezik egy `Update()` metódussal. Amikor a Subject meghívja, a nézet lekérdezi az új adatokat a Subject-től és újrarajzolja magát.



### Kódpélda:
```csharp
// Az Observer interfész, amit minden nézetnek meg kell valósítania
public interface View {
    void Update();
}
 
// A Subject, ami az adatokat tárolja és értesít
public class ExcelDocument {
    private double[,] data;
    private List<View> views = new List<View>(); // Lista a megfigyelőkről
 
    public void AttachView(View v) { views.Add(v); }
 
    public void SetCellData(int col, int row, double value) {
        data[col, row] = value;
        UpdateAllViews(); // Adat módosult -> Értesítés!
    }
 
    private void UpdateAllViews() {
        foreach (var v in views) {
            v.Update(); // Minden feliratkozott nézetnek szól
        }
    }
 
    public double[,] GetData() { return data; }
}
 
// Egy konkrét megfigyelő (Nézet)
public class PieChartView : View {
    private ExcelDocument document; // Referencia a Subject-re
 
    public PieChartView(ExcelDocument doc) {
        this.document = doc;
        doc.AttachView(this); // Feliratkozik a változásokra
    }
 
    // Ezt hívja a Subject, ha változás van
    public void Update() {
        var state = document.GetData(); // Lekéri a legújabb adatot
        Draw(state);
    }
 
    public void Draw(double[,] state) { /* Tortadiagram rajzolása */ }
}
```

### Push vs. Pull modell
A korábbi példában az Observer a változás tényéről értesült, majd lekérte az adatokat.

* **Pull modell:** A Subject csak a változást jelzi, az Observer utána lekéri (pull) az adatokat. (Rugalmasabb, ha az Observer csak részinformációkra kíváncsi).
* **Push modell:** A Subject az új adatokat paraméterként átküldi az `Update(ujAdat)` metódusban. (Ez főleg kis méretű adatok esetén praktikus és gyorsabb).

### Érdekesség: .NET események (Events)
A C# nyelv saját beépített eseménykezelő mechanizmusa (`event` és `delegate`) lényegében az **Observer minta** nyelvszintű implementációja. Amikor a `+=` operátorral feliratkozunk egy eseményre, valójában egy "megfigyelőt" adunk hozzá a Subject eseménykezelő listájához.