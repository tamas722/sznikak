# Szoftvertechnológia vizsgasor

## 1. Igazak vagy hamisak az alábbi állítások? A sorok elején I és H betűkkel jelölje az igaz és hamis válaszokat (jó válasz: 1p, üres: 0p, rossz: -1p)!

- [I] Egy bool változónak történő értékadás .NET környezetben atomi.
    - **Magyarázat:** A .NET specifikációja alapján a legfeljebb 32 bites egyszerű típusok és a referenciák írása/olvasása atomi.
- [I] A Mutex osztály lehetővé teszi a kölcsönös kizárás megvalósítását eltérő folyamatok szálai között is.
    - **Magyarázat:** Nevén is nevezhetjük (named mutex), így az operációs rendszer szintjén, folyamatok között is használható szinkronizációra.
- [I] A C# kódból a fordítás során - amikor a szerelvény létrejön - köztes kód keletkezik, vagyis a szerelvények köztes (IL) kódot tartalmaznak.
    - **Magyarázat:** A fordító (csc) CIL / MSIL kódra fordít, amit futásidőben a JIT fordít natív gépi kóddá.
- [I] A .NET reflection segítségével lekérdezhető, hogy egy .NET szerelvényben (pl. dll-ben) milyen típusok vannak.
    - **Magyarázat:** A reflexió lehetővé teszi a metaadatok lekérdezését, beleértve a szerelvények tartalmának vizsgálatát is.
- [H] A WinUI esetében az egyirányú (OneWay) adatkötés lehetővé teszi, hogy amikor a vezérlő tulajdonsága megváltozik, az adatköttött forrás tulajdonsága automatikusan megváltozzon.
    - **Magyarázat:** Az egyirányú (OneWay) kötésnél az adatforrás (Source) frissíti a vezérlőt (Target), és nem fordítva. A vezérlő változásának forrásra történő automatikus visszavezetése a kétirányú (TwoWay) kötés jellemzője.

## 2. Adja meg, mely beépített .NET típussal lehet void visszatérésű, két int paraméterű függvényekre hivatkozni. (2p)
- **Válasz:** `Action<int, int>`

## 3. Mit kell írni az alábbiakban pontozott részre C# nyelven, ha azt szeretné, hogy a blokkból való kilépés során az s1 változóra garantáltan hívódjon meg a Dispose művelet? A választ a kipontozott helyre írja be!
.... (StreamReader s1 = new StreamReader(@"c:\temp.txt"))
{ utasítások }
- **Válasz:** `using`
    - **Magyarázat:** A using utasítás garantálja, hogy a blokk végén – akár sikeres lefutás, akár kivétel esetén – automatikusan meghívódik a hivatkozott objektum (jelen esetben az s1) Dispose metódusa. A felszabadításhoz az IDisposable interfészt valósítjuk meg, ami kötelezővé teszi a Dispose metódus írását. Ezt legkényelmesebben és determinisztikusan a using blokk hívja meg. A destruktor (Finalizer) biztonsági hálóként szolgál arra az esetre, ha a programozó elfelejtené a kézi felszabadítást.

## 4. Mely interfészt kell .NET környezetben egy osztályban megvalósítani, amennyiben az osztály nem felügyelt erőforrásra hivatkozik, és szeretnénk azt determinisztikusan felszabadítani?
- **Válasz:** `IDisposable`
    - **Magyarázat:** Ezen interfész megvalósítása kötelező ahhoz, hogy a Dispose() metóduson keresztül determinisztikusan felszabadíthassuk a nem felügyelt erőforrásokat.

## 5. .NET környezetben a célunk az, hogy amikor egy szál beolvas egy adatot, akkor úgy tudjon jelezni más erre várakozó szálaknak, hogy azok közül egy futhasson tovább. Adja meg egy szóval, mit használna ehhez! (2p)
- **Válasz:** `AutoResetEvent`
    - **Magyarázat:** Szemben a ManualResetEvent-tel, az AutoResetEvent jelzéskor (Set hívásakor) a várakozók közül pontosan egy szálat enged tovább, majd azonnal automatikusan visszazár (Reset állapotba kerül).

## 6. ADO.NET alapú adatkezelés. Adjon 1-2 szavas választ az alábbi kérdésekre: (1p)
- Milyen osztályt kell használni egy Oracle adatbázishoz való kapcsolódáshoz?
    - **Válasz:** `OracleConnection`
- Mit kell használni az SQL injection elkerülésére (egy-két szóban válaszoljon)?
    - **Válasz:** Paraméterezett lekérdezés (vagy `SqlParameter`)

## 7. Egy alkalmazásban egy osztályt működését minél egyszerűbben kiterjeszthetővé (könnyen módosíthatóvá) szeretnénk tenni három különböző szempont szerint. Melyik a leginkább illeszkedő klasszikus (Gang of Four) tervezési minta ehhez? (2p)
- **Válasz:** `Strategy` (Stratégia) tervezési minta.
    - **Magyarázat:** Ha egy osztály viselkedését több (pl. három) független szempont szerint is cserélhetővé szeretnénk tenni, akkor öröklődés helyett kompozíciót használunk, és a 3 különböző szempontért felelős logikát 3 külön interfészen (stratégián) keresztül adjuk át az osztálynak.

## 8. Igazak vagy hamisak az alábbi állítások? A sorok elején I és H betűkkel jelölje az igaz és hamis válaszokat (jó válasz: 2p, üres: 0p, rossz: -1p)! (6p)
- [H] A Singleton tervezési mintában az osztály egyetlen példánya az osztály egy virtuális tagfüggvényével kérhető le.
    - **Magyarázat:** A Singleton mintában az egyetlen példányt nem virtuális, hanem egy statikus tagfüggvénnyel (vagy statikus property-vel) kérdezzük le.
- [I] Az Observer mintában az Observernek van egy hivatkozása a Subject-re.
    - **Magyarázat:** Az Observer (Megfigyelő) mintában az Observernek van egy hivatkozása a Subject-re (Megfigyeltre), mert az értesítés (Update) után jellemzően le kell kérdeznie a Subject megváltozott állapotát.
- [H] A Dependency Injection tervezési minta lényege, hogy az osztály a függőségeit a konstruktorban hozza létre (a new operátorral példányosítva azokat), majd egy interfész hivatkozásként tagváltozókban tárolja.
    - **Magyarázat:** A Dependency Injection (Függőséginjektálás) lényege pontosan az, hogy az osztály a függőségeit NEM maga hozza létre a new operátorral, hanem kívülről, kész példányokként (többnyire interfészként) kapja meg a konstruktorban.

## 9. Igazak vagy hamisak az alábbi állítások? A sorok elején I és H betűkkel jelölje az igaz és hamis válaszokat (jó válasz: 1p, üres: 0p, rossz: -1p)! (2p)
- [H] Az architektúra tervezés során az ajánlásokat követve a felhasználói felület (UI) és a logika különválasztásának elve általában arról szól, hogy a felhasználói felületnek függetlennek kell lennie a logikától.
    - **Magyarázat:** A hagyományos architektúra-tervezési elvek szerint a logikának kell függetlennek lennie a felhasználói felülettől, nem pedig fordítva. (A UI réteg ismeri és használja a logikát, de a logika nem tudhat a UI-ról).
- [I] A Pipes and Filters (vagyis Csővezeték) architektúrában, amikor aktív szűrő által vezérelt megoldásról beszélünk, egy aktív szűrő általános implementációjában van egy ciklus is.
    - **Magyarázat:** A Pipes and Filters architektúrában egy aktív szűrő (amely saját szálon fut) implementációjában általában van egy végtelen (vagy feltételes) ciklus (pl. while (true)), amely folyamatosan olvassa a bemeneti csővezetéket és dolgozza fel az adatokat.

## 10. Adjon választ az alábbi kérdésekre! A megoldás során a .NET és C# nyelv korszerű aszinkron programozási eszköztárát használja! (1p)
- a) Adja meg egy olyan aszinkron függvény fejlécét (deklarációját) egy sorban, mely megszámolja és visszaadja, hogy a paraméterben kapott útvonalon elérhető fájlban hány szó van.
    - **Válasz:** `async Task<int> CountWordsAsync(string path)`
- b) Mutasson egysoros példát az előző pontban adott függvény hívására, melyben kiírja a kapott eredményt a konzolra (azt feltesszük, hogy a hívás egy aszinkron függvény belsejében történik).
    - **Válasz:** `Console.WriteLine(await CountWordsAsync(eleresiUt));`
- c) Mely kulcsszót kell azon függvény fejlécének elejére kiírni, melyben az előző pontban megadott függvényhívás történik?
    - **Válasz:** `async`

## 11. Reflection
- a) Melyik C# kulcsszóval lehet egy adott .NET típushoz egy Type objektumot szerezni?
    - **Válasz:** `typeof` (pl. `typeof(MyClass)`)
- b) Adja meg, hogy a Type típus mely műveletének segítségével lehet egy adott típus tagváltozóinak leírását lekérdezni, és milyen típussal tér vissza ez a művelet.
    - **Válasz:** A `GetFields()` művelet segítségével, ami `FieldInfo[]` (FieldInfo tömb) típussal tér vissza.