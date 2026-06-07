# Szoftvertechnológia vizsgasor

## 1. Igazak vagy hamisak az alábbi állítások? A sorok elején I és H betűkkel jelölje az igaz és hamis válaszokat (jó válasz: 1p, üres: 0p, rossz: -1p)!
- [I] A kétrétegű architektúra egyik rétege az üzleti logikai réteg (Business Logic Layer).
    - **Válasz/Magyarázat:** I. A kétrétegű, azaz kliens-szerver architektúrában is jelen van az üzleti logika, amely tipikusan vagy a vastagkliensbe integrálva, vagy az adatbázisban, tárolt eljárások formájában fut.
- [I] A Pipes and Filters architektúra lehetővé teszi a filterek többféle kombinálását.
    - **Válasz/Magyarázat:** I. Ez az architektúra fő erőssége: az egymástól független, önálló feladatot ellátó szűrők sorrendje és kapcsolódása rugalmasan változtatható.

## 2. Adja meg, mely beépített .NET típussal lehet olyan függvényekre hivatkozni, melyeknek egy bool és egy double paramétere van (ebben a sorrendben), és void a visszatérésük.
- **Válasz:** `Action<bool, double>`
    - **Magyarázat:** A void visszatérési értékű metódusokhoz a beépített Action delegáltat használjuk, a típusparaméterek pedig a bemenő paraméterek típusait és sorrendjét határozzák meg.

## 3. Mit kell írni az alábbiakban pontozott részre C# nyelven, ha azt szeretné, hogy a blokkból való kilépés során az s1 változóra garantáltan hívódjon meg a Dispose művelet? A választ a kipontozott helyre írja be!
.... (StreamReader s1 = new StreamReader(@"c:\temp.txt"))
{ utasítások }
- **Válasz:** `using`
    - **Magyarázat:** A using utasítás garantálja, hogy a blokk végén – akár sikeres lefutás, akár kivétel esetén – automatikusan meghívódik a hivatkozott objektum (jelen esetben az s1) Dispose metódusa.

## 4. Mely interfészt kell .NET környezetben egy osztályban megvalósítani, amennyiben az osztály nem felügyelt erőforrásra hivatkozik, és szeretnénk azt determinisztikusan felszabadítani?
- **Válasz:** `IDisposable`
    - **Magyarázat:** Ezen interfész megvalósítása kötelező ahhoz, hogy a Dispose() metóduson keresztül determinisztikusan felszabadíthassuk a nem felügyelt erőforrásokat.

## 5. .NET környezetben a célunk az, hogy amikor egy szál beolvas egy adatot, akkor úgy tudjon jelezni más erre várakozó szálaknak, hogy azok közül egy futhasson tovább. Adja meg egy szóval, mit használna ehhez!
- **Válasz:** `AutoResetEvent`
    - **Magyarázat:** Szemben a ManualResetEvent-tel, az AutoResetEvent jelzéskor (Set hívásakor) a várakozók közül pontosan egy szálat enged tovább, majd automatikusan azonnal visszazár (Reset állapotba kerül).

## 6. ADO.NET alapú adatkezelés. Milyen problémához vezethet, ha felhasználói bemenetet (pl. egy termék nevét, melyet a felhasználó egy szövegdobozba begépel) egyszerű string műveletekkel belefűzünk a futtatandó SQL parancsba? Adja meg a probléma nevét két szóban:
- **Válasz:** `SQL injection` (vagy SQL injekció)
    - **Magyarázat:** Ha a felhasználói bemenetet közvetlenül fűzzük be az SQL parancsba paraméterezett lekérdezések (SqlParameter) használata helyett, a támadó módosíthatja az SQL utasítás logikáját kártékony kód befűzésével.

## 7. Igazak vagy hamisak az alábbi állítások? A sorok elején I és H betűkkel jelölje az igaz és hamis válaszokat (jó válasz: 2p, üres: 0p, rossz: -1p)!
- [I] A Document-View architektúrában a dokumentum egy közös gyűjteményben tárolja a különböző típusú nézeteket.
    - **Válasz/Magyarázat:** I. Az Observer minta alapján a dokumentum egy általános interfészen keresztül tartja nyilván a feliratkozott nézeteket.
- [I] Az Adapter tervezési mintában (legalábbis annak object adapter változatában) az Adapter (adaptáló) osztály – amennyiben lehetősége van rá – továbbítja (delegálja) a kéréseket az Adaptee (adaptálandó) osztálynak.
    - **Válasz/Magyarázat:** I. Az objektum szintű illesztő egy belső tagváltozóban tárolja a becsomagolt "Adaptee" objektumot, és továbbhívja felé a kéréseket.
- [I] A Memento mintában az Originator osztálynak van olyan művelete, melynek egy Memento objektumot lehet átadni. Ez a művelet a Mementoban levő adatok alapján az Originator állapotát állítja.
    - **Válasz/Magyarázat:** I. Ez általában a RestoreFromMemento(Memento m) metódus.
- [H] A Singleton tervezési mintában lényeges, hogy a singleton osztálynak legyen egy publikus konstruktora.
    - **Válasz/Magyarázat:** H. Épp ellenkezőleg: a konstruktornak privátnak (vagy védettnek) kell lennie, hogy a kliensek ne tudjanak új példányt létrehozni a new operátorral.

## 8. Igazak vagy hamisak az alábbi állítások? A sorok elején I és H betűkkel jelölje az igaz és hamis válaszokat (jó válasz: 1p, üres: 0p, rossz: -1p)!
- [H] Két lock blokkon belül egyszerre csak egy szál tartózkodhat, amennyiben különböző objektum a két lock blokk paramétere.
    - **Válasz/Magyarázat:** H. A lock a paraméterként átadott objektumhoz kötődik. Ha két különböző objektumra kérünk zárat, azok nem akadályozzák egymást, így mindkét blokkban lehet egy-egy szál.
- [I] Az Interlocked osztály alkalmas arra, hogy egy változó értékét szálbiztos módon eggyel megnöveljük.
    - **Válasz/Magyarázat:** I. Például az Interlocked.Increment(ref valtozo) hívással, ami hardver szintű, atomi műveletet hajt végre.
- [I] C#-ban célszerű destruktort írni egy osztályban, amennyiben az osztály a konstruktorában nem felügyelt erőforrást foglal le és tárol el egy tagváltozójában.
    - **Válasz/Magyarázat:** I. Ha nem felügyelt erőforrásról van szó, a destruktor (Finalizer) biztonsági hálóként szolgál, amely felszabadítja az erőforrást a GC futásakor, amennyiben a programozó elfelejtette meghívni a Dispose()-t.
- [I] A C# kódból a fordítás során - amikor a szerelvény létrejön - köztes kód keletkezik, vagyis a szerelvények köztes (IL) kódot tartalmaznak.
    - **Válasz/Magyarázat:** I. A fordító (csc) CIL / MSIL kódra fordít, amit futásidőben a JIT fordít natív gépi kóddá.
- [H] A ReaderWriterLock megakadályozza, hogy egy erőforrást több szál olvasson egyszerre.
    - **Válasz/Magyarázat:** H. Ennek a zárnak éppen az a célja, hogy párhuzamosan több olvasó szálat (Reader) is beengedjen egyszerre, de író szálból (Writer) egy időben szigorúan csak egy kaphat hozzáférést.

## 9. Adjon választ az alábbi kérdésekre! A megoldás során a .NET és C# nyelv korszerű aszinkron programozási eszköztárát használja!
- a) Adja meg egy olyan aszinkron függvény fejlécét (deklarációját) egy sorban, mely visszaadja a string paraméterben kapott hosszú karaktersorozatot byte tömb formában titkosított adatként. (1p)
    - **Válasz:** `async Task<byte[]> EncryptDataAsync(string input)`
- b) Mutasson egysoros példát az előző pontban adott függvény hívására, melyben a kapott eredményt (a titkosított adatot) beleteszi egy lokális változóba (azt feltesszük, hogy a hívás egy aszinkron függvény belsejében történik). (0,5p)
    - **Válasz:** `byte[] encryptedData = await EncryptDataAsync(myString);`
- c) Mely kulcsszót kell azon függvény fejlécének elejére kiírni, melyben az előző pontban megadott függvényhívás történik? (0,5p)
    - **Válasz:** `async`

## 10. Reflection
- a) Melyik az a beépített .NET típus, mely segítségével le lehet kérdezni, hogy egy adott típusnak milyen tagváltozói és tagfüggvényei vannak?
    - **Válasz:** `Type` osztály (pl. `typeof(OsztalyNev)` vagy `obj.GetType()`)
- b) Adja meg, hogy az előző pontban megadott típus mely műveletének segítségével lehet egy típus tagváltozóit lekérdezni, és milyen típussal tér vissza ez a művelet.
    - **Válasz:** `GetFields()` művelet, amely egy `FieldInfo[]` (FieldInfo tömb) típussal tér vissza.