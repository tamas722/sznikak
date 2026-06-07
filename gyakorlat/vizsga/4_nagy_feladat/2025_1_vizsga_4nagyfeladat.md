# Tervezési minták (20p)

Jellemezze röviden a "Composite" tervezési mintát! Mire ad megoldást a "Composite" tervezési minta? Mutassa be általánosságban vagy a minta egy alkalmazásán keresztül (elég az egyik) a minta működését! Ezen belül rajzolja fel a minta osztálydiagramját, valamint röviden adja meg a mintában szereplő osztályok szerepét! Az osztálydiagramon a minta működésének szempontjából kritikus metódusok esetében pszeudokódot is adjon meg!

## 1. Mire ad megoldást a "Composite" (Összetétel) tervezési minta?

A Composite tervezési minta egy szerkezeti (structural) minta, amelynek fő célja, hogy objektumokat fa-struktúrába (rész-egész hierarchiába) szervezzen, és lehetővé tegye a kliensek számára, hogy az egyes objektumokat (leveleket) és az objektumok kompozícióit (összetett csomópontokat) egységesen kezeljék.

Ennek köszönhetően a kliens kódjában nem kell telepakolni a programot if-else vagy típusvizsgáló (pl. `is`, `instanceof`) utasításokkal attól függően, hogy egyetlen elemmel vagy egy bonyolult, több elemből álló csoporttal van-e dolga. A kliens csak a közös interfészt hívja, és a minta a háttérben rekurzívan elintézi a többit.

## 2. A minta működésének bemutatása (Fájlrendszer példán keresztül)

Képzeljünk el egy operációs rendszer fájlkezelőjét. Vannak benne **Fájlok** (egyszerű elemek) és **Mappák** (összetett elemek, amik tartalmazhatnak Fájlokat és további Mappákat is).

Ha a felhasználó ki akarja számolni egy mappa méretét, a mappa végigmegy a benne lévő összes elemen. Ha az elem egy fájl, lekéri a méretét. Ha az elem egy almappa, akkor annak is kiadja a "kérlek, add meg a méreted" parancsot (ami ugyanígy fog eljárni). A kliensnek elég a legfelső elem (mappa vagy fájl) `MéretLekérdezése()` metódusát meghívni, a többi a struktúrán végigfutva automatikusan megtörténik, hiszen mind a Mappa, mind a Fájl ugyanazt a "FájlrendszerElem" interfészt valósítja meg.

## 3. Az osztálydiagram (Mermaid)

```
  +------------+          +-----------------------+
  |   Client   | -------> | <<interface/abstract>>|
  +------------+          |      Component        | <-------------------------+
                          +-----------------------+                           |
                          | + Operation()         |                           |
                          | + Add(Component)      |                           |
                          | + Remove(Component)   |                           |
                          | + GetChild(int)       |                           |
                          +-----------------------+                           |
                                     ^                                        |
                                     | megvalósítja (implements)              |
                  +------------------+------------------+                     |
                  |                                     |                     |
      +-----------------------+              +-----------------------+        |
      |         Leaf          |              |       Composite       |        |
      +-----------------------+              +-----------------------+        |
      | + Operation()         |              | - children: List<Comp>| <>-----+
      +-----------------------+              +-----------------------+  tartalmaz
                                             | + Operation()         |  (0..* Component)
                                             | + Add(Component)      |
                                             | + Remove(Component)   |
                                             +-----------------------+
```

**Megjegyzés:** Az `Add()`/`Remove()` metódusok helye a mintában egy klasszikus tervezési dilemma: betehetjük őket a `Component`-be is – ekkor a kliensnek kényelmesebb, de a `Leaf`-ben üresen kell hagyni vagy kivételt kell dobni; illetve tehetjük csak a `Composite`-ba – ez biztonságosabb, de a kliensnek tudnia kell, mikor dolgozik `Composite`-tal. A GoF könyv az előbbit (`Component` szintű definíció) javasolja a teljes transzparencia érdekében.

## 4. A mintában szereplő osztályok szerepe

- **Component (Komponens)**: Absztrakt bázisosztály vagy interfész a kompozíció minden eleméhez. Deklarálja a közös műveleteket (pl. `Operation()`), amelyeket mind a levelek, mind az összetett elemek meg fognak valósítani.

- **Leaf (Levél)**: Fa-struktúra legalsó eleme, nincsenek gyerekei. Ő végzi el a tényleges, érdemi munkát az `Operation()` metódusban.

- **Composite (Összetett / Csomópont)**: Olyan elem, amelynek vannak gyerekei (amik lehetnek további `Composite`-ok vagy `Leaf`-ek). Tárolja a gyerek-komponenseket, és az `Operation()` hívást továbbítja (delegálja) nekik.

- **Client (Kliens)**: A `Component` interfészen keresztül kommunikál a struktúra elemeivel, anélkül, hogy érdekelné, hogy épp egy levéllel vagy egy egész ággal van-e dolga.

## 5. A minta működése és kritikus metódusok pszeudokódja

**Működés:** A minta működésének magja a `Composite` osztály `Operation()` metódusa. Amikor ezt a kliens meghívja, a `Composite` nem maga próbálja megoldani a feladatot, hanem egy ciklussal végigmegy a belső listájában (`children`) tárolt összes gyerekén, és mindegyikre meghívja az `Operation()`-t (rekurzió). Az eredményeket esetleg összesíti.

**Pszeudokód a kritikus működésre:**

```csharp
// Közös komponens interfész
public abstract class Component {
    public abstract void Operation();
    
    // Alapértelmezett implementációk (hogy a levélnek ne kelljen megírnia)
    public virtual void Add(Component c) { throw new NotSupportedException(); }
    public virtual void Remove(Component c) { throw new NotSupportedException(); }
}

// Levél osztály (egyszerű elem, pl. Fájl)
public class Leaf : Component {
    public override void Operation() {
        // A levél elvégzi a konkrét feladatot
        Console.WriteLine("Levél elvégzi a munkát.");
    }
}

// Összetett osztály (tartalmazó elem, pl. Mappa)
public class Composite : Component {
    // A gyerek elemeket tároló lista (lehetnek Levelek és más Composite-ok is)
    private List<Component> children = new List<Component>();

    public override void Add(Component c) {
        children.Add(c);
    }

    public override void Remove(Component c) {
        children.Remove(c);
    }

    // A KRITIKUS METÓDUS: a kérés továbbítása (delegálása) a gyerekeknek
    public override void Operation() {
        Console.WriteLine("Composite delegálja a feladatot a gyerekeinek:");
        
        // Végigmegyünk minden tárolt gyereken (legyen az Leaf vagy másik Composite)
        foreach (Component child in children) {
            child.Operation(); // Rekurzív hívás!
        }
    }
}