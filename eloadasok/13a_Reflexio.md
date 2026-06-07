12 előadás
Ez az összefoglaló a reflexió (reflection) fogalmát és gyakorlati alkalmazását mutatja be a mellékelt előadásanyag alapján, felkészítve téged a vizsgára. 
1. Mi az a reflexió?
A reflexió egy olyan technika, amely lehetővé teszi a program számára, hogy futás közben (runtime) kérdezzen le információkat saját szerelvényeiről (assembly) és típusairól (osztályok, interfészek stb.). 
A reflexió főbb képességei:
•	Metaadatok lekérdezése: Megtudhatjuk egy szerelvény típusait, illetve egy típus tagjait (tagváltozók, metódusok, eventek). 
•	Dinamikus példányosítás: Létrehozhatunk objektumokat úgy, hogy a típusuk fordításkor még nem ismert (sztringként adjuk meg). 
•	Dinamikus elérés: Futás közben olvashatjuk vagy módosíthatjuk a tagváltozók értékét, és meghívhatjuk a metódusokat. 
•	Attribútumok kezelése: Lekérdezhetjük a nyelvi elemekhez tartozó attribútumokat, vagy akár sajátokat is definiálhatunk. 
2. A reflexió osztályhierarchiája
A reflexió az System.Reflection névtérben található. Az osztályok egy közös őstől, az Object-től származnak. 
•	MemberInfo: Az általános ős a tagokhoz. 
•	Type: Egy típust reprezentál (a legfontosabb osztály). 
•	MethodBase: Metódusok és konstruktorok közös őse. 
•	Konkrét típusok: PropertyInfo (tulajdonságok), FieldInfo (mezők), MethodInfo (metódusok), EventInfo (események), ConstructorInfo (konstruktorok). 
3. Gyakorlati kódpéldák
A) Típus lekérdezése
Két módja van: példányról (GetType) vagy típusnév alapján (typeof). 
```csharp
Complex c1 = new Complex(10, 10);
// 1. Objektum példányról:
Type t1 = c1.GetType(); 
Console.WriteLine(t1.FullName); // Kiírja a névtérrel együtt

// 2. Típusnév alapján (C# operátor):
Type t2 = typeof(Complex);
```

## B) Szerelvény (Assembly) vizsgálata
A reflexió segítségével egy külső `.dll` vagy `.exe` fájl tartalmát futás közben is listázhatjuk, még akkor is, ha a fordításkor nem állt rendelkezésünkre a forráskód.

### Kódpélda:
```csharp
// Betöltjük a külső szerelvényt
Assembly assembly = Assembly.LoadFrom("MyLib.dll");

// Megkapjuk az összes típust, ami a szerelvényben található
Type[] types = assembly.GetTypes(); 

// Kilistázzuk a típusok teljes nevét
foreach (Type type in types) {
    Console.WriteLine(type.FullName);
}
```

## C) Osztály tagjainak listázása
A `Type` osztály metódusai segítségével lekérdezhetjük egy osztály mezőit, metódusait, tulajdonságait stb. Ezek mind a `MemberInfo` osztály leszármazottai.

### Kódpélda: Mezők (Fields) listázása
```csharp
Type type = typeof(Complex);

// Mezők lekérdezése (FieldInfo objektumokon keresztül)
foreach (FieldInfo fi in type.GetFields()) {
    // Kiírja a mező nevét
    Console.WriteLine(fi.Name); 
}
```

## D) Objektum dinamikus példányosítása és hívása
Ez a reflexió legrugalmasabb képessége, amely lehetővé teszi, hogy fordítási időben nem ismert típusokat példányosítsunk és módosítsunk.

* **Megjegyzés:** Bár ez a leghatékonyabb technika moduláris rendszerekhez, egyben a leglassabb is a hagyományos kódhoz képest, ezért csak akkor használd, ha a dinamizmus elengedhetetlen.

### Kódpélda:
```csharp
// 1. Típus lekérése név alapján (névtérrel együtt)
Type complexType = Type.GetType("MiscDemos.Complex"); 

// 2. Konstruktor keresése (paraméterek nélküli)
ConstructorInfo ci = complexType.GetConstructor(new Type[0]); 

// 3. Példányosítás (létrehozzuk az objektumot)
Object instance = ci.Invoke(null); 

// 4. Mező elérése és értékének beállítása
FieldInfo fi = complexType.GetField("Re");
fi.SetValue(instance, 10);
```

## 4. Attribútumok használata (Speciális témakör)
Az attribútumokkal "deklaratív" módon rendelhetünk kiegészítő metaadatokat a kódunkhoz (osztályokhoz, metódusokhoz, mezőkhöz), amelyeket a reflexió segítségével futás közben lekérdezhetünk.

### Saját attribútum definiálása:
1.  **Öröklés:** Leszármazunk a `System.Attribute` osztályból.
2.  **Konfiguráció:** Az `AttributeUsage` attribútummal szabályozzuk, hogy az új attribútumunkat mire alkalmazhatjuk (pl. osztályokra, metódusokra).



### Kódpélda:
```csharp
// Megadjuk, hogy ez az attribútum csak osztályokra tehető
[AttributeUsage(AttributeTargets.Class)]
class StorableClassAttribute : System.Attribute 
{ 
    // Itt definiálhatunk konstruktorokat is, ha adatot akarunk tárolni
    public string Description { get; set; }
}

// Alkalmazás:
[StorableClass(Description = "Adatbázisba menthető entitás")]
class Termek { ... }
```

## Attribútum lekérdezése (példa mentésnél)

A `GetCustomAttributes` metódussal kérdezhetjük le, hogy egy osztályon vagy mezőn rajta van-e az adott attribútum.

* **Elv:** Ha az objektum osztályán nincs `StorableClassAttribute`, nem mentünk semmit.
* **Iteráció:** Végigmegyünk a mezőkön (`GetFields`), és ha egy mezőn rajta van a `StorableAttribute`, elmentjük az értékét.

---

## Összegzés a vizsgához

* **Teljesítmény:** A reflexió lassú, ezért csak akkor használd, ha a dinamikusság elengedhetetlen (pl. keretrendszerek, plugin-rendszerek, automatikus szerializáció).
* **Névtér:** Mindig `System.Reflection`.
* **Metaadatok:** A metaadatok (név, típus, elérés) futási időben érhetők el.
* **Attribútumok:** Az attribútumok statikus "címkék", amiket reflexióval kérdezünk le.