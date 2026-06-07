# 2. WinUI/XAML! (20p)

## a) (10p)

Valósítsa meg az alábbi elrendezésnek és viselkedésnek megfelelő felületet: adja meg a felület XAML leírását! Az XML névtereket nem kell megadnia.

- Egy két oszlopos és két soros elrendezést alakítson ki a jobbra látható ábrának megfelelően.
- A baloldali oszlop 150 pixel széles legyen.
- A jobboldali oszlop pontosan olyan széles legyen, mint amekkora helyet a benne levő tartalom (a „Reset” feliratú gomb) igényel.
- A második sor olyan magas legyen, mint amekkora helyet a benne levő jelölőnégyzet igényel, az első sor pedig töltse ki a teljes maradó helyet az ablakban (akkor is, ha a felhasználó átméretezi az ablakot).
- Az első oszlop első sorában egy 80 pixel széles szerkeszthető szövegdoboz található, mely függőleges irányban kitölti a teljes teret, vízszintes irányban pedig középre van igazítva.
- Az első sor második oszlopában egy vízszintesen balra, függőlegesen pedig középre igazított, „Reset” feliratú gomb található.
- Az első oszlop második sorában egy 90 pixel széles, jobbra igazított, „Watched” feliratú jelölőnégyzet található. A jelölőnégyzet körül alul és felül legyen 5 pixel szabad hely.

## b) (6p)

A fenti felületet filmek adatainak megjelenítésére és szerkesztésére kívánjuk használni. Egészítse ki a korábbi megoldását a következővel, a megoldásban **NE használja az MVVM mintát**:

- Írjon meg egy megfelelő `Movie` osztályt, és azt használja a megoldásban. A filmnek van:
  - leírása (`string`)
  - „megtekintett” (`bool`) tulajdonsága (azt határozza meg, hogy az adott film megtekintettnek van-e megjelölve).
- A szövegdobozban egy film leírása jelenjen meg és azt lehessen szerkeszteni.
- A jelölőnégyzet a film „megtekintett” tulajdonságát jelenítse meg, illetve ezt a tulajdonságot lehessen segítségével megváltoztatni.
- A megjelenített/szerkesztett film objektumot a *code behind*ban vegye fel.
- A „Reset” feliratú gombra kattintva a film „megtekintett” tulajdonsága `false` legyen (a jelölőnégyzet állapota is frissüljön ennek megfelelően).
- A szövegdoboz tartalmát és a jelölőnégyzet állapotát a `Movie` osztályra épülő **adatkötés** segítségével olvassa ki és írja vissza (vagyis nem férhet hozzá közvetlenül a vezérlő tulajdonságaihoz).
- A megoldás során használhat MVVM Toolkit támogatást, de az nem kötelező! Ha használja, akkor egy rövid mondatban adja meg, minek a megvalósítását segíti az MVVM Toolkit a megoldásban.
- A korábbi XAML megoldását úgy egészítse ki, hogy `*1`, `*2` stb.-vel jelöljön meg helyeket benne, ahova új kódot kell beszúrni, majd külön adja meg ezekre vonatkozóan a beszúrandó kódot. Az újonnan bevezetett XAML részeken túl adja meg a *code behind* fájlban levő osztály kódját is!

## c) (4p)

A feladat a korábbi megoldás „átalakítása” MVVM minta alapúra (nem szükséges a *Strict* modellt használnia):

- Írjon meg egy megfelelő, ablakhoz tartozó `ViewModel` osztályt.
- A viselkedés/működés meg kell egyezzen a korábbival: a gombkattintást **Command minta** alapján valósítsa meg (ehhez használhat MVVM Toolkit támogatást), vagy eseménykötéssel.
- Parancsok tiltásával/engedélyezésével nem kell foglalkoznia.
- Adja meg, hogy a XAML fájlban milyen változtatásokra van szükség (ehhez használjon `#1`, `#2` stb. hivatkozásokat).
- Adja meg egy mondatban, hogy ennél a megoldásnál **mit tartalmaz** az ablakhoz tartozó *code behind* fájl.

* **Megoldás**:
a) XAML elrendezés megvalósítása (10p)
Az alábbi XAML kód definiálja a rácsszerkezetet és a benne lévő vezérlőket. A *1, *2, *3 jelölések a későbbi feladatok bekötési pontjait mutatják.

```xml
<Grid>
    <Grid.ColumnDefinitions>
        <ColumnDefinition Width="150" />
        <ColumnDefinition Width="Auto" />
    </Grid.ColumnDefinitions>

    <Grid.RowDefinitions>
        <RowDefinition Height="*" />
        <RowDefinition Height="Auto" />
    </Grid.RowDefinitions>

    <TextBox Grid.Row="0" Grid.Column="0" 
             Width="80" 
             VerticalAlignment="Stretch" 
             HorizontalAlignment="Center" 
             *1 /> 

    <Button Grid.Row="0" Grid.Column="1" 
            Content="Reset" 
            HorizontalAlignment="Left" 
            VerticalAlignment="Center" 
            *2 /> 

    <CheckBox Grid.Row="1" Grid.Column="0" 
              Content="Watched" 
              Width="90" 
              HorizontalAlignment="Right" 
              Margin="0,5,0,5" 
              *3 /> 
</Grid>
```

b) Adatkötés (Data Binding) MVVM nélkül (6p)
Ebben a részben az MVVM Toolkitet csak arra használjuk, hogy lerövidítsük a felületértesítési kódokat, de a modellt közvetlenül a code behind-ban kezeljük (nincs ViewModel).

Az MVVM Toolkit szerepe a megoldásban:
Az MVVM Toolkit a megoldásban az INotifyPropertyChanged interfész egyszerűsített megvalósítását segíti az ObservableObject ősosztály és az [ObservableProperty] attribútum révén, így a felület értesül az adatváltozásokról.

1. A Movie osztály:

```csharp
// A Movie osztály az ObservableObject-ből származik le a felületi értesítések (INotifyPropertyChanged) miatt
public partial class Movie : ObservableObject
{
    // A toolkit legenerálja ehhez a mezőhöz a 'Description' publikus tulajdonságot értesítéssel
    [ObservableProperty]
    private string description;

    // A toolkit legenerálja ehhez a mezőhöz a 'Watched' publikus tulajdonságot értesítéssel
    [ObservableProperty]
    private bool watched;
}
2. A Code behind osztály:

C#
// Az ablakhoz tartozó C# háttérkód (Code Behind)
public sealed partial class MainWindow : Window
{
    // Publikus tulajdonságként vesszük fel a filmet, hogy a XAML hivatkozhasson rá a {x:Bind}-dal
    public Movie MyMovie { get; set; }

    // Konstruktor
    public MainWindow()
    {
        // A felületi elemek inicializálása (WinUI-ban kötelező)
        this.InitializeComponent();
        
        // Példányosítjuk a filmet egy kezdő állapottal
        MyMovie = new Movie { Description = "Példa film", Watched = true };
    }

    // A "Reset" gombhoz tartozó kattintási eseménykezelő metódus
    private void ResetButton_Click(object sender, RoutedEventArgs e)
    {
        // A kód közvetlenül a Movie objektum megtekintett tulajdonságát állítja hamisra
        // A felület (a checkbox) az adatkötés miatt automatikusan frissülni fog.
        MyMovie.Watched = false;
    }
}
```
3. A XAML fájl kiegészítései (az a) feladat jelölései helyére):

```xml
Text="{x:Bind MyMovie.Description, Mode=TwoWay}"

Click="ResetButton_Click"

IsChecked="{x:Bind MyMovie.Watched, Mode=TwoWay}"
```

c) Átalakítás MVVM minta alapúra (4p)
Ebben az esetben megszüntetjük az eseménykezelőket a code behind-ban, és egy dedikált ViewModel veszi át az irányítást parancsok (Command) használatával.

1. A ViewModel osztály:

```csharp
// A felülethez tartozó ViewModel osztály, amely tárolja az állapotot és a logikát
public partial class MainWindowViewModel : ObservableObject
{
    // A film leírása tulajdonság (itt most közvetlenül a ViewModel-ben deklarálva)
    [ObservableProperty]
    private string description;

    // A film megtekintett állapota
    [ObservableProperty]
    private bool watched;

    // Ez az attribútum automatikusan generál egy 'ResetCommand' nevű parancsot a gomb számára
    [RelayCommand]
    private void Reset()
    {
        // Végrehajtáskor a megtekintett tulajdonságot hamisra billenti
        Watched = false;
    }
}
```
2. A Code behind feladata egy mondatban:
Ennél az MVVM megoldásnál a code behind fájl kizárólag az ablak inicializálását (az InitializeComponent() hívását) és a MainWindowViewModel egy példányának létrehozását és tárolását tartalmazza (pl. a ViewModel nevű tulajdonságban), a felületi logikát viszont egyáltalán nem.

3. A XAML fájl szükséges változtatásai (a b) részhez képest):

```xml
Text="{x:Bind ViewModel.Description, Mode=TwoWay}"
Command="{x:Bind ViewModel.ResetCommand}"
IsChecked="{x:Bind ViewModel.Watched, Mode=TwoWay}"
```