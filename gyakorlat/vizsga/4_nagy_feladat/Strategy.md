# Tervezési minták (20p)

Jellemezze röviden a "Strategy" tervezési mintát! Mire ad megoldást a "Strategy" tervezési minta? Mutassa be általánosságban vagy a minta egy alkalmazásán keresztül (elég az egyik) a minta működését! Ezen belül rajzolja fel a minta osztálydiagramját, valamint röviden adja meg a mintában szereplő osztályok szerepét! Az osztálydiagramon a minta működésének szempontjából kritikus metódusok esetében pszeudokódot is adjon meg!

## 1. Mire ad megoldást a "Strategy" (Stratégia) tervezési minta?

A **Strategy** egy **viselkedési (behavioral)** tervezési minta, amelynek fő célja, hogy **algoritmusok egy családját definiálja**, azokat külön osztályokba zárva (**encapsulate**) egymással kicserélhetővé tegye.

Megoldást nyújt arra a problémára, amikor egy osztályban (vagy metódusban) rengeteg feltételes elágazás (`if-else` vagy `switch-case`) halmozódik fel csak azért, hogy egy feladatot különböző módokon hajtsunk végre. A Strategy minta alkalmazásával az algoritmus a használó klienstől (a kontextustól) függetlenül változhat, így a kód könnyen bővíthetővé válik új algoritmusokkal anélkül, hogy a meglévő kódot módosítani kellene (ezzel támogatva a Nyílt/Zárt – Open/Closed elvet).

## 2. A minta működésének bemutatása (Útvonaltervező példán keresztül)

Képzeljünk el egy térképes navigációs alkalmazást. A felhasználó megad egy kezdőpontot és egy célállomást, az alkalmazás pedig útvonalat tervez. Az útvonaltervezés módja azonban teljesen más, ha autóval, gyalog, biciklivel vagy tömegközlekedéssel megyünk.

**Strategy minta nélkül** az Útvonaltervező osztály egy óriási `TervezzUtvonalat()` metódussal rendelkezne, amiben rengeteg `if (Gyalogos) ... else if (Autos) ...` logika lenne.

**A minta alkalmazásával** létrehozunk egy közös `IUtvonalTervezoStrategia` interfészt. Ezt megvalósítjuk külön osztályokban: `AutosStrategia`, `GyalogosStrategia`, stb. A navigációs alkalmazás (a **Kontextus**) csak letárol egy hivatkozást az aktuálisan kiválasztott stratégiára, és amikor tervezni kell, csupán rászól a stratégiára: "Tervezd meg az utat!". Ha a felhasználó a felületen közlekedési eszközt vált, a futó program egyszerűen kicseréli a háttérben lévő stratégia-objektumot egy másikra.

## 3. Az osztálydiagram

```
  +-----------------------+          +-----------------------+
  |        Context        |          | <<interface/abstract>>|
  +-----------------------+          |       Strategy        |
  | - strategy: Strategy  | <>-----> +-----------------------+
  +-----------------------+          | + ExecuteAlgorithm()  |
  | + SetStrategy(strat)  |          +-----------------------+
  | + DoSomeWork()        |                      ^
  +-----------------------+                      | megvalósítja
                                                 |
                     +---------------------------+---------------------------+
                     |                           |                           |
         +-----------------------+   +-----------------------+   +-----------------------+
         |   ConcreteStrategyA   |   |   ConcreteStrategyB   |   |   ConcreteStrategyC   |
         +-----------------------+   +-----------------------+   +-----------------------+
         | + ExecuteAlgorithm()  |   | + ExecuteAlgorithm()  |   | + ExecuteAlgorithm()  |
         +-----------------------+   +-----------------------+   +-----------------------+
```

## 4. A mintában szereplő osztályok szerepe

- **Context (Környezet / Kontextus):** Fenntart egy hivatkozást az aktuális Strategy objektumra. Nem ismeri a konkrét algoritmus implementációját, csak a Strategy interfészen keresztül kommunikál vele. Általában biztosít egy metódust (pl. `SetStrategy`), amivel a stratégia futási időben lecserélhető.

- **Strategy (Stratégia):** Egy közös interfész (vagy absztrakt bázisosztály) az összes támogatott algoritmus számára. A Context ezt használja a konkrét algoritmusok meghívására.

- **ConcreteStrategy (Konkrét Stratégia):** A Strategy interfészt megvalósító konkrét osztályok (pl. `ConcreteStrategyA`, `ConcreteStrategyB`). Mindegyik egy-egy specifikus algoritmust vagy viselkedést zár magába (pl. autós tervezés, gyalogos tervezés).

## 5. A minta működése és kritikus metódusok pszeudokódja

**Működés:** A Context osztály kritikus metódusa a `DoSomeWork()` (vagy a példában `Tervezz()`), amely nem maga végzi el a számítást, hanem meghívja a beállított stratégia objektum `ExecuteAlgorithm()` metódusát. A másik fontos elem a beállító metódus, amely lehetővé teszi a stratégia dinamikus cseréjét.

**Pszeudokód a kritikus működésre:**

```csharp
// 1. A közös Stratégia interfész
public interface IRouteStrategy {
    void BuildRoute(string start, string end);
}

// 2. Konkrét stratégiák (Algoritmusok)
public class DriveStrategy : IRouteStrategy {
    public void BuildRoute(string start, string end) {
        Console.WriteLine($"Autós útvonal tervezése {start} és {end} között (Autópályák figyelembe vétele)...");
    }
}

public class WalkStrategy : IRouteStrategy {
    public void BuildRoute(string start, string end) {
        Console.WriteLine($"Gyalogos útvonal tervezése {start} és {end} között (Járdák, parkok figyelembe vétele)...");
    }
}

// 3. A Kontextus (A kliens által használt fő osztály)
public class NavigatorContext {
    // Referencia az aktuális stratégiára
    private IRouteStrategy _strategy;

    // Stratégia beállítása (akár konstruktorban, akár futási időben)
    public void SetStrategy(IRouteStrategy strategy) {
        _strategy = strategy;
        Console.WriteLine("Navigátor: Stratégia lecserélve.");
    }

    // A KRITIKUS METÓDUS: A munka delegálása a stratégiának
    public void PlanRoute(string start, string end) {
        if (_strategy == null) {
            Console.WriteLine("Navigátor: Nincs kiválasztva stratégia!");
            return;
        }
        
        Console.WriteLine("Navigátor: Tervezés indítása...");
        
        // Delegálás az interfészen keresztül
        _strategy.BuildRoute(start, end);
    }
}
```