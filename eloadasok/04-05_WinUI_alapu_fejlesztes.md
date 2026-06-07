# 4. Előadás: Felhasználói felületek és XAML

## 1. Bevezetés: Microsoft UI technológiák
A Windowsos felhasználói felületek (UI) fejlesztése hosszú utat járt be az évek során:

* **Korai idők:** GDI, WinForms.
* **A "XAML" korszak kezdete:** WPF (2006), UWP (2012), Xamarin Forms. Ezek közös jellemzője, hogy a felületet XAML nyelv segítségével írják le.
* **Modern keretrendszerek:** MAUI (multiplatform), Avalonia, UNO és a **WinUI 3**.
* **WinUI 3:** A Microsoft modern, natív asztali fejlesztési technológiája. C++ és C# nyelven is programozható, az operációs rendszertől független könyvtár, amely a modern "Fluent Design" formanyelvet használja.

---

## 2. A XAML alapjai
A XAML egy deklaratív (leíró), XML-alapú nyelv. Célja, hogy elkülönítse a grafikus felületet a programozási logikától.

* **Példányosítás:** Minden XML tag (pl. `<Button>`) egy C# objektumot (osztályt) hoz létre. Az attribútumok (pl. `Width="50"`) az adott objektum tulajdonságait (`property`) állítják be.
* **Vektoros rajzolás:** A felület vektorgrafikus, tehát felbontástól függetlenül éles marad.
* **Mögöttes kód (Code-behind):** A XAML fájlhoz mindig tartozik egy C# osztály (pl. `.xaml.cs` kiterjesztéssel), ahol a felület eseményeit (pl. kattintás) kezeljük.

### Példa a XAML és a C# kapcsolatára:

**XAML (Felület):**
```xml
<Button x:Name="SajatGomb" Content="Kattints ide" Click="SajatGomb_Click"/>
```

* **Magyarázat: Az x:Name azonosítót ad a gombnak, amivel a C# kódból hivatkozhatunk rá. A Click attribútum megadja, melyik metódus fusson le kattintáskor.

### C# (Code-behind):

```csharp
private void SajatGomb_Click(object sender, RoutedEventArgs e) {
    SajatGomb.Content = "Rákattintottál!"; // A C# kód eléri az x:Name-mel jelölt gombot
}
```

## 3. Komponálhatóság és a legfontosabb vezérlők
A WinUI egyik legnagyobb ereje a **komponálhatóság**: a vezérlők (pl. gombok, listaelemek) bármilyen más vezérlőt tartalmazhatnak. Egy gomb (`Button`) `Content` tulajdonsága lehet sima szöveg, de lehet benne egy `StackPanel`, abban pedig kép és szöveg is.

### Vezérlők (Controls) hierarchiája:
* **Szöveg megjelenítők:** * `TextBlock`: Egyszerű vagy formázott szöveg (nem szerkeszthető).
    * `TextBox`: Szerkeszthető adatbeviteli mező (egysoros vagy többsoros).
* **ContentControl (Egy tartalmat kezelő vezérlők):** Megjelenítenek "valamit" (pl. stringet vagy UI elemet). Ebből származik a `Button`, a `CheckBox`, a `RadioButton` és a `ToggleButton`.
* **ItemsControl (Listázó vezérlők):** Több elemet jelenítenek meg. Ezekből származnak a Selector vezérlők (pl. `ComboBox`, `ListBox`, `ListView`).

---

## 4. Dependency Property (Függőségi Tulajdonság)
Ez egy nagyon fontos ZH téma!

* **Probléma:** Egy átlagos vezérlőnek több mint 100 tulajdonsága van. 1000 vezérlő esetén ez rengeteg memóriát emésztene fel, miközben az értékek 95%-a alapértelmezett.
* **Megoldás:** A vezérlők egy `DependencyObject` ősosztályból származnak, ami egy belső szótárban csak azokat a tulajdonságokat menti el, amik **eltérnek** az alapértelmezettől.
* **Előnyök:** Ez teszi lehetővé az Adatkötést (Data Binding), a stílusok alkalmazását és az animációkat reflexió használata nélkül.
* **Csatolt tulajdonságok (Attached Properties):** A vezérlőn olyan tulajdonság állítható be, ami egy "fölötte" lévő elemhez tartozik (pl. `Canvas.Left="120"` egy `TextBlock`-on).

---

## 5. Elrendezés (Layout panelek)
A panelek segítségével építjük fel a felületet:

### Igazítások és térközök:
* **Margin:** A vezérlő kívülről mért távolsága.
* **Padding:** A vezérlő határa és a belső tartalma közötti távolság.
* **Horizontal/VerticalAlignment:** Lehet `Left/Top`, `Center`, `Right/Bottom` és `Stretch` (alapértelmezett, kitölti a teret).

### Legfontosabb panelek:
* **StackPanel:** Elemek egymás alá (Vertical) vagy mellé (Horizontal) pakolása.
* **Grid:** Táblázatszerű elrendezés (`RowDefinitions`, `ColumnDefinitions`). Méretezés: fix pixel, `Auto`, vagy `*` (arányos). Gyerekeket csatolt tulajdonságokkal helyezzük el: `Grid.Row="1"`, `Grid.Column="0"`.
* **Canvas:** Abszolút pozicionálás (`Canvas.Left`, `Canvas.Top`, `Canvas.ZIndex`).
* **RelativePanel:** Vezérlők egymáshoz képesti igazítása.

**Példa Gridre:**
```xml
<Grid>
    <Grid.RowDefinitions>
        <RowDefinition Height="Auto"/> 
        <RowDefinition Height="*"/>
    </Grid.RowDefinitions>
    <TextBlock Grid.Row="0" Text="Cím"/> 
</Grid>
```

## 6. Erőforrások (Resources) és Stílusok (Styles)

### Erőforrás (Resource)
A XAML-ben példányosíthatunk C# objektumokat (például egy ecsetet, színt vagy egy egyedi osztályt), adunk neki egy `x:Key` azonosítót, majd hivatkozunk rá a `{StaticResource KulcsNeve}` segítségével. Ez biztosítja az újrahasználhatóságot a felületen.

### Stílus (Style)
A stílus egy adott típusú vezérlő (pl. gomb, téglalap) megjelenési tulajdonságait gyűjti össze egységesen. A `<Setter>` tagekkel állítjuk be a konkrét értékeket, amelyeket azután egyszerre alkalmazhatunk több vezérlőre.

**Példa kód:**
```xml
<Grid>
    <Grid.Resources>
        <Style TargetType="Rectangle" x:Key="PirosNegyzet">
            <Setter Property="Fill" Value="Red"/>
            <Setter Property="Width" Value="100"/>
        </Style>
    </Grid.Resources>

    <Rectangle Style="{StaticResource PirosNegyzet}" />
</Grid>
```

## 7. Adatkötés (Data Binding) és MVVM alapok
Az adatkötés köti össze a UI elemeket az üzleti logikával. Ha az adat változik, a UI automatikusan frissül.

### Adatkötés típusai
* **{Binding}:** Régebbi, reflexió alapú, futásidőben dől el, kicsit lassabb. Alapértelmezett módja a `OneWay`.
* **{x:Bind}:** Újabb, fordítási időben generált kód, erősen típusos, ezért gyorsabb. Alapértelmezett módja a `OneTime`.

### Irányok (Mode)
* **OneTime:** Csak a felület betöltésekor veszi át az adatot az adatforrásból.
* **OneWay:** Csak olvas. Ha a C# kód változtatja a változót, a UI frissül.
* **TwoWay:** Kétirányú. Pl. egy beviteli mezőnél a UI frissítheti a C# változót is.

### Frissítési mechanizmus - INotifyPropertyChanged (INPC)
Ahhoz, hogy a felület tudjon egy változó módosulásáról (`OneWay` vagy `TwoWay` esetén), az adatforrásnak jeleznie kell a változást. Ezt az `INotifyPropertyChanged` interfésszel tehetjük meg. A modern fejlesztés során ezt gyakran a `CommunityToolkit.MVVM` könyvtárral használják.

**Példa INPC-vel (MVVM Toolkit):**

```csharp
public partial class Dancer : ObservableObject 
{
    // Generál nekünk egy nagybetűvel kezdődő 'Name' property-t, 
    // ami automatikusan frissíti a felületet!
    [ObservableProperty] 
    string name = "Béla"; 
}
```
### Adatkonverzió (IValueConverter)
Ha az adatforrás és a UI elem típusa eltér (például egy igaz/hamis (boolean) adatból szeretnénk láthatóságot (`Visibility`) szabályozni), egy konverter osztályt kell írnunk. A konvertert erőforrásként fel kell venni a XAML-ben, majd beállítani a kötésnél a `Converter={StaticResource Neve}` attribútummal.

---

## 8. Sablonok (Templates)
A sablonok (főleg a `DataTemplate`) biztosítják az azonos típusú C# objektumok (például lista elemek) egységes vizuális megjelenítését a UI-on. Egy memóriában lévő adathoz (pl. `Dancer` osztály példány) hozzárendelünk egy megjelenési struktúrát (pl. egy `StackPanel`-t, benne képpel és a nevével).



**Példa kód egy listás adatkötésre sablonnal:**

```xml
<Grid.Resources>
    <DataTemplate x:Key="TancosSablon" x:DataType="local:Dancer">
        <StackPanel Orientation="Horizontal">
            <TextBlock Text="{x:Bind Name}" FontWeight="Bold" />
            <TextBlock Text="{x:Bind Role}" Margin="10,0,0,0"/>
        </StackPanel>
    </DataTemplate>
</Grid.Resources>
 
<ListView ItemsSource="{x:Bind Dancers}" ItemTemplate="{StaticResource TancosSablon}"/>
```
*Magyarázat:* Az `ItemsSource` megkap egy C# gyűjteményt (pl. `ObservableCollection<Dancer> Dancers`). A `ListView` automatikusan végigiterál ezen a listán, és minden egyes elemre ráhúzza a `TancosSablon` nevű `DataTemplate`-et, majd lefordítja az adatkötéseket a megfelelő adatokra.

---

### 9. Parancsok (Commands) az MVVM mintában

Amikor MVVM mintát használunk, a UI eseményeit (pl. egy gombkattintást) nem a hagyományos eseménykezelőkkel (`Click="Gomb_Click"`) a Code-behindban kezeljük, hanem úgynevezett **Parancsokat (Commands)** kötünk a vezérlőkhöz. Ezzel az üzleti logika teljesen elválasztható a felülettől.

A WinUI-ban erre az `ICommand` interfészt használjuk, leggyakrabban a `RelayCommand` osztály segítségével.



#### Példa a parancsok használatára:

**1. A C# oldal (ViewModel):**
```csharp
public class MyViewModel {
    // A Command objektum, amit a XAML-ből el fogunk érni
    public ICommand SaveDocCommand { get; }
 
    public MyViewModel() {
        // Konstruktorban példányosítjuk a parancsot, és megadjuk, melyik metódust hívja meg
        SaveDocCommand = new RelayCommand<string>(SaveDoc);
    }
 
    // Ez a metódus fog lefutni, ha rákattintanak a gombra
    private void SaveDoc(string param) {
        // ... Mentési logika ...
        // A 'param' értéke a XAML-ből jön ("MyParam")
    }
}
```
```xml
<Button Content="Mentés" 
        Command="{x:Bind SaveDocCommand}" 
        CommandParameter="MyParam"/>
```

### 10. Behaviors (Viselkedések)

Ha egy vezérlőnek (pl. egy sima `TextBlock`-nak) nincs beépített `Command` tulajdonsága, de mégis szeretnénk, ha kattintásra lefutna egy parancs, akkor úgynevezett **"Behaviors"**-t használunk. 

Ehhez telepíteni kell egy kiegészítő csomagot, majd a XAML-ben az `EventTriggerBehavior` segítségével egy eseményt (pl. `Tapped` - érintés/kattintás) át tudunk alakítani parancshívássá (`InvokeCommandAction`).