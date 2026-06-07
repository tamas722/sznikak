# Tervezési minták

Jellemezze röviden a "Proxy" tervezési mintát! Mire ad megoldást a "Proxy" tervezési minta? Mutassa be általánosságban vagy a minta egy alkalmazásán keresztül (elég az egyik) a minta működését! Ezen belül rajzolja fel a minta osztálydiagramját, valamint röviden adja meg a mintában szereplő osztályok szerepét! Az osztálydiagramon a minta működésének szempontjából kritikus metódusok esetében pszeudokódot is adjon meg!

Íme a feladat teljes körű megoldása és magyarázata a "Proxy" (Helyettes) tervezési mintára vonatkozóan, az előző válaszhoz hasonló, strukturált formában.

## 1. Mire ad megoldást a "Proxy" (Helyettes) tervezési minta?

A Proxy tervezési minta (amely szintén egy szerkezeti/structural minta) célja, hogy egy másik objektumot helyettesítsen vagy képviseljen, ezáltal szabályozva a hozzáférést az eredeti objektumhoz. A Proxy ugyanazt az interfészt valósítja meg, mint az az objektum, amit helyettesít, így a kliens számára észrevehetetlen a különbség.

A hozzáférés-szabályozás célja leggyakrabban háromféle lehet (ezek a Proxy fő variánsai):

- **Virtuális Proxy (Virtual Proxy / Lazy Initialization):** Erőforrás-igényes objektumok létrehozását késlelteti addig a pillanatig, amíg ténylegesen szükség nem lesz rájuk (pl. nagy felbontású képek letöltése csak akkor, ha a felhasználó odagörget).

- **Védelmi Proxy (Protection Proxy):** Jogosultság-ellenőrzést végez, megvizsgálja, hogy a kliensnek van-e joga meghívni a valódi objektum metódusát.

- **Távoli Proxy (Remote Proxy):** Egy másik címtartományban (pl. egy távoli szerveren) lévő objektum helyi, lokális képviselője (pl. hálózati kommunikáció, RMI, WCF elrejtése a kliens elől).

## 2. A minta működésének bemutatása (Virtuális Proxy példán keresztül)

Képzeljünk el egy dokumentumnézegetőt, amelyben nagyon sok, nagy méretű kép van. Ha a dokumentum megnyitásakor az összes képet letöltenénk és memóriába töltenénk, a program nagyon lassú lenne.

Ehelyett a dokumentumba csak "Kép Proxy" objektumokat teszünk. Ezek a Proxyk gyorsan létrejönnek, tudják a kép nevét/méreteit, így a dokumentum azonnal megjelenik egy üres kerettel ("Betöltés..."). Amikor a felhasználó a képhez görget, és a rendszer meghívja a Proxy Kirajzol() metódusát, a Proxy a háttérben – ekkor először és utoljára – példányosítja a valódi kép objektumot, letölti a fájlt, és továbbítja neki a kirajzolás feladatát.

## 3. Az osztálydiagram

Az osztálydiagram szöveges/karakteres formában:

```
  +------------+          +-----------------------+
  |   Client   | -------> | <<interface/abstract>>|
  +------------+          |        Subject        |
                          +-----------------------+
                          | + Request()           |
                          +-----------------------+
                                     ^
                                     | megvalósítja / öröklődik
                  +------------------+------------------+
                  |                                     |
      +-----------------------+              +-----------------------+
      |        Proxy          |              |      RealSubject      |
      +-----------------------+              +-----------------------+
      | - realSubject         | -----------> |                       |
      +-----------------------+   tartal-    +-----------------------+
      | + Request()           |    mazza     | + Request()           |
      +-----------------------+              +-----------------------+
```



## 4. A mintában szereplő osztályok szerepe

**Subject (Tárgy/Alany):** Egy közös interfész (vagy absztrakt bázisosztály) a RealSubject és a Proxy számára. Mivel mindketten ezt valósítják meg, a kliens bárhol használhatja a Proxyt, ahol a valódi objektumot várja.

**RealSubject (Valódi Tárgy):** A tényleges, nehézsúlyú objektum, amely a valódi üzleti logikát vagy a drága/védett műveleteket elvégzi.

**Proxy (Helyettes):** Az osztály, ami a Client felé a RealSubject-et képviseli. Tartalmaz egy referenciát (realSubject) a valódi objektumra. Elvégzi a kiegészítő feladatokat (létrehozás, jogosultság-vizsgálat stb.), mielőtt/miután továbbítja a kérést a valódi objektumnak.

**Client (Kliens):** Csak a Subject interfészt ismeri, és anélkül használja azt, hogy tudná, vajon egy Proxyval vagy a valódi objektummal áll-e kapcsolatban.

## 5. A minta működése és kritikus metódusok pszeudokódja (Virtuális Proxy esetében)

**Működés:** Amikor a kliens hívja a Request() metódust, a hívást a Proxy kapja el. Megvizsgálja, hogy a RealSubject létezik-e már. Ha nem, akkor létrehozza. Ezután meghívja a RealSubject Request() metódusát (delegálja a hívást).

**Pszeudokód a kritikus működésre:**

```csharp
// A közös interfész, amit a kliens is használ
public interface ISubject {
    void Request();
}

// A valódi, erőforrás-igényes objektum
public class RealSubject : ISubject {
    public RealSubject() {
        // Nagyon lassú művelet, pl. fájl beolvasása hálózatról, adatbázis kapcsolat felépítése
        Console.WriteLine("Valódi objektum inicializálása (sokáig tart)...");
    }

    public void Request() {
        Console.WriteLine("Valódi objektum elvégzi a hasznos munkát.");
    }
}

// A Helyettes osztály
public class Proxy : ISubject {
    // Referencia a valódi objektumra. Kezdetben null!
    private RealSubject realSubject = null;

    // A kritikus metódus, ami szabályozza a hozzáférést
    public void Request() {
        // Virtuális Proxy logika: Lazy initialization (késleltetett példányosítás)
        if (this.realSubject == null) {
            // Csak akkor hozzuk létre, ha tényleg hívják a metódusát
            this.realSubject = new RealSubject();
        }
        
        // Előzetes logikák is kerülhetnének ide (pl. jogosultság ellenőrzés)
        
        // A feladat delegálása a valódi objektumnak
        this.realSubject.Request();
        
        // Utólagos logikák is kerülhetnének ide (pl. naplózás)
    }
}
```