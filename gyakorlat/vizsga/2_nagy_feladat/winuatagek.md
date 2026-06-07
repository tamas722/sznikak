# 1. Szöveges vezérlők
* `<TextBlock>`: Formázott szöveg megjelenítésére szolgál, egy vagy több sorban.
    * Text: A megjelenítendő szöveges tartalom.
    * FontFamily, Foreground, FontSize, FontWeight: A betűtípus, betűszín, betűméret és vastagság beállításai.
    * TextWrapping: Meghatározza a szöveg tördelését.
* `<TextBox>`: Szövegdoboz, amely szerkeszthető, a felhasználó bele tud gépelni.
    * Text: A beírt vagy megjelenített szöveg értéke.
    * TextWrapping, AcceptsReturn: Lehetővé teszi a többsoros és tördelt szövegbevitelt.
    * IsSpellCheckEnabled: Bekapcsolja a helyesírásellenőrzést.

# 2. Gombok és kapcsolók
* `<Button>`: Alapvető nyomógomb.
    * Content: A gomb tartalma, ami lehet szöveg, kép vagy akár más összetett vezérlő is.
    * Click: A gombkattintás eseményét kezeli.
    * Command, CommandParameter: MVVM tervezési minta esetén a parancsok hozzákötésére szolgál.
* `<RepeatButton>`: Olyan gomb, amely nyomva tartás esetén többször is elsüti a kattintás eseményt.
    * Delay, Interval: A késleltetést és az ismétlődési időközt állítják be.
* `<HyperlinkButton>`: Linkeket megjelenítő gomb.
    * MapsUri: Automatikusan megnyitja a megadott URL-t a böngészőben.
* `<ToggleButton>`, `<CheckBox>`, `<RadioButton>`: Állapotkapcsolók és jelölőnégyzetek.
    * IsChecked: A vezérlő bejelölt állapota (CheckBox esetén null érték is lehet).
    * IsThreeState (ToggleButton esetén): Háromállapotú mód engedélyezése.
    * GroupName (RadioButton esetén): Csoportba foglalja a gombokat, így csak egy választható ki közülük.

# 3. Tartalom és listamegjelenítők
* `<ContentControl>`: Valamilyen tartalom (pl. szöveg vagy objektumfa) megjelenítésére szolgáló alapvezérlő.
    * Content: A megjelenítendő tartalom.
    * ContentTemplate: A tartalom szerkezetét leíró sablon.
* `<ItemsControl>`, `<ComboBox>`, `<ListBox>`, `<ListView>`: Listák megjelenítését és elemek kiválasztását szolgálják.
    * Items, ItemsSource: A megjelenítendő fix lista vagy adatkötött forrás.
    * ItemTemplate: Az egyes listaelemek vizuális felépítését (sablonját) írja le.
    * ItemsPanel: A listaelemeket elrendező panel típusa.
    * SelectedItem, SelectedIndex: A kiválasztott elemet vagy annak indexét jelölik.
    * Header, PlaceholderText (ComboBox esetén): A legördülő lista címe és az üres állapotot jelző helykitöltő szöveg.

# 4. Elrendezések (Panelek) és tárolók
* `<Grid>`: Táblázatos elrendezést biztosít.
    * RowDefinitions, ColumnDefinitions: A sorok és oszlopok méreteit (fix, arányos vagy auto) határozzák meg.
    * Csatolt tulajdonságai a benne lévő elemeken: Grid.Row, Grid.Column (pozíció cellák szerint), valamint Grid.RowSpan, Grid.ColumnSpan (több cella összevonása).
* `<StackPanel>`: Az elemeket egymás alá vagy egymás mellé rendezi.
    * Orientation: Az elrendezés iránya (függőleges vagy vízszintes).
    * Spacing: A vezérlők közötti térkihagyás.
* `<Canvas>`: Szabad elhelyezést tesz lehetővé abszolút koordináták alapján.
    * Csatolt tulajdonságai a gyerek elemeken: Canvas.Left, Canvas.Top, Canvas.ZIndex (x, y pozíció és a rétegek sorrendje).
* `<VariableSizedWrapGrid>`: Folytonos (tördelődő) elrendezés.
    * ItemWidth, ItemHeight: Az elemek alapértelmezett mérete.
    * MaximumRowsOrColumns, Orientation: Az adott irányban elhelyezhető elemek maximális száma és a folytonosság iránya.
    * Csatolt tulajdonságai: VariableSizedWrapGrid.RowSpan, VariableSizedWrapGrid.ColumnSpan.
* `<RelativePanel>`: Elemek egymáshoz vagy a panelhez viszonyított elhelyezését oldja meg.
    * Csatolt tulajdonságai: Pl. RelativePanel.Below, RelativePanel.RightOf, RelativePanel.AlignTopWith stb.
* `<Border>`: Szegélyt és hátteret ad egy elemnek.
    * Background: A háttérszín.
    * CornerRadius: A sarkok lekerekítése.
    * BorderBrush, BorderThickness: A keret színe és vastagsága.

# 5. Általános méretezési és megjelenési tulajdonságok (több vezérlőn elérhetőek)
* Width, Height, MinWidth, MinHeight: A vezérlők szélessége és magassága.
* HorizontalAlignment, VerticalAlignment: A vezérlő szülőn belüli igazítása (Left, Center, Right, Stretch).
* Margin: A vezérlő külső távolsága a többi elemtől.
* Padding: A vezérlő belső tartalmának távolsága a szélektől.
* Visibility: A vezérlő láthatósága (Visible / Collapsed).

# 6. Sablonok és Stílusok (Metaprogramozás)
* `<Style>`: Egy vezérlő kinézetét és tulajdonságait egységesíti.
    * TargetType: Meghatározza, milyen típusú vezérlőre vonatkozik.
    * x:Key: A stílus hivatkozási neve az erőforrásokban.
    * BasedOn: Egy másik stílusból való öröklődés.
* `<DataTemplate>`: Adatok megjelenítési sémája.
    * x:DataType: Meghatározza, hogy milyen típusú adatforráshoz kapcsolódik a sablon, ami típusos adatkötésnél fontos.
    * x:Key: Sablon kulcsa az azonosításhoz.