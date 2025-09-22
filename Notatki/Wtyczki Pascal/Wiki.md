```pascal title:Stałe
const
  LN = #13#10;
  TAB  = #9;
  NULL = #0;
```
```pascal fold title:"Pusta wtyczka"
// TfOperacjaEd
{$ADDTYPE TfOperacjaEd}

var
  frm : TfOperacjaEd;

procedure TworzeniePowiazania(Sender : TObject);
begin
end;

begin
  if (Self is TfOperacjaEd) then
  begin
    if (frm = nil) then
    begin
      frm := Self as TfOperacjaEd;
    end;
    if (frm <> nil) then
    begin
      inf300(IntToStr(frm.WindowId));
      PluginsAddAction(Self, 'Utwórz powiązanie', 'Attach', @TworzeniePowiazania); //Dodanie przycisku
    end;
  end;
end.
```
```pascal title:"Window ID"
function ParentWindowId: Integer; //wzięcie windowID
var
  i: Integer;

begin
  Result := -1;
  for i := 0 to Screen.FormCount - 1 do
    if (Screen.Forms[i] is TfRozliczeniaNieroz) then
    begin
      Result := TfRozliczeniaNieroz(Screen.Forms[i]).WindowId;
      Break;
    end;
end;
if (ParentWindowId = 5810) //wywolanie
```
```pascal title:"Wywołanie procedury wtyczkowej"
vDataInList : TStringList;

vDataInList := nil;
vDataInList := TStringList.Create;
try
  vDataInList.Values['ID_NAGL'] := IntToStr(vID_NAGL_PW_Po);

  if ExecutePlugin('DodajPrzygotowaniePWWKIT', vDataInList, nil) then
      infSRWP('Wykoanno procedurę po zamknięciu PW: ' + 'DodajPrzygotowaniePWWKIT');
finally
    CloseQuerySQL(vDS);
    if Assigned(vDataInList) then vDataInList.Free;
    vDataInList := nil;
end;
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

procedure DodajKolumny; //czasami nie potrzebne jest to tworzenie kolumn, to zalezy od okna
var
  vDefPola: TdefPola;
  vColumn: TColumn;

begin
  if (Pos(UpperCase('xxx_kart_w_zam_do_dost'), UpperCase(fDokZamowieniaDoDost.QueryMain.SqlFrom)) = 0) then
    fDokZamowieniaDoDost.QueryMain.SqlFrom := fDokZamowieniaDoDost.QueryMain.SqlFrom
      + ' left join xxx_kart_w_zam_do_dost(n.id_nagl, ' + IntToStr(IdKartoteka) + ') xkz on (1=1)';

  if (Pos(UpperCase('xkz.kart_w_zam_do_dost'), UpperCase(fDokZamowieniaDoDost.QueryMain.SqlSelect)) = 0) then
    fDokZamowieniaDoDost.QueryMain.SqlSelect := fDokZamowieniaDoDost.QueryMain.SqlSelect
      + ', xkz.kart_w_zam_do_dost';

  if (fDokZamowieniaDoDost.QueryMain.FindFieldFromSqlName('kart_w_zam_do_dost') = nil) then
  begin
    vDefPola := TdefPola.Create;
    vDefPola.FieldName := 'kart_w_zam_do_dost';
    vDefPola.FieldType := ftSmallint;
    vDefPola.rodzajWysw := rwTakNie;
    vDefPola.OpisPola := 'Kartoteka w dokumencie';
    vDefPola.visible := True;
    vDefPola.DisplayLabel := 'Kartoteka w dokumencie';
    vDefPola.required := True;
    vDefPola.rodzajPola := rpData;
    fDokZamowieniaDoDost.QueryMain.AddField(vDefPola, 'xkz');
  end;

  if (TstQuery(fSekDokEd.DS_DodNaglSekDok.DataSet).FindFieldFromSqlName('MIEJSCOWOSC') = nil) then
    begin
      vDefPola := TdefPola.Create;
      vDefPola.FieldName := 'MIEJSCOWOSC';
      vDefPola.FieldType := ftString;
      vDefPola.rodzajWysw := rwInne;
      vDefPola.OpisPola := 'Miejscowość';
      vDefPola.visible := True;
      vDefPola.DisplayLabel := 'Miejscowość';
      vDefPola.required := True;
      vDefPola.rodzajPola := rpData;
      TstQuery(fSekDokEd.DS_DodNaglSekDok.DataSet).AddField(vDefPola, 'DK');
    end;

  vColumn := fDokZamowieniaDoDost.DBGmain.FindColumn('kart_w_zam_do_dost');
  if (vColumn = nil) then
  begin
    vColumn := TColumn.Create(fDokZamowieniaDoDost.DBGmain.Columns);
    vColumn.FieldName := 'kart_w_zam_do_dost';
    vColumn.Width := 150;
  end;
end;
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
```pascal title:Akcje
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

AD_ZamknijDokOnExecute := fDokGmEd0.AD_ZamknijDok.OnExecute;
	fDokGmEd0.AD_ZamknijDok.OnExecute := @AD_ZamknijDok_OnExecute
```
```pascal title:"Obsługa plików"
//zapis
WriteComponentResFile('c:\temp\plik.dfm', Self);

//Wzięcie nazwy pliku  
ExtractFileName(XMLFilePaths);
```
```pascal title:"Przyciskanie przycisków"
PostMessage(fMat.DBGmain.Handle, WM_KEYDOWN, 27, 0); //escape
PostMessage(fMat.DBGmain.Handle, WM_KEYDOWN, 13, 0); //enter
```
```pascal title:"Słowniki"
vIdMagazyn := Slownik('MAGAZYN'); - otwiera okno z wyborem magazynu
```
```pascal title:"Wyświetl DataSet"
procedure WyswietlDataSet(dataset: TDataSet);
var
  i: integer;
  s: string;
begin
  s:='';
  for i:=0 to dataset.fieldcount-1 do
    begin
      with dataset do
        begin
          s:=s+Fields[i].fieldname+ #9 + FIELDBYNAME(Fields[i].fieldname).ASSTRING+#13#10;
        end;
      end;
  inf300(s);
end;
```
```pascal title:"Iterowanie po dataset"
vColumnsDS.DataSet.FIRST;
while not vColumnsDS.DataSet.EOF do
begin
  if ((vColumnsDS.DataSet.FIELDBYNAME('windowid').ASSTRING = '') or
      (vColumnsDS.DataSet.FIELDBYNAME('windowid').ASINTEGER = frm.WindowId) ) then
  begin
  end;
  vColumnsDS.DataSet.NEXT;
end;
```
```pascal title:"Branie wszystkich zaznaczonych rekordów"
Branie wszystkich zaznaczonych rekordow

procedure UruchomProcedure(sIdNagl1: string; sIdNagl2: string);
var
   sSql: string;
   iRes: integer;
begin
   sSql := ' insert into NAGLSPRZ (RODZAJSPRZNAGL, ID_NAGLCO, ID_NAGLCZYM, ID_POWKOREKTY) ' + LN +
           ' values (45, ' + sIdNagl1 + ', ' + sIdNagl2 + ', null); ';
   if (ExecuteSQL(vSql, 0) <> 1) then
   begin
     Inf('Nie utworzono pozycji urządzenia zewnętrznego.', 100);
     Exit;
   end;
end;

procedure UruchomProcedure(idZlecenie: string; procName: string);
var
   sql: string;

begin
   sql := 'execute procedure ' + procName + '(' + idZlecenie + ')';
   //inf300(sql);
   if (ExecuteSQL(sql, 0) <> 1) then
   begin
     Inf('Błąd wywołania procedury. sql: ' + sql, 100);
     Exit;
   end;
end;

procedure BookingLOT(procName: string);
var
  idZlecenie: string;

  i: integer;
  vLista: string;
  vIdZlecenie: string;
  vListaIdZlecenie: TStringList;
begin
  vIdZlecenie := inttostr(frm.QueryMain.FieldByName('ID_ZLECENIE').AsInteger);
  if (frm.QueryMain.MarkCount > 0) then
    vLista := frm.QueryMain.GetMarkedRows
  else
    vLista := '(' + vIdZlecenie + ')';

  vLista := Trim(Copy(vLista, 2, Length(vLista) - 2));
  Zastap(',', #13 + #10, vLista);
  vListaIdZlecenie := TStringList.Create;
  try
    vListaIdZlecenie.Text := vLista;

    for i := 0 to vListaIdZlecenie.Count - 1 do
    begin
      idZlecenie := vListaIdZlecenie[i];

      UruchomProcedure(idZlecenie, procName);
    end;
  finally
    vListaIdZlecenie.Free;
  end;
end;

procedure TworzeniePowiazania(seder : TObject);
var
  idNagl1: string;
  idNagl2: string;

  i: integer;
  vLista: string;
  vIdNagl: string;
  vListaIdNagl: TStringList;
begin
  //przygotowanie vListaIdNagl z nagłówkami zaznaczonych pozycji
  vIdNagl := inttostr(frm.QueryMain.FieldByName('Id_Nagl').AsInteger);
  if (frm.QueryMain.MarkCount > 0) then
    vLista := frm.QueryMain.GetMarkedRows
  else
    vLista := '(' + vIdNagl + ')';

  vLista := Trim(Copy(vLista, 2, Length(vLista) - 2));
  Zastap(',', #13 + #10, vLista);
  vListaIdNagl := TStringList.Create;
  try
    vListaIdNagl.Text := vLista;
    //odczyt dokumentu do powiązania
    idNagl1 := inttostr(Slownik('nagl'));
    inff('idNagl1=' + idNagl1, -1);
    //zapis do bazy
    for i := 0 to vListaIdNagl.Count - 1 do
    begin
      idNagl2 := vListaIdNagl[i];

      inff('idNagl2=' + idNagl2, -1);
      UruchomProcedure(idNagl1, idNagl2);
    end;
  finally
    vListaIdNagl.Free;
  end;
end;
```
```pascal title:"Zczytywanie wartości z pola"
{$ADDTYPE TFCzytajCos}
fCzytajCos: TfCzytajCos;
  vWartosc: string;
  vResult: Boolean;
fCzytajCos := TfCzytajCos.Create(nil);
try
  vResult := fCzytajCos.CzytajCos('Indeks kartoteki', vWartosc, 40, False, True);
finally
  fCzytajCos.Free;
end;
```
```pascal title:"Wyświetlenie zapytanie WHERE"
var
   x: TstQuery;

x := TstQuery(frm.DS_Grupy.DataSet);

inf300('SqlWhereUser: ' + x.SqlWhereUser + #13 +
       'SqlWhereProg: ' + x.SqlWhereProg +  #13 +
       'SqlWhereProg2: ' + x.SqlWhereProg2 +  #13 +
       'SqlWhereProg3: ' + x.SqlWhereProg3 +  #13 +
       'SqlWhereMscWystDok: ' + x.SqlWhereMscWystDok +  #13 +
       'SqlWhereJoinMaker: ' + x.SqlWhereJoinMaker +  #13 +
       'SqlWhereProgColActive: ' + x.SqlWhereProgColActive +  #13 +
       'SqlWhereFilter: ' + x.SqlWhereFilter + #13 +
       'SqlWhereSearch: ' + x.SqlWhereSearch);

inf300('SqlWhereUser: ' + frm.QueryMain.SqlWhereUser + #13 +
       'SqlWhereProg: ' + frm.QueryMain.SqlWhereProg +  #13 +
       'SqlWhereProg2: ' + frm.QueryMain.SqlWhereProg2 +  #13 +
       'SqlWhereProg3: ' + frm.QueryMain.SqlWhereProg3 +  #13 +
       'SqlWhereMscWystDok: ' + frm.QueryMain.SqlWhereMscWystDok +  #13 +
       'SqlWhereJoinMaker: ' + frm.QueryMain.SqlWhereJoinMaker +  #13 +
       'SqlWhereProgColActive: ' + frm.QueryMain.SqlWhereProgColActive +  #13 +
       'SqlWhereFilter: ' + frm.QueryMain.SqlWhereFilter + #13 +
       'SqlWhereSearch: ' + frm.QueryMain.SqlWhereSearch);
```
```pascal title:"Zakładki"
frm.PC_Zakladki.OnChange := @ZakladkiOnChange;
procedure ZakladkiOnChange(Sender : TObject);
begin
  if (frm.PC_Zakladki.ActivePageIndex = 0) then
  begin

  end;

  frm.PC_ZakladkiChange(Sender);
end;
```
```pascal title:"Jeśli coś na zakładce nie che mi się zaktualizować, to wystarczy przejść na tę zakładkę - takie coś jest we wtyczce wtyczka_TfKontrahEd_20221107_132855.ROPS"
procedure ZmienZakladke(aNaTabSheet: TTabSheet);
begin
 frm.PC_Zakladki.ActivePage := aNaTabSheet;  //Zmieniamy zakladke by aktywowac DataSet
 frm.PC_ZakladkiChange(frm.PC_Zakladki);         //Musimy też wywołać zdarzenie zmiany na PageControl
end;

procedure DodajGrupyKart;
var
  Transaction: TstTransaction;
  vSql: string;

begin
   ZmienZakladke(frm.TS_Cechy);
   ZmienZakladke(frm.TS_Formularze);
   ZmienZakladke(frm.TS_DokumentyProdukcyjne);
   ZmienZakladke(frm.TS_Podstawowe);

   Transaction := TstTransaction(TstQuery(frm.DS_FormularzeTech.DataSet).Transaction);

   vSql := 'INSERT INTO DRUK_RAPORTRB (ID_DRUK_RAPORTRB, ID_TECHNOLOGIA, ID_RAPORTRB, ID_DRUK_KONF, AKTYWNY, DOTYCZY) ' +
           'VALUES (GEN_ID(GEN_DRUK_RAPORTRB,1), ' + TstQuery(frm.DS_Technologia.DataSet).FieldByName('id_technologia').AsString + ', 10002, NULL, 1, 0)';
   try
     Transaction.WykonajSQL(vSql);
   except
     Exit;
   end;
   inf300(vsql);
end;
```
```pascal title:"Obsługa pola czasowego"
procedure AktualizujCzas(first: boolean);
var
  vSql: String;

begin
  czas := 0;
  if (first) and (Pos('Zakończenie', frm.E_OperacjaSl.Text) >= 1) then
  begin
    vSql := 'select time ''00:00:00'' + SUM(datediff(second, timestamp ''1899-12-30 00:00:00'', ZT.CZAS))' +
            ' from ZASOBTECHCRP ZT' +
            ' left join OPERACJATECH O on O.ID_OPERACJATECH = ZT.ID_OPERACJATECH' +
            ' where (O.ID_TECHNOLOGIA = ' + IntToStr(frm.DmEdycja.FID_Technologia) + ') and' +
            '      (O.ID_OPERACJASL = 10393)';

    frm.CE_Czas.Text := GetFromQuerySQL(vSql, 0);
    czas := StrToTime(frm.CE_Czas.Text);
  end
  else
  begin
    frm.DS_ZasobyTechCrp.DataSet.FIRST;
    while not frm.DS_ZasobyTechCrp.DataSet.EOF do
    begin
      czas := czas + frm.DS_ZasobyTechCrp.DataSet.FIELDBYNAME('CZAS').ASFLOAT;
      frm.DS_ZasobyTechCrp.DataSet.NEXT;
    end;
    frm.CE_Czas.Text := TimeToStr(czas);
  end;

  frm.DmEdycja.ZmianaCzasu(czas);
  frm.DmEdycja.AktualizujCzasOperacji;
end;
```
```pascal title:"Przyciski edycji"
palec.KlFun = F5
```
```pascal title:"Ustawinie kolejności komuln na zakładkach na sekdoku"
fSekDokEd.PC_Main.OnChange := @PC_Main_OnChange;
procedure PC_Main_OnChange(Sender: TObject);
begin
  fSekDokEd.PC_MainChange(Sender);
  fSekDokEd.DBG_DodNaglSekDok.FindColumn('opisdefdok').Index := 1;
end;
```
```pascal title:"StrToCurr"
function CzyMoznaWystawicDokDlaPoz(const AListaPoz: string): Boolean;
var
  vSql: string;
  vIlosc: Currency;

begin
  Result := True;

  vSql := 'select coalesce(sum(r.ilrez), 0)'
       + ' from poz p'
       + ' join pozzr pz on pz.id_poz = p.id_poz'
       + ' join rez r on r.id_pozzr = pz.id_pozzr'
       + ' where p.id_poz in ' + AListaPoz
       + ' and r.status = 0'
       + ' and r.anulowany = 0';

  vIlosc := StrStToCurr(Trim(GetFromQuerySQL(vSql, 0)));
  frm.QE_CenaB.SetValue(vCenaZak);
  if (vIlosc > 0) then Exit;
end;
```
```pascal title:"Progres"
progress : TfPostepOp //zmienna globalna
progress := TfPostepOp.Create(Self);
try
    progress.Title := 'Trwa akceptacja - ilość dokumentów : ' + IntToStr(listaSekDok.Count);
    progress.Caption := 'Trwa akceptacja - ilość dokumentów : ' + IntToStr(listaSekDok.Count);
    progress.Max1 := listaSekDok.Count;
    progress.ShowWindow;
    progress.Position1 := I + 1;
    finally
    SetGlobalData('TW_AUTOAKCEPTACJA','0');
    progress.Hide;
    progress.Free;
    progress = nil;
```
```pascal title:"Ustawianie wartości na dataset"
TstQuery(frm.DS_Cechy.DataSet).FieldByName('WARTOSC').NEWVALUE := '123';
```
```pascal title:"Combobox"
procedure UstawWartoscCombo(const AComboBox: TstComboBox; const AId: Integer);
begin
  TstComboBox(AComboBox).ItemIndex := TstStringList(TstComboBox(AComboBox).Items).GetPozFromInt(AId);
end;


vItemIndex := TstXComboBox(frm.CB_SposPl).Items.IndexOf(cSposPlatnosci_DlaLimitPrzedplata);
if (vItemIndex >= 0) then
  frm.CB_SposPl.ItemIndex := vItemIndex;
```

```pascal title:"Aktualnie zaznaczone okno (Zakładki czy główne)"
frm.ActiveControl.Parent.Name
```

```pascal title:"Wykonywanie dla zaznaczonych elementów w pętli"
procedure PrzeliczMeldunki(Sender: TObject);
var
  idZlecenie: string;
  i: integer;
  vLista: string;
  vSql: string;
  vIdZlecenie: string;
  vListaIdZlecenie: TStringList;
  wService: TServiceZpPlugins;
  vDataSource: TDataSource;

begin
  vIdZlecenie := inttostr(frm.QueryMain.FieldByName('ID_ZLECENIE').AsInteger);
  if (frm.QueryMain.MarkCount > 0) then
    vLista := frm.QueryMain.GetMarkedRows
  else
    vLista := '(' + vIdZlecenie + ')';

  vLista := Trim(Copy(vLista, 2, Length(vLista) - 2));
  Zastap(',', #13 + #10, vLista);
  vListaIdZlecenie := TStringList.Create;
  wService := TServiceZpPlugins.Create(nil, nil);
  try
    vListaIdZlecenie.Text := vLista;

    for i := 0 to vListaIdZlecenie.Count - 1 do
    begin
      idZlecenie := vListaIdZlecenie[i];

      vSql := 'SELECT ID_MELDUNEK FROM XXX_ZP_PRZELICZ_MELDUNKI(' + idZlecenie + ')';
      vDataSource := OpenQuerySQL(vSQL, 0);
      if vDataSource <> nil then
      begin
        try
          vDataSource.DataSet.First;
          while not vDataSource.DataSet.Eof do
          begin
            wService.PrzeszacujMeldunek(vDataSource.DataSet.FIELDBYNAME('ID_MELDUNEK').ASINTEGER);
            vDataSource.DataSet.Next;
          end;
        finally
          CloseQuerySQL(vDataSource);
        end;
      end;
    end;
  finally
    wService.Free;
    vListaIdZlecenie.Free;
  end;
end;
```

```pascal title:"Pobieranie wartości z procedury i potem ich analiza"
procedure ProcPrzeliczMeldunki(idZlecenie: string);
var
  vSql: string;
  vTransaction: TstTransaction;
  vDataSource: TDataSource;
  stSql: TstSQL;
  wService: TServiceZpPlugins;
  vListaIdMeldunek: TIntegerList;
  i: integer;
  vIdMeldunek: integer;

begin
  vTransaction := TstTransaction.Create(nil);
  vListaIdMeldunek := TIntegerList.Create;
  wService := TServiceZpPlugins.Create(nil, nil);
  try
    vDataSource := OpenQuerySQL('select 1 from rdb$database', 0);
    try
      vTransaction.DefaultDatabase := TstQuery(vDataSource.DataSet).Transaction.DefaultDatabase;
    finally
      CloseQuerySQL(vDataSource);
    end;

    vSQL := 'SELECT ID_MELDUNEK FROM XXX_ZP_PRZELICZ_MELDUNKI(' + idZlecenie + ')';

    vTransaction.stBeginTrans('');
    try
      stSql := TstSQL.Create(nil);
      try
        stSql.Transaction := TFIBTransaction(vTransaction);
        stSql.SQL.Text := vSql;

        stSql.Open('');
        while not stSql.Eof do
        begin
          vListaIdMeldunek.Add(stSql.FieldByName('ID_MELDUNEK').AsInteger);
          stSql.Next;
        end;
      finally
        stSql.Free;
      end;
      vTransaction.stCommit('');
    finally
      if vTransaction.InTransaction then
        vTransaction.stRollback('');
    end;

    if (vListaIdMeldunek.Count > 0) then
    begin
      for i := 0 to vListaIdMeldunek.Count - 1 do
      begin
        vIdMeldunek := vListaIdMeldunek.GetValue(i);
        //inf300(inttostr(vidmeldunek));
        wService.PrzeszacujMeldunek(vIdMeldunek);
        wService.RozliczMeldunek(vIdMeldunek);
      end;
      inf300('Meldunki zostały przeliczone');
      frm.DS_main.DataSet.REFRESH;
    end;
  finally
    vListaIdMeldunek.Free;
    vTransaction.Free;
    wService.Free;
  end;
end;
```

```pascal title:"Tworzenie tablicy obiektów"
//nie musimy zwalniać obiektu jak w przypadku list
type TKurier = record
    id:string;
    nazwa:string;
    typ_etykiety:string;
    typ_uslugi:string;
    typ_paczki:string;
    platnik:string;
    link:string;
end;

var
  kurierzy: array of TKurier;
  
function szukajWLisciePoId(wartosc:String):TKurier;
var
  i: Integer;
begin
    for i:=0 to length(modules) - 1 do
    begin
        if(kurierzy[i].id=wartosc) then
        begin
             Result:=kurierzy[i];
             break;
        end;
    end ;
end;

function konfigurujModul(nazwa:string; id:string) :TKurier;
var
module :TKurier;
begin
  module.id:=id;
  module.nazwa:=nazwa;

  case nazwa of
   'Kurier GLS': begin
    module.typ_etykiety:='roll_160x100_pdf';
    module.typ_uslugi:='0';
    module.typ_paczki:='';
    module.platnik:='0';
    ... //dużo tego było...
    
  Result:=module;
end;    


procedure initKurierzy();
var
  data : TDataSource;
  i, ile:integer;
  nazwa, id:string;
begin
    ile:=StrToInt(GetFromQuerySQL('select count(*) from SPOSDOSTAWY where aktywny=1 and nazwasposdostawy containing ''kurier''', 0));
    SetLength(modules, ile);
    data := OpenQuerySQL('select ID_SPOSDOSTAWY, nazwasposdostawy, aktywny from SPOSDOSTAWY where aktywny=1 and nazwasposdostawy containing ''kurier''',0);
    data.DataSet.First;
    i:=0;
    while not data.DataSet.EOF do
    begin
         id:=data.DataSet.FIELDBYNAME('ID_SPOSDOSTAWY').ASSTRING;
         nazwa :=data.DataSet.FIELDBYNAME('nazwasposdostawy').ASSTRING;
         modules[i]:= konfigurujModul(nazwa, id);

        inc(i);
        data.DataSet.Next;
    end;
    CloseQuerySQL(data);
end;
```

```pascal title:"Liczba rekordów w DataSet"
vDataSet.RecordCount
```