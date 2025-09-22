```pascal title:"Wyświetl historie"
{$ADDTYPE TstQuery}
{$ADDTYPE TfWindowPlugins}
{$ADDTYPE TWindowPlugins}

procedure wyswietlHistorie;
var
  wp: TWindowPlugins;
  aSqlSelect, aSqlFrom, aSqlWhere, aSqlOrderBy, aSqlOrderType: String;
  wId: Integer;
  IdNagl: String;
  vDefPola: TDefPola;

begin
   IdNagl := Trim(string(DATAIN_ID_NAGL));
   //IdNagl := '10064';
   if (IdNagl <> '') then
   begin
     try
       wp := TWindowPlugins.Create(0);
       wp.Caption := 'Historia Cechy';
       wp.IdColumns := 'data_modyfikacji_fakt';
       aSqlSelect := 'x.id_zmiana_cechy, CD.nazwa, x.data_modyfikacji_fakt, coalesce(x.wart_varchar, x.wart_numeric) as wartosc, cast(x.data_zmieniona as varchar(50)) as data_zmieniona';
       aSqlFrom := 'XXX_HIST_ZMIAN_CECHY_ZOFE x'
                 + ' join cechadokk cd on cd.id_cechadokk = x.id_cechadokk';
       aSqlWhere := 'x.ID_NAGL = ' + IdNagl;
       aSqlOrderBy := 'x.data_modyfikacji_fakt';
       aSqlOrderType := 'DESC';
       wp.SqlSet(aSqlSelect, aSqlFrom, aSqlWhere, aSqlOrderBy, aSqlOrderType, '');

       vDefPola := TdefPola.Create; //Czasami tak trzeba przez takie coś przy kolumnach XXX nwm dlaczego
       vDefPola.FieldName := 'data_zmieniona';
       vDefPola.FieldType := ftString;
       vDefPola.rodzajWysw := rwInne;
       vDefPola.OpisPola := 'data_zmieniona';
       vDefPola.visible := True;
       vDefPola.DisplayLabel := 'Data cechy';
       vDefPola.required := True;
       vDefPola.rodzajPola := rpData;
       wp.fWindowsPlugins.QueryMain.AddField(vDefPola, 'x');

       vDefPola.FieldName := 'wartosc';
       vDefPola.OpisPola := 'wartosc';
       vDefPola.DisplayLabel := 'Wartość';
       wp.fWindowsPlugins.QueryMain.AddField(vDefPola, 'x');
       wp.AddFields('cechadokk', 'nazwa', 'cd');
       wp.AddFieldsXXX('XXX_HIST_ZMIAN_CECHY_ZOFE', 'data_modyfikacji_fakt', 'Data modyfikacji', 'x');

       wp.ShowWindow(wId);
     finally
       wp.Free;
       wp := nil;
       vDefPola.Free;
     end;
   end;
end;
```
```pascal title:"Show window"
procedure ShowWindow();
var
  aSqlSelect, aSqlFrom, aSqlWhere, aSqlOrderBy, aSqlOrderType, aSqlGroupBy: String;
  vSql:String;
  ID_KONTRAH : string;
  vDataSource: TDataSource;
  wp: TWindowPlugins;
  checkValues: string;

begin
  if Assigned(wp) then Exit;
  wp := TWindowPlugins.Create(0);
  ID_KONTRAH := GetGlobalData('ID_KONTRAH');
  try
      wp.Caption := 'Wysyłanie';
      wp.IdColumns := 'ID_EMAILOSOBAKTR';

      aSqlSelect := 'K.ID_KONTRAH,' + #39 + 'OsobaKontaktowa' + #39 + 'as TYP, K.NRKONTRAH, K.NAZWASKR, (O.NAZWISKO || ' + #39 + ' ' + #39 + ' || O.IMIE) as NAZWISKOIMIE, DK.MIEJSCOWOSC, T.NUMER, E.EMAIL, TM.NAZWATYPU, E.ID_EMAILOSOBAKTR';
      aSqlFrom := 'ODDZIALKONTRAH OD'
                + ' join KONTRAH K on K.ID_KONTRAH = OD.ID_KONTRAH'
                + ' join DANEKONTRAH DK on (DK.ID_KONTRAH = K.ID_KONTRAH) and (DK.BAZADANEKONTRAH = 1)'
                + ' join OSOBAKONTRAH O on (OD.ID_ODDZIALKONTRAH = O.ID_ODDZIALKONTRAH)'
                + ' left outer join TELOSOBAKTR T on (O.ID_TELOSOBAKTR = T.ID_TELOSOBAKTR)'
                + ' left outer join EMAILOSOBAKTR E on (O.ID_osobaKontrah = E.ID_osobaKontrah)'
                + ' left outer join TYPEMAIL TM on (TM.ID_TYPEMAIL = E.ID_TYPEMAIL)';
      aSqlWhere := '((E.EMAIL <> ' + #39 + #39 +') AND (K.ID_KONTRAH = ' + ID_KONTRAH + '))';
      aSqlOrderBy := 'K.ID_KONTRAH';
      aSqlOrderType := 'ASC';
      aSqlGroupBy := '';
      wp.SqlSet(aSqlSelect, aSqlFrom, aSqlWhere, aSqlOrderBy, aSqlOrderType, aSqlGroupBy);
      wp.AddFields('TYPEMAIL', 'NAZWATYPU', 'TM');
      wp.AddFields('OSOBAKONTRAH', 'NAZWISKOIMIE', 'O');
      wp.AddFields('EMAILOSOBAKTR', 'EMAIL;ID_EMAILOSOBAKTR', 'E');
      wp.AddFields('KONTRAH', 'NAZWASKR', 'K');
      if wp.ShowWindowCheckStr(checkValues) then //UWAGA przy niezaznaczeniu z lewej strony żadnego checkbox i tak bierze pozycje zaznaczoną
      begin
        if (checkValues <> '') then
        begin
          inf300(checkValues);
          vSQL := 'SELECT E.EMAIL FROM EMAILOSOBAKTR E JOIN TYPEMAIL TM on (TM.ID_TYPEMAIL = E.ID_TYPEMAIL) WHERE E.ID_EMAILOSOBAKTR IN (' + tempString + ')';
          vDataSource := OpenQuerySQL(vSQL, 0);
          if vDataSource <> nil then
          begin
            try
              vDataSource.DataSet.First;
              while not vDataSource.DataSet.Eof do
              begin
                inf300('1');
                eMails.Add(vDataSource.DataSet.FIELDBYNAME('EMAIL').ASSTRING);
                vDataSource.DataSet.Next;
              end;
            finally
              CloseQuerySQL(vDataSource);
            end;
          end;
          inf300('2');
          inf300(eMails[0] + eMails[1]);
        end;
      end;
  finally
      wp.Free;
      wp := nil;
  end;
end;
```
```pascal title:"Dodaj wiele dokumentów"
{$ADDTYPE TfWindowPlugins}
{$ADDTYPE TWindowPlugins}
procedure DodajWieleDokumentow_OnClick(Sender: TObject);
var
  wP: TWindowPlugins;
  vListaId: TIntegerList;
  vSql: string;
  i: Integer;
begin
  wp := TWindowPlugins.Create(cWindowIdWieleDokumentow);
  try
    wp.Caption := 'Wybierz dokumenty';

    wp.IdColumns := 'id_nagl';

    wp.SqlSet('n.id_nagl, n.datadok, n.nrdokzew, n.nrdokwew, n.sumaog, dd.skrotdefdok, k.nazwaskr',
      'nagl n'
      + ' inner join defdok dd on n.id_defdok = dd.id_defdok'
      + ' left outer join kontrah k on n.id_kontrah = k.id_kontrah',
      'n.bazanagl in (0, 5, 3, 8, 13, 14)'
      + ' and dd.id_defdok in (10174)'
      + ' and ((n.datadok >= dateadd (-6 month to current_date)) and (n.datadok <= current_date))', '', '', '');

    wp.AddFields('nagl', 'id_nagl;datadok;nrdokzew;nrdokwew;sumaog', 'n');
    wp.AddFields('defdok', 'skrotdefdok', 'dd');
    wp.AddFields('kontrah', 'nazwaskr', 'k');

    if not wp.ShowWindowCheck(vListaId) then Exit;
  finally
    wp.Free
  end;

  if (vListaId = nil) then Exit;

  try
    if (vListaId.Count > 0) then
      for i := 0 to vListaId.Count - 1 do
      begin
        vSql := 'update or insert into naglsprz (rodzajsprznagl, id_naglco, id_naglczym)'
             + ' values ('
             + '45' + ', '
             + IntToStr(fDokGmEd.PluginID_Nagl) + ', '
             + IntToStr(vListaId.GetValue(i)) + ')'
             + ' matching (rodzajsprznagl, id_naglco, id_naglczym)';

        ExecuteSQL(vSql, 0);
      end;
  finally
    vListaId.Free;
  end;
end;
```
```pascal title:"Branie wartości z zaznaczonych wierszy"
if (frmDokumenty.fWindowsPlugins.QueryMain.Locate('ID_POZ', vListaId.GetValue(i), [])) then
begin
  kodTow := frmDokumenty.fWindowsPlugins.QueryMain.FieldByName('INDEKS').AsString;
  ilosc := frmDokumenty.fWindowsPlugins.QueryMain.FieldByName('ILOSC').AsString;
  lp := frmDokumenty.fWindowsPlugins.QueryMain.FieldByName('LP').AsString;
  //inf300(kodTow + ' ### ' + ilosc);
  createHeaderDocumentReceiveInner(ilosc, kodTow, lp);
end;
```
```pascal title:"Zaznaczenie wiersza"
wId := 30;
    if wp.ShowWindow(wId) then
      ...
```

```pascal title:"Edytowalny grid"
CONTIMAX - TfPromTow
PAWO - TfDokGmEdV2


//------------------------------------------------------------------------------
procedure DodajFieldNumeric(aDataSet: TDataSet; aFieldName, aDisplayLabel: String; aVisible: Boolean);
var vField : TBCDField;
begin
  vField              := TBCDField.Create(aDataSet);
  vField.FieldName    := aFieldName;
  vField.DisplayLabel := aDisplayLabel;
  TBCDField(vField).DisplayFormat := '0.0000';
  vField.DataSet      := aDataSet;
  vField.Visible      := aVisible;
end;
//------------------------------------------------------------------------------
procedure DodajFieldInteger(aDataSet: TDataSet; aFieldName, aDisplayLabel: String; aVisible: Boolean);
var vField : TIntegerField;
begin
  vField              := TIntegerField.Create(aDataSet);
  vField.FieldName    := aFieldName;
  vField.DisplayLabel := aDisplayLabel;
  vField.DataSet      := aDataSet;
  vField.Visible      := aVisible;
end;
//------------------------------------------------------------------------------
procedure wp_OnCloseQuery(Sender: TObject; var CanClose: Boolean);
var vTrans : TstTransaction;
begin
  //Potwierdzenie transakcji jeśli jest otwarta
  if not (Sender is TfWindowPlugins) then
    Exit;

  vTrans := TstTransaction(TstQuery(TfWindowPlugins(Sender).DBGmain.DataSource.DataSet).Transaction);
  if not Assigned(vTrans) then
    Exit;

  if not TstTransaction(vTrans).InTransaction then
    Exit;

  if not PytTNie('Zapisać wprowadzone zmiany?') then
    Exit;

  TstTransaction(vTrans).Commit;
end;
//------------------------------------------------------------------------------
procedure wp_DataSourceOnDataChange(Sender: TObject; Field: TField);
var vTrans : TstTRANSACTION;
    vSQL   : String;
begin
  if not Assigned(Field) then
    Exit;

//Dodanie aktualizacja danych
  vTrans := TstTRANSACTION(TStQuery(Field.DataSet).Transaction);

  vSQL := 'EXECUTE BLOCK' + sLineBreak +
          'AS    ' + sLineBreak +
          '  declare variable ID_PROMTOW integer;       ' + sLineBreak +
          '  declare variable ID_KARTOTEKA integer;     ' + sLineBreak +
          '  declare variable OKRES_MIESIAC varchar(14);' + sLineBreak +
          '  declare variable PLAN_NADZIEN date;        ' + sLineBreak +
          '  declare variable ILOSC numeric(18,4);      ' + sLineBreak +
          'BEGIN ' + sLineBreak +
          '   ID_PROMTOW    = ' + Field.DataSet.FieldByName('ID_PROMTOW').AsString + ';' + sLineBreak +
          '   ID_KARTOTEKA  = ' + Field.DataSet.FieldByName('ID_KARTOTEKA').AsString + ';' + sLineBreak +
          '   OKRES_MIESIAC = ' + QuotedStr(Field.DisplayName) + ';' + sLineBreak +
          '   PLAN_NADZIEN  = CAST(:OKRES_MIESIAC || ''-01'' AS Date);' + sLineBreak +
          '   ILOSC         = ' + CurrToStrSt(Field.AsCurrency) + ';' + sLineBreak +
          '   ' + sLineBreak +
          '   --Kasujemy wszystko co jest w okresie' + sLineBreak +
          '   DELETE FROM  XXX_PROMTOWDLAKART_PLAN_PROD WHERE ID_PROMTOW = :ID_PROMTOW AND ID_KARTOTEKA = :ID_KARTOTEKA AND OKRES_MIESIAC = :OKRES_MIESIAC;' + sLineBreak +
          '   ' + sLineBreak +
          '   --Dodajemy jeden rekord dla danego miesiaca na pierwszy dzien' + sLineBreak +
          '   UPDATE OR INSERT INTO XXX_PROMTOWDLAKART_PLAN_PROD (ID_PROMTOW,  ID_KARTOTEKA,  PLAN_NADZIEN,  ILOSC) ' + sLineBreak +
          '                                                VALUES(:ID_PROMTOW, :ID_KARTOTEKA, :PLAN_NADZIEN, :ILOSC)' + sLineBreak +
          '   MATCHING(ID_PROMTOW, ID_KARTOTEKA, PLAN_NADZIEN);' + sLineBreak +
          'END';

  vTrans.WykonajSQL(vSQL);
  if CzyBladApiError('wp_DataSourceOnDataChange().', vSQL) then
  begin
    TstQuery(Field.DataSet).stClose('');
    TstQuery(Field.DataSet).stOpen('');
  end;
end;
//------------------------------------------------------------------------------
procedure fPromTow_PlanowanieProdukcji(Sender: TObject);
var wp       : TWindowPlugins;
    vDataSet : TDataSet;
    vConn    : TFDCustomConnection;

    i, vId_Promtow : Integer;
    vSqlSelect, vSqlFrom, vSqlWhere, vOrderBy : String;
    vNazwaPromocji : String;

    OkresMinus1, OkresZero, OkresPlus1, OkresPlus2 : String; //MS tytuly kolumn jednoczesie sa warunkami WHERE podzapytan
begin
  //MS 2021-07-12 Planowanie produkcji

  wp := nil;
  try
    if not Assigned(fPromTow) then
    begin
      UwagaOk('Tormularz TfPromTow nie został utworzony.');
      Exit;
    end;

    vId_Promtow := fPromTow.QueryMain.FieldByRelName('id_promtow').AsInteger;
    if vId_Promtow = 0 then
    begin
      UwagaOk('Promocja nie została jeszcze utworzona.');
      Exit;
    end;

    OkresMinus1 := FormatDateTime('yyyy-mm', IncMonth(Date(), -1));
    OkresZero   := FormatDateTime('yyyy-mm', Date());
    OkresPlus1  := FormatDateTime('yyyy-mm', IncMonth(Date(), 1));
    OkresPlus2  := FormatDateTime('yyyy-mm', IncMonth(Date(), 2));

    vNazwaPromocji := 'Kartoteki w umowie/promocji: "' + fPromTow.QueryMain.FieldByRelName('opis').AsString + '"';

    vSqlSelect := '  PT.ID_PROMTOW   ' + sLineBreak +
                  '  ,K.ID_KARTOTEKA ' + sLineBreak +
                  '  ,K.INDEKS       ' + sLineBreak +
                  '  ,K.NAZWASKR     ' + sLineBreak +
                  '  ,K.NAZWADL      ' + sLineBreak +
      //Aktualny Miesiąc -1
                  '  ,Okres_MINUS1.Ilosc AS OkresMinus1' + sLineBreak +
      //Aktualny Miesiąc
                  '  ,Okres_ZERO.Ilosc   AS OkresZero  ' + sLineBreak +
      //Aktualny Miesiąc +1
                  '  ,Okres_PLUS1.Ilosc  AS OkresPlus1 ' + sLineBreak +
      //Aktualny Miesiąc -2
                  '  ,Okres_PLUS2.Ilosc  AS OkresPlus2 ';

    vSqlFrom   := 'PROMTOWDLAKART PT' + sLineBreak +
                  '  INNER JOIN KARTOTEKA K ON PT.ID_KARTOTEKA = K.ID_KARTOTEKA AND PT.ID_PROMTOW = ' + IntToStr(vId_Promtow) + sLineBreak +
                  '' + sLineBreak +
       //Aktualny Miesiąc -1
                  '  LEFT JOIN (SELECT' + sLineBreak +
                  '                   XP.ID_PROMTOW    ' + sLineBreak +
                  '                   ,XP.ID_KARTOTEKA ' + sLineBreak +
                  '                   ,SUM(ilosc) AS ilosc           ' + sLineBreak +
                  '             FROM XXX_PROMTOWDLAKART_PLAN_PROD XP ' + sLineBreak +
                  '             WHERE                                ' + sLineBreak +
                  '                   XP.OKRES_MIESIAC = ' + QuotedStr(OkresMinus1) + sLineBreak +
                  '             GROUP BY 1, 2 ' + sLineBreak +
                  '            )Okres_MINUS1 ON PT.ID_PROMTOW = Okres_MINUS1.ID_PROMTOW AND PT.ID_KARTOTEKA = Okres_MINUS1.ID_KARTOTEKA' + sLineBreak +
                  '' + sLineBreak +

       //Aktualny Miesiąc
                  '  LEFT JOIN (SELECT' + sLineBreak +
                  '                   XP.ID_PROMTOW    ' + sLineBreak +
                  '                   ,XP.ID_KARTOTEKA ' + sLineBreak +
                  '                   ,SUM(ilosc) AS ilosc           ' + sLineBreak +
                  '             FROM XXX_PROMTOWDLAKART_PLAN_PROD XP ' + sLineBreak +
                  '             WHERE                                ' + sLineBreak +
                  '                   XP.OKRES_MIESIAC = ' + QuotedStr(OkresZero) + sLineBreak +
                  '             GROUP BY 1, 2 ' + sLineBreak +
                  '            )Okres_ZERO ON PT.ID_PROMTOW = Okres_ZERO.ID_PROMTOW AND PT.ID_KARTOTEKA = Okres_ZERO.ID_KARTOTEKA' + sLineBreak +

       //Aktualny Miesiąc +1
                  '  LEFT JOIN (SELECT' + sLineBreak +
                  '                   XP.ID_PROMTOW    ' + sLineBreak +
                  '                   ,XP.ID_KARTOTEKA ' + sLineBreak +
                  '                   ,SUM(ilosc) AS ilosc           ' + sLineBreak +
                  '             FROM XXX_PROMTOWDLAKART_PLAN_PROD XP ' + sLineBreak +
                  '             WHERE                                ' + sLineBreak +
                  '                   XP.OKRES_MIESIAC = ' + QuotedStr(OkresPlus1) + sLineBreak +
                  '             GROUP BY 1, 2 ' + sLineBreak +
                  '            )Okres_PLUS1 ON PT.ID_PROMTOW = Okres_PLUS1.ID_PROMTOW AND PT.ID_KARTOTEKA = Okres_PLUS1.ID_KARTOTEKA' + sLineBreak +

       //Aktualny Miesiąc +2
                  '  LEFT JOIN (SELECT' + sLineBreak +
                  '                   XP.ID_PROMTOW    ' + sLineBreak +
                  '                   ,XP.ID_KARTOTEKA ' + sLineBreak +
                  '                   ,SUM(ilosc) AS ilosc           ' + sLineBreak +
                  '             FROM XXX_PROMTOWDLAKART_PLAN_PROD XP ' + sLineBreak +
                  '             WHERE                                ' + sLineBreak +
                  '                   XP.OKRES_MIESIAC = ' + QuotedStr(OkresPlus2) + sLineBreak +
                  '             GROUP BY 1, 2 ' + sLineBreak +
                  '            )Okres_PLUS2 ON PT.ID_PROMTOW = Okres_PLUS2.ID_PROMTOW AND PT.ID_KARTOTEKA = Okres_PLUS2.ID_KARTOTEKA';
    vSqlWhere  := '(1 = 1)';
    vOrderBy   := 'K.INDEKS ASC';

    wp := TWindowPlugins.Create(0);
    wp.Caption   := vNazwaPromocji;
    wp.IdColumns := 'ID_KARTOTEKA';
    wp.SqlSet(vSqlSelect, vSqlFrom, vSqlWhere, vOrderBy, '', '');

    wp.AddFields('KARTOTEKA', 'INDEKS;NAZWASKR;NAZWADL;ID_KARTOTEKA', 'K');
    wp.LastField.VISIBLE := False;

    vDataSet := TDataSet(wp.fWindowsPlugins.DBGmain.DataSource.DataSet);

    DodajFieldNumeric(vDataSet, 'OkresMinus1', OkresMinus1, True);
    DodajFieldNumeric(vDataSet, 'OkresZero', OkresZero, True);
    DodajFieldNumeric(vDataSet, 'OkresPlus1', OkresPlus1, True);
    DodajFieldNumeric(vDataSet, 'OkresPlus2', OkresPlus2, True);
    DodajFieldInteger(vDataSet, 'ID_PROMTOW', 'ID_PROMTOW', False); //Pole potrzebne do wygnerowania warunku w DataSetUpdate

  //Ustawiamy Grid-a do możliwości edycji
    wp.fWindowsPlugins.DBGmain.DataSource.AutoEdit := True;
    wp.fWindowsPlugins.DBGmain.ReadOnly := False;
    wp.fWindowsPlugins.DBGmain.Options  := wp.fWindowsPlugins.DBGmain.Options  - [dgRowSelect];
    wp.fWindowsPlugins.DBGmain.Options  := wp.fWindowsPlugins.DBGmain.Options  + [dgEditing];

  //Blokujemy domyślnie wyszystkie pola do edycji
    for i := 0 to vDataSet.Fields.Count -1 do
      vDataSet.Fields[i].ReadOnly := True;

  //Odblokowujemy do edycji tylko kilka wybranych
    vDataSet.FieldByName('OkresMinus1').ReadOnly := False;
    vDataSet.FieldByName('OkresZero').ReadOnly   := False;
    vDataSet.FieldByName('OkresPlus1').ReadOnly  := False;
    vDataSet.FieldByName('OkresPlus2').ReadOnly  := False;

  //Tworzymy własną transakcję w celu edycji danych - sama edycja w zdarzeniu DataSet.OnDataChange ale musimy mieć własną transakcje
    vConn := TstQuery(vDataSet).Transaction.Connection;
    TstQuery(vDataSet).Transaction            := TFIBTransaction(TstTransaction.Create(vDataSet.Owner));
    TstQuery(vDataSet).Transaction.Connection := vConn;

  //Ustawiamy UPDATESQL alby nic nie robiło
    with TstQuery(vDataSet).UpdateSQL do
    begin
      Clear;
      BeginUpdate;
      Add('UPDATE XXX_PROMTOWDLAKART_PLAN_PROD SET ');
      Add('    ilosc = ilosc');
      Add('WHERE (1 = 0) ');
      EndUpdate;
    end;

  //Przejmujemy zdarzenie
    TDataSource(wp.fWindowsPlugins.DBGmain.DataSource).OnDataChange := @WP_DataSourceOnDataChange;  //Właściwa aktualizacja danych w transakcji
    wp.fWindowsPlugins.OnCloseQuery := @WP_OnCloseQuery;  //Potwierdzenie Transakcji przy zamknięcu okna z pytaniem do użytkownika
    wp.fWindowsPlugins.DBGmain.DisabledFingerSearch := True; //Wyłaczamy możliwość szukania po tabeli skoro mamy edytować dane.

  //Pokazujemy WindowPlugin w trybie przeglądania.
    wp.ShowBrowseWindow(False);
  finally
    if Assigned(WP) then
      wp.Free;
    WP := nil;
  end; //try..finally
end;
//------------------------------------------------------------------------------
```

```pascal title:"Edytowalny grid - przykład z dodaniem pustej kolumny"
{$ADDTYPE TFIBDatabase}
{$ADDTYPE TKrDBGrid}
{$ADDTYPE TFDCustomConnection}
{$ADDTYPE TIBTransaction}
{$ADDTYPE TFIBTransaction}
{$ADDTYPE TstTRANSACTION}
{$ADDTYPE TstQuery}
{$ADDTYPE TfWindowPlugins}
{$ADDTYPE TWindowPlugins}

var
  wp: TWindowPlugins;

procedure DodajFieldNumeric(aDataSet: TDataSet; aFieldName, aDisplayLabel: String; aVisible: Boolean);
var vField : TBCDField;
begin
  vField              := TBCDField.Create(aDataSet);
  vField.FieldName    := aFieldName;
  vField.DisplayLabel := aDisplayLabel;
  TBCDField(vField).DisplayFormat := '0.0000';
  vField.DataSet      := aDataSet;
  vField.Visible      := aVisible;
end;

//Zabezpieczenie przed wciśnięciem Enter
procedure wpKalkulatorImportu_OnActivate(Sender: TObject);
begin
  wp.fWindowsPlugins.FormActivate(Sender);
  wp.fWindowsPlugins.AZ_ZamknijWybierz.ShortCut := 0;
end;

procedure ShowWindow();
var
  aSqlSelect, aSqlFrom, aSqlWhere, aSqlOrderBy, aSqlOrderType, aSqlGroupBy: String;
  vSql:String;
  vDataSource: TDataSource;

  checkValues: string;
  vDefPola: TDefPola;
  NewField: TFloatField;
  i: Integer;
  vDataSet : TDataSet;
  vConn    : TFDCustomConnection;
  wId: Integer;
  vListaId: TIntegerList;

begin
  if Assigned(wp) then Exit;
  wp := TWindowPlugins.Create(1);

  try
    wp.Caption := 'Test';
    wp.IdColumns := 'ID_NAGL';

    aSqlSelect := 'n.id_nagl, n.nrdokwew, 0.00 as Ilosc';
    aSqlFrom := 'nagl n'
    aSqlWhere := 'n.id_nagl > 11666';
    aSqlOrderBy := 'n.id_nagl';
    aSqlOrderType := 'ASC';
    aSqlGroupBy := '';
    wp.SqlSet(aSqlSelect, aSqlFrom, aSqlWhere, aSqlOrderBy, aSqlOrderType, aSqlGroupBy);

    wp.AddFields('nagl', 'id_nagl;nrdokwew', 'n');

    vDataSet := TDataSet(wp.fWindowsPlugins.DBGmain.DataSource.DataSet);
    DodajFieldNumeric(vDataSet, 'Ilosc', 'Ilość', True);

    //Ustawiamy Grid-a do możliwości edycji
    wp.fWindowsPlugins.DBGmain.DataSource.AutoEdit := True;
    wp.fWindowsPlugins.DBGmain.ReadOnly := False;
    wp.fWindowsPlugins.DBGmain.Options  := wp.fWindowsPlugins.DBGmain.Options  - [dgRowSelect];
    wp.fWindowsPlugins.DBGmain.Options  := wp.fWindowsPlugins.DBGmain.Options  + [dgEditing];
    wp.fWindowsPlugins.DBGmain.TitleClickSortEnabled := false;
    wp.fWindowsPlugins.DBGmain.OnKeyDown := nil;
    wp.fWindowsPlugins.DBGmain.OnKeyUp := nil;
    wp.fWindowsPlugins.DBGmain.OnDblClick := nil;

    //Blokujemy domyślnie wyszystkie pola do edycji
    for i := 0 to vDataSet.Fields.Count -1 do
      vDataSet.Fields[i].ReadOnly := True;

    //Blokujemy domyślnie wyszystkie pola do edycji
    vDataSet.FieldByName('Ilosc').ReadOnly  := False;

     //Tworzymy własną transakcję w celu edycji danych - sama edycja w zdarzeniu DataSet.OnDataChange ale musimy mieć własną transakcję
    vConn := TstQuery(vDataSet).Transaction.Connection;
    TstQuery(vDataSet).Transaction            := TFIBTransaction(TstTransaction.Create(vDataSet.Owner));
    TstQuery(vDataSet).Transaction.Connection := vConn;
    wp.fWindowsPlugins.ZamknijWyczyscDisabled := true;
    wp.fWindowsPlugins.OnActivate := @wpKalkulatorImportu_OnActivate;

    if wp.ShowWindowCheck(vListaId) then
    begin
      vDataSet.First;
      while not vDataSet.Eof do
      begin
        for i := 0 to vDataSet.FieldCount - 1 do
        begin
          inf300(vDataSet.FieldByName('nrdokwew').AsString);
          inf300(vDataSet.FieldByName('ilosc').AsString);
          vDataSet.next;
        end;
      end;
    end;
  finally
    wp.Free;
    frmWp := nil;
  end;
end;

begin
  ShowWindow();
end.
```