# 11. Előadás: Szoftverarchitektúra alapok

## 1. Mi az architektúra?
Az architektúra a szoftverrendszer átfogó, magasszintű szervezése és strukturálása.
* **Lényege:** A rendszer dekomponálása alapvető strukturális elemekre, azok kapcsolatának és együttműködésének meghatározása.
* **Tervezés időzítése:** A projekt kezdeti fázisában történik. Utólagos módosítása rendkívül nehéz és költséges.
* **Ad-hoc tervezés következménye:** A rendszer nehezen érthető, bővíthető, karbantartható, és hiányozni fognak belőle az újrafelhasználható minták.

---

## 2. Az architektúra nézetei (Views)
A rendszert több szempontból is vizsgáljuk:
* **Use case nézet:** Funkcionális követelmények.
* **Tervezési/Logikai nézet:** Főbb csomagok, alrendszerek, osztályok.
* **Implementációs nézet:** Komponensek (pl. .dll, .exe), forráskód szervezése.
* **Processz nézet:** Konkurencia, szálak, skálázhatóság, teljesítmény.
* **Telepítési nézet:** Melyik komponens melyik fizikai csomópontra kerül.



---

## 3. Alapelv: Separation of Concerns (SoC)
Az egyik legfontosabb tervezési elv: a kódrészeket aszerint különítjük el, hogy milyen felelősségi körbe tartoznak.
* **Cél:** * **High cohesion:** Nagy összetartozás (az elemek egy dologra fókuszálnak).
    * **Loose coupling:** Laza csatolás (minimális függőség az elemek között).

### Kódpélda: UI és Logika különválasztása

**Rossz gyakorlat (logika a UI-ban):**
```csharp
void Button_Click(object sender, EventArgs e) {
    int nettó = int.Parse(textBox.Text);
    double bruttó = nettó * 1.27; // Logika a UI-ban!
    label.Text = bruttó.ToString();
}

Helyes gyakorlat:
C#
// HELYES: Logika független a UI-tól
class Számológép {
    public double SzámolBruttó(int nettó) => nettó * 1.27;
}

// UI csak használja a logikát
void Button_Click(object sender, EventArgs e) {
    var calc = new Számológép();
    label.Text = calc.SzámolBruttó(int.Parse(textBox.Text)).ToString();
}
```

### Separation of Concerns előnye
A logika unit tesztelhető a UI nélkül is, könnyebb cserélni a felületet (pl. WinUI-ról webes felületre), és a kód karbantarthatósága jelentősen javul.

---

## 4. Alapvető architekturális minták

* **A. MVC (Model-View-Controller):** Webalkalmazásoknál elterjedt (pl. ASP.NET Core).
    * *Model:* Adatok, üzleti logika.
    * *View:* Megjelenítés.
    * *Controller:* Interakciók fogadása, Model és View összekapcsolása.
* **B. Document-View:** Desktop alkalmazásoknál gyakori. Több nézetet is támogathat ugyanahhoz az adathoz (az Observer mintával tartják őket szinkronban).
* **C. Pipes and Filters:** Adatfolyam-feldolgozó rendszerekhez. A szűrők végzik a transzformációt, a csővezetékek pedig továbbítják az adatot.
* **D. Rétegelés (Layers):** A legfontosabb szervezési elv. A függőségi irány egyirányú (felülről lefelé).
    * *UI:* Megjelenítés.
    * *BLL (Business Logic Layer):* Üzleti szabályok, folyamatok.
    * *DAL (Data Access Layer):* Adatbázis műveletek.



---

## 5. Összegző táblázat: Mikor mit használjunk?

| Minta | Jellemző környezet | Fő előny |
| :--- | :--- | :--- |
| **MVC** | Webalkalmazások | Logika és UI szigorú szétválasztása |
| **Document-View** | Desktop alkalmazások | Többnézetű adatkezelés konzisztenciája |
| **Pipes & Filters** | Adatfeldolgozó pipeline-ok | Újrafelhasználható, kombinálható lépések |
| **Rétegelés** | Általános üzleti rendszerek | Komplexitás kezelése, cserélhetőség |