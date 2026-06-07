# Tervezési minták 

Jellemezze röviden az "Adapter" tervezési mintát, annak object adapter variánsát! Mire ad megoldást a "Adapter" tervezési minta? Mutassa be általánosságban vagy a minta egy alkalmazásán keresztül (elég az egyik) a minta működését! Ezen belül rajzolja fel a minta osztálydiagramját, valamint röviden adja meg a mintában szereplő osztályok szerepét! Az osztálydiagramon a minta működésének szempontjából kritikus metódusok esetében pszeudokódot is adjon meg!

* **Megoldás:**

1. Mire ad megoldást az "Adapter" (Illesztő) tervezési minta?

Az Adapter tervezési minta célja, hogy inkompatibilis interfésszel rendelkező osztályok együttműködését tegye lehetővé. Egy meglévő osztály (vagy komponens) interfészét olyan interfésszé alakítja át (burkolja be), amilyet a kliens (felhasználó kód) elvár. Emiatt a mintát gyakran Wrapper (Csomagoló) néven is említik.

Tipikus példa a való életből: amikor egy európai csatlakozójú laptop töltőt (Adaptee) egy amerikai konnektorba (Target) szeretnénk bedugni (Client), szükségünk van egy hálózati adapterre (Adapter), ami a két inkompatibilis felület között közvetít.

2. Az "Object adapter" (Objektum adapter) variáns jellemzése

Az Adapter mintának két fő variánsa létezik: az osztály adapter (amely többszörös öröklést használ) és az objektum adapter.

Az objektum adapter variáns a funkciók eléréséhez kompozíciót (objektum referenciát) és delegációt használ öröklés helyett.

- Az adapter osztály megvalósítja (implementálja) a kliens által elvárt interfészt.
- Belső állapotként (mezőként) tartalmaz egy referenciát a már meglévő, adaptálandó osztály (Adaptee) egy példányára.
- Rugalmasabb, mint az osztály adapter, mivel egyetlen adapter képes az Adaptee osztállyal és annak minden leszármazottjával is együttműködni.

3. Az osztálydiagram
Az osztálydiagram szöveges/karakteres formában:

```
+------------+          +-----------------------+
|   Client   | -------> | <<interface/abstract>>|
+------------+          |        Target         |
                         +-----------------------+
                         | + Request()           |
                         +-----------------------+
                                    ^
                                    | megvalósítja (implements)
                                    |
                         +-----------------------+          +-------------------+
                         |       Adapter         | -------> |      Adaptee      |
                         +-----------------------+  tartal- +-------------------+
                         | - adaptee: Adaptee    |  mazza   | + SpecificRequest |
                         +-----------------------+  (has-a) +-------------------+
                         | + Request()           |
                         +-----------------------+
```

4. A mintában szereplő osztályok szerepe

**Client (Kliens):** Az az osztály, amely az együttműködést kezdeményezi. Ő kizárólag a Target interfészt ismeri, azon keresztül próbál műveleteket végezni.

**Target (Cél):** Az az interfész (vagy absztrakt osztály), amelyet a kliens elvár és használni tud. Definiálja a Request() metódust.

**Adaptee (Adaptálandó):** A már meglévő, hasznos funkciót megvalósító osztály, amelynek az interfésze inkompatibilis a kliens elvárásaival (pl. Request() helyett SpecificRequest() metódusa van, vagy más paramétereket vár).

**Adapter (Illesztő):** Az az osztály, amely áthidalja a szakadékot. Megvalósítja a Target interfészt, de a kéréseket a háttérben a tartalmazott Adaptee objektum felé delegálja.

5. A minta működése és kritikus metódusok pszeudokódja

**Működés:** Amikor a kliens meghívja a Target interfészen a Request() metódust, a hívás valójában az Adapter példányhoz fut be. Az Adapter elvégzi a szükséges paraméter-konverziókat (ha kellenek), majd meghívja az Adaptee objektum SpecificRequest() metódusát. A visszatérési értéket (ha van) visszakonvertálja a kliens számára érthető formátumba, és visszaadja.

**Pszeudokód az Adapter osztályra** (kifejezetten a kritikus működésre fókuszálva):

```csharp
// A kliens által elvárt interfész
public interface Target {
    void Request();
}

// Az inkompatibilis meglévő osztály (Adaptee)
public class Adaptee {
    public void SpecificRequest() {
        // ... itt van a meglévő, hasznos logika ...
    }
}

// Az Adapter osztály, amely implementálja a Target-et
public class Adapter : Target {
    // Kompozíció: az Adapter tartalmaz egy referenciát az Adaptee-re
    private Adaptee adaptee;

    // Konstruktor, amelyen keresztül megkapja az adaptálandó objektumot
    public Adapter(Adaptee a) {
        this.adaptee = a;
    }

    // A kritikus metódus: A Target interfész megvalósítása
    public void Request() {
        // Esetleges paraméterkonverzió vagy előkészületek
        
        // Delegáció (továbbhívás) az Adaptee megfelelő metódusához
        adaptee.SpecificRequest();
        
        // Esetleges utólagos logika vagy visszatérési érték konverziója
    }
}
```