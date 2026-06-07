# 8. Előadás: Command minta

## 1. Command (Parancs) minta
**Célja:** Egy kérést objektumként egységbezárni, ami lehetővé teszi kérések sorba állítását, naplózását, visszavonását (undo), és a kliens felparaméterezését különböző kérésekkel.

### Struktúra és működés
* **Invoker (pl. MenuItem):** Meghívja a parancsot (`Execute()`).
* **Command (Interface):** Deklarálja az `Execute()` metódust.
* **ConcreteCommand (pl. PasteCommand):** Implementálja a konkrét logikát.
* **Receiver (pl. TextDocument):** Az objektum, amelyen a művelet végrehajtódik.



### Kódpélda:
```csharp
// Command interfész
interface ICommand { void Execute(); }
 
// Receiver
class TextDocument { public void Paste() { /* ... */ } }
 
// ConcreteCommand
class PasteCommand : ICommand {
    TextDocument doc;
    public PasteCommand(TextDocument d) { doc = d; }
    
    // Delegálás a Receiver felé
    public void Execute() { doc.Paste(); } 
}
```

### A Command minta magyarázata
A `PasteCommand` elkapja a `TextDocument`-et, és az `Execute`-ban meghívja a `Paste()` metódust. Az Invoker (pl. menüelem) csak annyit tud, hogy van egy `ICommand` objektuma, aminek meghívja az `Execute()`-ját, így a menüelem nem függ a konkrét alkalmazáslogikától.

---

## 2. Command Processor
A Command minta olyan változata, amely beépítetten támogatja a visszavonást (Undo).

* **Bővítés:** Az interfészbe bekerül az `UnExecute()` művelet.
* **CommandProcessor osztály:** Tárolja a már végrehajtott parancsokat (stack/lista).
* **Működés:** A parancsok futtatását és visszavonását rajta keresztül végezzük (`ExecuteCommand`, `UnExecuteLastCommand`). A parancsoknak az `Execute()` során el kell menteniük a visszavonáshoz szükséges állapotot.

---

## 3. Memento
**Célja:** Az egységbezárás (encapsulation) megsértése nélkül elmenteni egy objektum belső állapotát, hogy később visszaállítható legyen.

### Szereplők:
* **Originator (pl. TextDocument):** Az objektum, aminek az állapotát mentjük/visszaállítjuk.
* **Memento:** Tárolja az állapotot (tükörképe az Originator tagváltozóinak).
* **CareTaker (pl. Command):** Nyilvántartja a memento objektumot, de nem módosítja.

**Példa logikája:**
1. `Originator.CreateMemento()`: Visszaad egy `Memento` objektumot az aktuális állapotokkal.
2. `Originator.RestoreFromMemento(m)`: A paraméterként kapott memento alapján visszaállítja a belső tagváltozókat.



---

## 4. Adapter (Wrapper)
**Célja:** Egy osztály interfészét a kliens által elvárt interfésszé alakítani. Akkor használjuk, ha egy meglévő osztályt (`Adaptee`) nem tudunk módosítani, de használni szeretnénk egy másik interfész környezetében.

### Változatok:
* **Object Adapter:** Tartalmaz egy hivatkozást az `Adaptee`-re (delegálás).
* **Class Adapter:** Leszármaztatással éri el az `Adaptee`-t.

### Object Adapter kód:
```csharp
class EditableTextShape : Shape { // Target interfész/osztály
    TextBox textBox; // Adaptee
    
    public override void GetBoundingBox() { 
        return textBox.GetExtent(); // Delegálás
    }
}
```

## 5. Facade (Homlokzat)
**Célja:** Egységes, egyszerűsített interfészt biztosít egy komplex alrendszer (számos osztály) eléréséhez. 
* **Haszna:** Csökkenti a kliens és az alrendszer közötti függőséget, hordozhatóbbá teszi a kódot.
* **Figyelem:** Ha az alrendszer túl komplex, a Facade is túl nagy lehet – ilyenkor érdemes logikailag darabolni.



---

## 6. Composite (Összetett)
**Célja:** Objektumokat fastruktúrába rendezni rész-egész viszonyban, és lehetővé tenni az elemi (`Leaf`) és összetett (`Composite`) objektumok egységes kezelését.

### Struktúra:
* **Component:** Közös interfész.
* **Leaf:** Az elemi elem.
* **Composite:** Tartalmazza a gyerekeket, a műveletet a gyerekeken rekurzívan végzi el.

*Megjegyzés:* A gyerekkezelő műveletek (`Add`, `Remove`) csak a `Composite` osztályon értelmezettek, az elemi `Leaf`-en nem (itt gyakran üres implementációt vagy kivételt kapunk).



---

## 7. Proxy (Helyettes)
**Célja:** Egy objektum helyett egy helyettesítőt használ, amely szabályozza a hozzáférést a tényleges objektumhoz.

### Felhasználási módok:
* **Virtuális Proxy:** Nagy erőforrásigényű objektumok igény szerinti (lazy) létrehozása (pl. képek betöltése csak megjelenítéskor).
* **Védelmi Proxy:** Jogosultságok szabályozása.
* **Távoli Proxy:** Távoli objektumok lokális elérése.

### Virtuális Proxy kód:
```csharp
public void Draw() {
    if (image == null) { 
        image = new Image(); // Igény szerinti betöltés
        image.Load();
    }
    image.Draw();
}
```

### Magyarázat a Proxyhoz
A kliens számára a proxy pont úgy néz ki, mint az eredeti objektum (pl. `Graphic` interfész), de elrejti a betöltés komplexitását, amíg arra nincs feltétlenül szükség.

---

### Vizsgatitkok (Design Patterns)

* **Adapter:** Figyelj az **Object vs. Class** különbségre!
    * *Object Adapter:* Delegálást használ (hivatkozás egy példányra).
    * *Class Adapter:* Öröklést használ (interfészeken keresztül).

* **Composite:** Értsd meg, hogy a kliens számára **transzparens** (láthatatlan), hogy egyetlen elemre (`Leaf`) vagy egy egész fára (`Composite`) hívja a műveletet; mindkettőt ugyanazon közös interfészen keresztül kezeli.

* **Proxy:** Ne csak "közvetítőként" gondolj rá! A Proxy az objektumhoz való **hozzáférést szabályozó** elem (pl. jogosultságok ellenőrzése, erőforrás-takarékosság, távoli elérés).