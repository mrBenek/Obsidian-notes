```pascal title:"Wyświetl historie"
{$ADDTYPE TstQuery}
{$ADDTYPE TfWindowPlugins}
{$ADDTYPE TWindowPlugins}

procedure wyswietlHistorie;
var
  frmHistoria: TWindowPlugins;
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
       frmHistoria := TWindowPlugins.Create(0);
       frmHistoria.Caption := 'Historia Cechy';
       frmHistoria.IdColumns := 'data_modyfikacji_fakt';
       aSqlSelect := 'x.id_zmiana_cechy, CD.nazwa, x.data_modyfikacji_fakt, coalesce(x.wart_varchar, x.wart_numeric) as wartosc, cast(x.data_zmieniona as varchar(50)) as data_zmieniona';
       aSqlFrom := 'XXX_HIST_ZMIAN_CECHY_ZOFE x'
                 + ' join cechadokk cd on cd.id_cechadokk = x.id_cechadokk';
       aSqlWhere := 'x.ID_NAGL = ' + IdNagl;
       aSqlOrderBy := 'x.data_modyfikacji_fakt';
       aSqlOrderType := 'DESC';
       frmHistoria.SqlSet(aSqlSelect, aSqlFrom, aSqlWhere, aSqlOrderBy, aSqlOrderType, '');

       vDefPola := TdefPola.Create; //Czasami tak trzeba przez takie coś przy kolumnach XXX nwm dlaczego
       vDefPola.FieldName := 'data_zmieniona';
       vDefPola.FieldType := ftString;
       vDefPola.rodzajWysw := rwInne;
       vDefPola.OpisPola := 'data_zmieniona';
       vDefPola.visible := True;
       vDefPola.DisplayLabel := 'Data cechy';
       vDefPola.required := True;
       vDefPola.rodzajPola := rpData;
       frmHistoria.fWindowsPlugins.QueryMain.AddField(vDefPola, 'x');

       vDefPola.FieldName := 'wartosc';
       vDefPola.OpisPola := 'wartosc';
       vDefPola.DisplayLabel := 'Wartość';
       frmHistoria.fWindowsPlugins.QueryMain.AddField(vDefPola, 'x');
       frmHistoria.AddFields('cechadokk', 'nazwa', 'cd');
       frmHistoria.AddFieldsXXX('XXX_HIST_ZMIAN_CECHY_ZOFE', 'data_modyfikacji_fakt', 'Data modyfikacji', 'x');

       frmHistoria.ShowWindow(wId);
     finally
       frmHistoria.Free;
       frmHistoria := nil;
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
  frmWp: TWindowPlugins;
  checkValues: string;

begin
  if Assigned(frmWp) then Exit;
  frmWp := TWindowPlugins.Create(0);
  ID_KONTRAH := GetGlobalData('ID_KONTRAH');
  try
      frmWp.Caption := 'Wysyłanie';
      frmWp.IdColumns := 'ID_EMAILOSOBAKTR';

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
      frmWp.SqlSet(aSqlSelect, aSqlFrom, aSqlWhere, aSqlOrderBy, aSqlOrderType, aSqlGroupBy);
      frmWp.AddFields('TYPEMAIL', 'NAZWATYPU', 'TM');
      frmWp.AddFields('OSOBAKONTRAH', 'NAZWISKOIMIE', 'O');
      frmWp.AddFields('EMAILOSOBAKTR', 'EMAIL;ID_EMAILOSOBAKTR', 'E');
      frmWp.AddFields('KONTRAH', 'NAZWASKR', 'K');
      if frmWp.ShowWindowCheckStr(checkValues) then //UWAGA przy niezaznaczeniu z lewej strony żadnego checkbox i tak bierze pozycje zaznaczoną
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
      frmWp.Free;
      frmWp := nil;
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
  wP := TWindowPlugins.Create(cWindowIdWieleDokumentow);
  try
    wP.Caption := 'Wybierz dokumenty';

    wP.IdColumns := 'id_nagl';

    wP.SqlSet('n.id_nagl, n.datadok, n.nrdokzew, n.nrdokwew, n.sumaog, dd.skrotdefdok, k.nazwaskr',
      'nagl n'
      + ' inner join defdok dd on n.id_defdok = dd.id_defdok'
      + ' left outer join kontrah k on n.id_kontrah = k.id_kontrah',
      'n.bazanagl in (0, 5, 3, 8, 13, 14)'
      + ' and dd.id_defdok in (10174)'
      + ' and ((n.datadok >= dateadd (-6 month to current_date)) and (n.datadok <= current_date))', '', '', '');

    wP.AddFields('nagl', 'id_nagl;datadok;nrdokzew;nrdokwew;sumaog', 'n');
    wP.AddFields('defdok', 'skrotdefdok', 'dd');
    wP.AddFields('kontrah', 'nazwaskr', 'k');

    if not wp.ShowWindowCheck(vListaId) then Exit;
  finally
    wP.Free
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
    if wP.ShowWindow(wId) then
      ...
```
```pascal title:"Dodanie kolumny do zakładki"
Dodanie kolumny do zakładki
procedure AddTabSheet;
var
  pPanel: TPanel;
begin
  TS_Pozycje := TTabSheet.Create(Self);
  TS_Pozycje.Name := 'TS_Pozycje';
  TS_Pozycje.Caption := 'Pozycje dokumentu';
  TS_Pozycje.PageControl := fDokZamowieniaOdOdb.PC_dodinfo;

  pPanel := TPanel.Create(Self);
  pPanel.Name := 'pPanel';
  pPanel.Parent := TS_Pozycje;
  pPanel.Align := alClient;
  pPanel.Caption := '';

  dbgPozycje := TKrDBGrid.Create(Self);
  dbgPozycje.Name := 'dbgPozycje';
  dbgPozycje.Parent := pPanel;
  dbgPozycje.Align := alClient;
  dbgPozycje.ReadOnly := True;
  dbgPozycje.TitleFont := fDokZamowieniaOdOdb.DBGmain.TitleFont;
  dbgPozycje.Options := fDokZamowieniaOdOdb.DBGmain.Options;
  dbgPozycje.Font := fDokZamowieniaOdOdb.DBGmain.Font;

  with dbgPozycje.Columns.Add do begin
    Expanded := False;
    FieldName := 'ID_POZ';
    Visible := False;
  end;
  with dbgPozycje.Columns.Add do begin
    Expanded := False;
    FieldName := 'LP';
    Title.Caption := 'Lp';
    Width := 40;
    Visible := True;
  end;
  with dbgPozycje.Columns.Add do begin
    Expanded := False;
    FieldName := 'INDEKS';
    Title.Caption := 'Indeks';
    Width := 100;
    Visible := True;
  end;
  with dbgPozycje.Columns.Add do begin
    Expanded := False;
    FieldName := 'NAZWASKR';
    Title.Caption := 'Identyfikator';
    Width := 100;
    Visible := True;
  end;
  with dbgPozycje.Columns.Add do begin
    Expanded := False;
    FieldName := 'kart_nazwadl';
    Title.Caption := 'Nazwa';
    Width := 400;
    Visible := True;
  end;
  with dbgPozycje.Columns.Add do begin
    Expanded := False;
    FieldName := 'ILOSC';
    Title.Caption := 'Ilość';
    Width := 84;
    Visible := True;
  end;
  with dbgPozycje.Columns.Add do begin
    Expanded := False;
    FieldName := 'JM';
    Title.Caption := 'JM';
    Width := 84;
    Visible := True;
  end;
  with dbgPozycje.Columns.Add do begin
    Expanded := False;
    FieldName := 'cenanetto';
    Title.Caption := 'Cena netto';
    Width := 84;
    Visible := True;
  end;
  with dbgPozycje.Columns.Add do begin
    Expanded := False;
    FieldName := 'wartbrutto';
    Title.Caption := 'Wartość brutto';
    Width := 84;
    Visible := True;
  end;
  with dbgPozycje.Columns.Add do begin
    Expanded := False;
    FieldName := 'uwagi';
    Title.Caption := 'Uwagi';
    Width := 400;
    Visible := True;
  end;
end;

procedure QPodstawaBeforeOpen(DataSet: TDataSet);
var
  vSql: string;
  vField: TField;
begin
  if (Pos(', kier.nazwiskoimie kierowca, kier.samochod nrrejestracyjny', frm.QPodstawa.SQL.Text) < 1) then
  begin
    vSql := frm.QPodstawa.SQL.Text;
    Zastap('NL.STATUS', 'NL.STATUS, kier.nazwiskoimie kierowca, kier.samochod nrrejestracyjny', vSql);
    Zastap('-1 as status', '-1 as status, null kierowca, null nrrejestracyjny', vSql);
    Zastap('D.ID_STATUSDYSP as STATUS', 'D.ID_STATUSDYSP as STATUS, null kierowca, null nrrejestracyjny', vSql);
    Zastap('INNER JOIN UZYTKOWNIK UZ ON (NL.ID_UZYTKOWNIK = UZ.ID_UZYTKOWNIK)' + sLineBreak,
      'INNER JOIN UZYTKOWNIK UZ ON (NL.ID_UZYTKOWNIK = UZ.ID_UZYTKOWNIK)' + sLineBreak
        + 'left join kierowca kier on nl.id_kierowca = kier.id_kierowca' + sLineBreak, vSql);
    Zastap('INNER JOIN UZYTKOWNIK UZ ON (Z.ID_UZYTKOWNIK = UZ.ID_UZYTKOWNIK)' + sLineBreak,
      'INNER JOIN UZYTKOWNIK UZ ON (Z.ID_UZYTKOWNIK = UZ.ID_UZYTKOWNIK)'  + sLineBreak
        + 'left join kierowca kier on nl.id_kierowca = kier.id_kierowca' + sLineBreak, vSql);
    Zastap('INNER JOIN UZYTKOWNIK UZ ON (NL.ID_UZYTKOWNIK = UZ.ID_UZYTKOWNIK)) ON (ZN.ID_NAGLZWIEKSZENIE = NL.ID_NAGL)' + sLineBreak,
      'INNER JOIN UZYTKOWNIK UZ ON (NL.ID_UZYTKOWNIK = UZ.ID_UZYTKOWNIK)) ON (ZN.ID_NAGLZWIEKSZENIE = NL.ID_NAGL)' + sLineBreak
        + 'left join kierowca kier on nl.id_kierowca = kier.id_kierowca' + sLineBreak, vSql);
    frm.QPodstawa.SQL.Text := vSql;
  end;

  vField := frm.QPodstawa.FindFieldFromSqlName('kierowca');
  if (vField = nil) then
  begin
    vField := TStringField.Create(TDataSet(frm.QPodstawa));
    vField.FieldName := 'kierowca';
    vField.DisplayLabel := 'Kierowca';
    vField.DataSet := TDataSet(frm.QPodstawa);
  end else
    vField.Visible := True;
  vField := frm.QPodstawa.FindFieldFromSqlName('nrrejestracyjny');
  if (vField = nil) then
  begin
    vField := TStringField.Create(TDataSet(frm.QPodstawa));
    vField.FieldName := 'nrrejestracyjny';
    vField.DisplayLabel := 'Nr rejestracyjny';
    vField.DataSet := TDataSet(frm.QPodstawa);
  end else
    vField.Visible := True;
//pole numeric
    if (field = nil) then
        begin
          field := TBCDField.Create(TDataSet(query));
          field.FieldName := cFieldNameIloscZrealizKor;
          field.DisplayLabel := cFieldCaptionIloscZrealizKor;
          field.DataSet := TDataSet(query);
          field.Size := 2;
          field.Visible := True;
          TBCDField(field).DisplayFormat := '0.0000';
        end;
      end;
end;

procedure BBzapisz_OnClick(Sender: TObject);
begin
  frm.BBzapiszClick(Sender);
end;
frm.BBzapisz.OnClick := @BBzapisz_OnClick;

procedure fZlecReklEd_BBzapisz_OnClick(Sender: TObject);
begin
  if (fZlecReklEd.klEdycji = F3) then

  fZlecReklEd.BBzapiszClick(Sender);
end;

fZlecReklEd.BBzapisz.OnClick := @fZlecReklEd_BBzapisz_OnClick;

procedure AD_Popraw_OnExecute(Sender: TObject);
begin
  if BlokujUzytkownikow = false then Exit;
  fDokZamowieniaOdOdb.AD_PoprawExecute(Sender);
end;
begin
  if (Self is TfDokMagazynDok) then
  begin
    if (fDokMagazynDok = nil) then
    begin
      fDokMagazynDok := TfDokMagazynDok(Self);
      fDokMagazynDok.AD_Popraw.OnExecute := @AD_Popraw_OnExecute;
      PluginsAddAction(Self, 'Popraw cechy dokumentów.', 'application_list_24', @PoprawCechyDokumentow_OnClick);
      PluginsAddAction(Self, 'Wyślij email z potwierdzeniem', 'mail_out_24', @WyslijMail_OnClick);
      PluginsAddAction(Self, 'Popraw numer dokumentu, kierowcę i cechy.', 'assets_24', @EdytujDokument_OnClick);
      PluginsAddAction(Self, 'Zmień ustawienia wyświetlanych cech MM-Edycja', 'Settings w10', @EdytujUstawieniaWyswietlaniaMM_OnClick);
      dsPozycje := nil;
      AddTabSheet;
      CreateDBEdit;
      TS_Pozycje.OnShow := @OnShowTabSheet;
      TempDBEdit.OnChange := @OnChangeDBEdit;
      PobIdUzytk;
      if (IdUzytkownik > 0) then
        if OdpowiedniaGrupaUzytkownikowAkwizytor then
          IdAkwizytorList := PobIdAkwizytorList;
    end;
    RefreshTabSheet;
    if (frm = nil) then
    begin
      frm :=  TfDokMagazynDok(Self);
      frm.QPodstawa.stClose('');
      frm.QPodstawa.BeforeOpen := @QPodstawaBeforeOpen;
      frm.QPodstawa.stOpen('');
    end;
    if not DodajKolumnyDodatkoweCechy() then
      if (IdAkwizytorList <> '') then
      begin
        fDokMagazynDok.QueryMain.BeforeOpen := @BeforeOpen;
        fDokMagazynDok.QueryMain.Close;
        fDokMagazynDok.QueryMain.Open('');
      end;
  end;
end.
```
```pascal title:"Usunięcie okna z widoku, aby np. kliknąć enter na oknie"
Usunięcie okna z widoku, aby np kliknąć enter na oknie:
frm.Constraints.MinHeight := 1;
frm.Constraints.MinWidth := 1;
frm.BorderStyle := bsNone;
frm.ClientHeight := 1;
frm.ClientWidth := 1
```
```pascal title:"Iteracja po liście"
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
```
```pascal title:"Timer"
var
  timer: TTimer;

procedure timerEvent(Sender: TObject);
begin
  timer.Enabled := False;
end;
if (timer = nil) then
begin
  timer := TTimer.Create(Self);
  //timer.Name := 'tmrShow';
  //tmrShow.Enabled := False;
  timer.Interval := 1000;
  timer.OnTimer := @timerEvent;
end;
```