# 2. WinUI/XAML! (20p)

## a) (10p)

Valósítsa meg az alábbi elrendezésnek és viselkedésnek megfelelő felületet: adja meg a felület XAML leírását! Az XML névtereket nem kell megadnia.

- Egy két oszlopos és két soros elrendezést alakítson ki a jobbra látható ábrának megfelelően.
- A baloldali oszlop pontosan olyan széles legyen, mint amekkora helyet a benne levő szövegek („Title” és „***”) igényelnek. Ebben az oszlopban a szövegek függőleges irányban legyenek felülre igazítva.
- A jobboldali oszlop töltse ki a teljes maradó helyet az ablakban (akkor is, ha a felhasználó átméretezi az ablakot).
- Az első sor 60 pixel magas legyen, a második pedig olyan magas, mint amekkora helyet a benne levő vezérlők igényelnek.
- A második oszlop:
  - **i.** első sorában egy balra igazított, 80 pixel széles, szerkeszthető szövegdoboz található, mely függőleges irányban kitölti a teljes teret.
  - **ii.** második sorában egy alapértelmezett magasságú, „Append” feliratú gomb található, mely vízszintes irányban kitölti a teljes teret.
- A második oszlopban levő vezérlők körül legyen minden irányban **4 pixel** szabad hely a szellősebb kialakítás érdekében.

## b) (6p)

A fenti felületet könyvek címének megjelenítésére és szerkesztésére kívánjuk használni. Egészítse ki a korábbi megoldását a következővel, a megoldásban **NE használja az MVVM mintát**:

- Írjon meg egy megfelelő `Book` osztályt (a könyvnek egyelőre csak címe van), és azt használja a megoldásban.
- A szövegdobozban egy könyv címe jelenjen meg és azt lehessen szerkeszteni (ezt a könyv objektumot a *code behind*ban vegye fel).
- Amikor a felhasználó kattint az „Append” feliratú gombon, a könyv címéhez (annak **elejére**) fűzze hozzá a `"[]"` szöveget, és azt írja vissza a szövegdobozba (pl., ha `"Star Wars"` a könyv címe, akkor `"[]Star Wars"` legyen).
- A szövegdoboz tartalmát a `Book` osztályra épülő **adatkötés** segítségével olvassa ki és írja vissza (vagyis nem férhet hozzá közvetlenül a vezérlő tulajdonságaihoz).
- A megoldás során használhat MVVM Toolkit támogatást, de az nem kötelező! Ha használja, akkor egy rövid mondatban adja meg, minek a megvalósítását segíti az MVVM Toolkit a megoldásban.
- A korábbi XAML megoldását úgy egészítse ki, hogy `*1`, `*2` stb.-vel jelöljön meg helyeket benne, ahova új kódot kell beszúrni, majd külön adja meg ezekre vonatkozóan a beszúrandó kódot. Az újonnan bevezetett XAML részeken túl adja meg a *code behind* fájlban levő osztály kódját is!

## c) (4p)

A feladat a korábbi megoldás „átalakítása” MVVM minta alapúra (nem szükséges a *Strict* modellt használnia):

- Írjon meg egy megfelelő, ablakhoz tartozó `ViewModel` osztályt.
- A viselkedés/működés meg kell egyezzen a korábbival: a gombkattintást **Command minta** alapján valósítsa meg (ehhez használhat MVVM Toolkitöt), vagy eseménykötéssel.
- Parancsok tiltásával/engedélyezésével nem kell foglalkoznia.
- Adja meg, hogy a XAML fájlban milyen változtatásokra van szükség (ehhez használjon `#1`, `#2` stb. hivatkozásokat).
- Adja meg egy mondatban, hogy ennél a megoldásnál **mit tartalmaz** az ablakhoz tartozó *code behind* fájl.


Íme a feladat teljes körű megoldása, az előzőekhez hasonlóan minden kódsor és tulajdonság fölött részletes magyarázattal ellátva, hogy pontosan érthető legyen a működése.

a) XAML elrendezés megvalósítása (10p)
Az alábbi XAML kód megvalósítja a kért kétszer-kettes rácsot és elhelyezi benne a vezérlőket a megfelelő igazításokkal és margókkal.

```xml
<Grid>

    <Grid.ColumnDefinitions>
        
        <ColumnDefinition Width="Auto" />
        
        <ColumnDefinition Width="*" />

    </Grid.ColumnDefinitions>

    <Grid.RowDefinitions>
        
        <RowDefinition Height="60" />
        
        <RowDefinition Height="Auto" />

    </Grid.RowDefinitions>


    <TextBlock Grid.Row="0" 
               Grid.Column="0" 
               Text="Title" 
               VerticalAlignment="Top" />


    <TextBlock Grid.Row="1" 
               Grid.Column="0" 
               Text="***" 
               VerticalAlignment="Top" />


    <TextBox Grid.Row="0" 
             Grid.Column="1" 
             HorizontalAlignment="Left" 
             Width="80" 
             VerticalAlignment="Stretch" 
             Margin="4" 
             *1 />


    <Button Grid.Row="1" 
            Grid.Column="1" 
            Content="Append" 
            HorizontalAlignment="Stretch" 
            Margin="4" 
            *2 />

</Grid>
```

b) Adatkötés (Data Binding) MVVM nélkül (6p)
Az MVVM Toolkit szerepe a megoldásban:
Az MVVM Toolkit az ObservableObject ősosztály és az [ObservableProperty] attribútumok révén automatikusan legenerálja a felületi értesítésekhez szükséges INotifyPropertyChanged kódot, így jelentősen csökkenti a megírandó "boilerplate" kódot.

1. A Book osztály:

```csharp
// A Book osztály, amely az ObservableObject-ből örököl, hogy jelezni tudjon a felületnek a változásokról
public partial class Book : ObservableObject
{
    // Az MVVM Toolkit legenerálja ebből a 'Title' publikus tulajdonságot, ami értesíti a XAML-t, ha módosul
    [ObservableProperty]
    private string title;
}
2. A Code behind osztály (MainWindow.xaml.cs):

C#
// Az ablakhoz tartozó C# háttérkód
public sealed partial class MainWindow : Window
{
    // Publikus tulajdonságként definiáljuk a könyvet, hogy a XAML adatkötéssel (x:Bind) elérhesse
    public Book MyBook { get; set; }

    // Konstruktor
    public MainWindow()
    {
        // A felület kötelező inicializálása
        this.InitializeComponent();
        
        // Példányosítjuk a könyvet egy kezdeti címmel
        MyBook = new Book { Title = "Star Wars" };
    }

    // Az "Append" gomb kattintási eseménykezelője
    private void AppendButton_Click(object sender, RoutedEventArgs e)
    {
        // A könyv címének elejére hozzáfűzzük a "[]" karaktereket.
        // Mivel adatkötést használunk az [ObservableProperty]-vel, a TextBox automatikusan frissül.
        MyBook.Title = "[]" + MyBook.Title;
    }
}
```

3. A XAML fájl kiegészítései (az a) feladat jelölései helyére):

```xml
Text="{x:Bind MyBook.Title, Mode=TwoWay}"

Click="AppendButton_Click"
```

c) Átalakítás MVVM minta alapúra (4p)
Ebben a lépésben az állapotot és a logikát leválasztjuk a felületről, és egy dedikált ViewModel osztályba mozgatjuk Command (parancs) minta használatával.

1. A ViewModel osztály:

```csharp
// A felület logikáját és adatait összefogó ViewModel osztály
public partial class MainWindowViewModel : ObservableObject
{
    // A könyv címe a ViewModel-ben
    [ObservableProperty]
    private string title;

    // A [RelayCommand] attribútum automatikusan generál egy 'AppendCommand' nevű parancsot a gombhoz
    [RelayCommand]
    private void Append()
    {
        // A parancs lefutásakor a logikát végrehajtjuk
        Title = "[]" + Title;
    }
}
```

2. A Code behind feladata egy mondatban:
Ennél az MVVM megoldásnál a code behind fájl eseménykezelőket már nem tartalmaz, egyetlen feladata az ablak felületi inicializálása (InitializeComponent()), valamint a MainWindowViewModel példányosítása és tárolása egy publikus tulajdonságban, hogy a XAML hozzáférjen.

3. A XAML fájl szükséges változtatásai (a b) részhez képest):

```xml
Text="{x:Bind ViewModel.Title, Mode=TwoWay}"

Command="{x:Bind ViewModel.AppendCommand}"
```