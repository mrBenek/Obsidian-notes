Jeżeli chcemy wziąć tylko jedną wartość z tranzakcji to wykorzystujemy Transaction.PodajzQuery
Jeżeli chcemy robić jakiś update/insert... to Transaction.WykonajSQL
Jeżeli checmy wziąć kilka column/wierszy z tranzakcji to musimy to zrobić tak:

```pascal title:"Biblioteki"
{$ADDTYPE TFIBDatabase}
{$ADDTYPE TFIBTransaction}
{$ADDTYPE TstTransaction}
{$ADDTYPE TstQuery}
```
```pascal title:"Podpięcie do transakcji"
Transaction
vField: TField;

fQueryPoz := TstQuery(fDokGmEd.DBG_PozDok.DataSource.DataSet);
if not fQueryPoz.Active then Exit;

vTransaction := TstTransaction(fQueryPoz.Transaction);
if (vTransaction = nil) then Exit;

vField := vTransaction.PodajzQuery('count(kv.id_waluta)',
  'nagl n'
  + ' join kontrahdefkbi_view kv on kv.id_kontrah = n.id_kontrah and kv.id_waluta = n.id_waluta',
  'n.id_nagl = ' + IntToStr(fDokGmEd.PluginID_Nagl),
  '', '', '', '', '', '');

if (vField.AsInteger = 0) then
begin
  Result := False;
  Inf('Brak konta bankowego dla wybranej waluty. Dodaj konto bankowe dla kontrahenta.', 100);
end;
```
```pascal title:"Tworzenie nowej zakładki"
vTransaction: TstTransaction;
vTransaction := TstTransaction.Create(nil);
vTransaction.DefaultDatabase := TstQuery(DataSource.DataSet).Transaction.DefaultDatabase;
vTransaction.stBeginTrans('');
vTransaction.WykonajSQL(vSql);
if (vLog <> '') then
begin
  if vTransaction.InTransaction then
    vTransaction.stRollback('');
end
else
  vTransaction.stCommit('');

  if (PytTN('Uwaga ! ' + LN +
            'Stawka może być już wykorzystywana do wyliczenia kosztów.' + LN +
            'Zapisanie zmian spowoduje niezgodność stawki z wyliczeniami w technologii i zleceniach produkcyjnych.' + LN +
            'Czy na pewno chcesz kontynuować?', 100)) then
```
```pascal title:"Branie wyniku wykonywania procedury"
{$ADDTYPE TFIBDatabase}
{$ADDTYPE TFIBTransaction}
{$ADDTYPE TstTransaction}
{$ADDTYPE TstQuery}
{$ADDTYPE TstSQL}
procedure GetIdPoz(IdNagl: string; IdMag: string; IdKartoteka: string; Ilosc: string);
var
  vSql: string;
  vTransaction: TstTransaction;
  vDataSource: TDataSource;
  vIdPoz: Integer;
  stSql: TstSQL;
begin
  vTransaction := TstTransaction.Create(nil);
  try
    vDataSource := OpenQuerySQL('select 1 from rdb$database', 0);
    try
      vTransaction.DefaultDatabase := TstQuery(vDataSource.DataSet).Transaction.DefaultDatabase;
    finally
      CloseQuerySQL(vDataSource);
    end;

    vSQL := 'select AID_POZ from POZ_ADD(' + IdNagl + ',' + IdMag + ',' + IdKartoteka + ',' + Ilosc + ', null, 0, 0, null,' +
            '0, null, null, null, null, null, null, null)';

    vTransaction.stBeginTrans('');
    try
      stSql := TstSQL.Create(nil);
      try
        stSql.Transaction := TFIBTransaction(vTransaction);
        stSql.SQL.Text := vSql;

        stSql.Open('');
        vIdPoz := stSql.FieldByName('AID_POZ').AsInteger;
      finally
        stSql.Free;
      end;
      vTransaction.stCommit('');
    finally
      if vTransaction.InTransaction then
        vTransaction.stRollback('');
    end;
  finally
    vTransaction.Free;
  end;
end;
```
```pascal title:"Dodaj wartości do combo - branie całego dataset"
procedure DodajWartosciSqlDoCombo(const AComboBox: TstComboBox; const ASql: string; const APierwszyPusty: Boolean);
var
  vQuery: TstQuery;
  vDataSource: TDataSource;
begin
  if APierwszyPusty then
    AComboBox.AddInt('', 0);

  vQuery := TstQuery.Create(Self);
  try
    vQuery.Database := Transaction.DefaultDatabase;
    vQuery.Transaction := TFIBTransaction(Transaction);

    vQuery.SQL.Text := ASql;

    vQuery.stPrepare('');
    vQuery.stOpen('');

    vDataSource := TDataSource.Create(Self);
    try
      vDataSource.DataSet := TDataSet(vQuery);
      vDataSource.DataSet.First;
      while not vDataSource.DataSet.Eof do
      begin
        AComboBox.AddInt(vDataSource.DataSet.FieldByName('WARTOSC').AsString, vDataSource.DataSet.FieldByName('ID').AsInteger);
        vDataSource.DataSet.Next;
      end;
    finally
      vDataSource.Free;
    end;
  finally
    vQuery.Free;
  end;

  if (AComboBox.Items.Count > 0) then
    AComboBox.ItemIndex := 0;
end;
```
```pascal title:"Branie ostatniego błędu z tranzakcji"
if (GetLastAPIError <> '') then
begin
  UwagaOk('Nie można zaktualizować CECHY kontrahenta.' + cLineBreak +
		  'APIError: ' + GetLastAPIError + cLineBreak +
		  'SQL: ' + cLineBreak + vSQL);
end;
```