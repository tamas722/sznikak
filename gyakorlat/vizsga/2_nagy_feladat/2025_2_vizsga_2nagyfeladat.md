# 2. WinUI/XAML! (20p)

## a) (10p)

Valósítsa meg az alábbi elrendezésnek és viselkedésnek megfelelő felületet: adja meg a felület XAML leírását! Az XML névtereket nem kell megadnia.

- Egy két oszlopos és két soros elrendezést alakítson ki a jobbra látható ábrának megfelelően.
- A baloldali oszlop olyan széles legyen, mint amit a benne levő vezérlők („Quantity” címke) igényelnek, a jobboldali oszlop pedig töltse ki a teljes maradó helyet az ablakban (akkor is, ha a felhasználó átméretezi az ablakot).
- Az első sor olyan magas legyen, mint amit a benne levő vezérlők igényelnek, a második sor 80 pixel magas.
- Az első sor első oszlopában egy „Quantity” feliratú, nem szerkeszthető címke található, függőlegesen középre igazítva.
- Az első sor második oszlopában középre igazítva, szorosan egymás mellett egy 30 pixel széles szerkeszthető szövegdoboz és tőle jobbra egy „+” feliratú gomb található (a kettő együtt van vízszintesen középre igazítva).
- A második sor második oszlopában egy 40 pixel magas szerkeszthető szövegdoboz található, mely vízszintes irányban kitölti a teret, függőleges irányban pedig felülre van igazítva. A szövegdoboz körül balról 10, jobbról 10, felülről pedig 15 pixel szabad hely.

## b) (6p)

A fenti felületet egy rendelési tétel adatainak megjelenítésére és szerkesztésére kívánjuk használni. Egészítse ki a korábbi megoldását a következővel, a megoldásban **NE használja az MVVM mintát**:

- Írjon meg egy megfelelő `OrderItem` osztályt, és azt használja a megoldásban. Egy rendelési tételnek van:
  - mennyisége (`int`)
  - „leírás” (`string`) tulajdonsága.
- Az alsó sorban levő szövegdobozban egy rendelési tétel leírása jelenjen meg és azt lehessen szerkeszteni.
- A felső sorban levő szövegdobozban egy rendelési tétel mennyisége jelenjen meg és azt lehessen szerkeszteni.
- A felső sorban levő „+” gombbal a rendelési tétel mennyiségét lehessen eggyel megnövelni.
- A megjelenített/szerkesztett rendelési tétel objektumot a *code behind*ban vegye fel.
- A szövegdobozok tartalmát a rendelési tétel osztályra épülő **adatkötés** segítségével olvassa ki és írja vissza (vagyis nem férhet hozzá közvetlenül a vezérlő tulajdonságaihoz).
- A megoldás során használhat MVVM Toolkit támogatást, de az nem kötelező! Ha használja, akkor egy rövid mondatban adja meg, minek a megvalósítását segíti az MVVM Toolkit a megoldásban.
- A korábbi XAML megoldását úgy egészítse ki, hogy `*1`, `*2` stb.-vel jelöljön meg helyeket benne, ahova új kódot kell beszúrni, majd külön adja meg ezekre vonatkozóan a beszúrandó kódot. Az újonnan bevezetett XAML részeken túl adja meg a *code behind* fájlban levő osztály kódját is!

## c) (4p)

A feladat a korábbi megoldás „átalakítása” MVVM minta alapúra (nem szükséges a *Strict* modellt használnia):

- Írjon meg egy megfelelő, ablakhoz tartozó `ViewModel` osztályt.
- A viselkedés/működés meg kell egyezzen a korábbival: a gombkattintást **Command minta** alapján valósítsa meg (ehhez használhat MVVM Toolkit támogatást), vagy eseménykötéssel.
- Parancsok tiltásával/engedélyezésével nem kell foglalkoznia.
- Adja meg, hogy a XAML fájlban milyen változtatásokra van szükség (ehhez használjon `#1`, `#2` stb. hivatkozásokat).
- Adja meg egy mondatban, hogy ennél a megoldásnál **mit tartalmaz** az ablakhoz tartozó *code behind* fájl.

a) XAML elrendezés megvalósítása (10p)
Az alábbi XAML kód megvalósítja a kért kétszer-kettes rácsszerkezetet, és a benne lévő vezérlőket. A *1, *2, *3 jelölések mutatják a b) feladat adatkötéseinek helyét.

```xml
<Grid>

    <Grid.ColumnDefinitions>
        
        <ColumnDefinition Width="Auto" />
        
        <ColumnDefinition Width="*" />

    </Grid.ColumnDefinitions>

    <Grid.RowDefinitions>
        
        <RowDefinition Height="Auto" />
        
        <RowDefinition Height="80" />

    </Grid.RowDefinitions>


    <TextBlock Grid.Row="0" 
               Grid.Column="0" 
               Text="Quantity" 
               VerticalAlignment="Center" />


    <StackPanel Grid.Row="0" 
                Grid.Column="1" 
                Orientation="Horizontal" 
                HorizontalAlignment="Center">

        <TextBox Width="30" 
                 *1 />

        <Button Content="+" 
                *2 />

    </StackPanel>


    <TextBox Grid.Row="1" 
             Grid.Column="1" 
             Height="40" 
             HorizontalAlignment="Stretch" 
             VerticalAlignment="Top" 
             Margin="10,15,10,0" 
             *3 />

</Grid>
```

b) Adatkötés (Data Binding) MVVM nélkül (6p)
Az MVVM Toolkit szerepe a megoldásban (1 mondat):
Az MVVM Toolkit az ObservableObject ősosztály és az [ObservableProperty] attribútumok segítségével automatikusan legenerálja az INotifyPropertyChanged interfészhez szükséges kódot, így a felület automatikusan értesül, ha a háttérben megváltozik egy adat.

1. Az OrderItem osztály:

```csharp
// Az OrderItem osztály, amely az ObservableObject-ből örököl a felületértesítések miatt
public partial class OrderItem : ObservableObject
{
    // A toolkit legenerál egy 'Quantity' publikus tulajdonságot, ami frissíti a felületet
    [ObservableProperty]
    private int quantity;

    // A toolkit legenerál egy 'Description' publikus tulajdonságot, ami frissíti a felületet
    [ObservableProperty]
    private string description;
}
```

2. A Code behind osztály (MainWindow.xaml.cs):

```csharp
// Az ablakhoz tartozó C# háttérkód
public sealed partial class MainWindow : Window
{
    // Publikus tulajdonság a rendelési tételnek, hogy a XAML hivatkozhasson rá a {x:Bind} segítségével
    public OrderItem MyItem { get; set; }

    // Konstruktor
    public MainWindow()
    {
        // A felületi elemek kötelező inicializálása
        this.InitializeComponent();
        
        // Létrehozunk egy kezdő objektumot a rendelési tételnek
        MyItem = new OrderItem { Quantity = 1, Description = "Példa rendelés" };
    }

    // A "+" gombhoz tartozó kattintási eseménykezelő metódus
    private void PlusButton_Click(object sender, RoutedEventArgs e)
    {
        // A gomb megnyomásakor a háttérobjektum mennyiségét növeljük 1-gyel.
        // A felület a beépített értesítés miatt automatikusan frissülni fog.
        MyItem.Quantity++;
    }
}
```

3. A XAML fájl kiegészítései (az a) feladat jelölései helyére):

```xml
Text="{x:Bind MyItem.Quantity, Mode=TwoWay}"

Click="PlusButton_Click"

Text="{x:Bind MyItem.Description, Mode=TwoWay}"
```

c) Átalakítás MVVM minta alapúra (4p)
Ebben a részben az adatokat és a gombkattintás logikáját (Parancs / Command formájában) egy külön ViewModel osztályba szervezzük ki.

1. A ViewModel osztály:

```csharp
// A felülethez tartozó ViewModel osztály
public partial class MainWindowViewModel : ObservableObject
{
    // Rendelési mennyiség
    [ObservableProperty]
    private int quantity;

    // Rendelés leírása
    [ObservableProperty]
    private string description;

    // Ez az attribútum automatikusan generál egy 'IncreaseQuantityCommand' nevű parancsot
    [RelayCommand]
    private void IncreaseQuantity()
    {
        // A parancs meghívásakor növeljük a mennyiséget
        Quantity++;
    }
}
```

2. A Code behind feladata egy mondatban:
Ennél az MVVM megoldásnál a code behind fájl már nem tartalmaz eseménykezelőket, kizárólag az ablak inicializálását (InitializeComponent()) végzi el, valamint létrehozza és publikus tulajdonságként (pl. ViewModel néven) elérhetővé teszi a MainWindowViewModel egy példányát a XAML számára.

3. A XAML fájl szükséges változtatásai (a b) részhez képest):

```xml
Text="{x:Bind ViewModel.Quantity, Mode=TwoWay}"

Command="{x:Bind ViewModel.IncreaseQuantityCommand}"

Text="{x:Bind ViewModel.Description, Mode=TwoWay}"
```