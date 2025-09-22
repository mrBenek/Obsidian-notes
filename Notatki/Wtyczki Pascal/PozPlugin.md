```pascal title:"Dodawanie pozycji"
procedure DodajPozycje(const AIdKartoteka: string; const ean: string);
var
  wListaPoz: TListDanePoz;
  wDanePoz: TDanePozImport;

begin
  wListaPoz := TListDanePoz.MCreate;
  try
    wDanePoz := TDanePozImport.Create;
    wDanePoz.Id_Kartoteka := StrToInt(AIdKartoteka);
    wDanePoz.Ilosc := 1;
    wListaPoz.AddDanePozImport(wDanePoz);
    frm.DodajPozPlugin(wListaPoz);

    if not KlasyfTowSprPozZamEan(IntToStr(wDanePoz.Id_Poz), ean) then
      frm.UsunPozPluginId(wDanePoz.Id_Poz);

  finally
    wListaPoz.Free;
  end;
end;
```

```pascal title:"Podział pozycji"
procedure PodzielPozycje;
var
  vIlosc: Currency;
  vIdPozOld: Integer;
  vIdPozNew: Integer;
  vTransaction: TstTransaction;
  wListaPoz: TListDanePoz;
  wDanePozOld: TDanePozImport;
  wDanePozNew: TDanePozImport;
  vSql: string;
begin
  maxIlosc := 0;

  wDanePozOld := TDanePozImport.Create;
  try
    if fDokGmEdv2.PobierzPozPluginId(fDokGmEdv2.IdPozycjiAkt, wDanePozOld) then
      maxIlosc := wDanePozOld.Ilosc;
  finally
    wDanePozOld.Free;
  end;

  if (maxIlosc = 0) then Exit;

  vIlosc := 0;

  fIlosc := TForm.Create(Application);
  try
    PrzygotujOknoIlosc;
    if (fIlosc.ShowModal = 1) then
      vIlosc := StrStToCurr(Trim(string(qeIlosc.GetValue)));
  finally
    fIlosc.Free;
  end;

  if (vIlosc = 0) then Exit;

  vIdPozOld := fDokGmEdv2.IdPozycjiAkt;
  vIdPozNew := 0;
  vTransaction := TstTransaction(TstQuery(fDokGmEdv2.DS_PozDok.DataSet).Transaction);

  wListaPoz := TListDanePoz.MCreate;
  try
    wDanePozOld := TDanePozImport.Create;
    if fDokGmEdv2.PobierzPozPluginId(vIdPozOld, wDanePozOld) then
    begin
      wDanePozNew := TDanePozImport.Create;
      if wDanePozNew.AssignFromId(vIdPozOld, vTransaction) then
      begin
        wDanePozOld.Ilosc := wDanePozOld.Ilosc - vIlosc;
        wListaPoz.AddDanePozImport(wDanePozOld);
        if fDokGmEdv2.EdytujPozPlugin(wListaPoz) then
        begin
          wDanePozNew.Ilosc := vIlosc;
          wDanePozNew.WymusCeneIBonifPrzyDod := True;
          wListaPoz.Clear;
          wListaPoz.AddDanePozImport(wDanePozNew);
          if fDokGmEdv2.DodajPozPlugin(wListaPoz) then
            vIdPozNew := wDanePozNew.Id_Poz;
        end else
          wDanePozNew.Free;
      end else
      begin
        wDanePozNew.Free;
        wDanePozOld.Free;
      end;
    end else
      wDanePozOld.Free;
  finally
    wListaPoz.Free;
  end;

  if (vIdPozNew = 0) then Exit;

  vSql := 'execute procedure XXX_AKT_POZ_PO_PODZ('
    + IntToStr(fDokGmEdv2.PluginID_Nagl) + ', '
    + IntToStr(vIdPozOld) + ', '
    + IntToStr(vIdPozNew) + ')';

  if (ExecuteSQL(vSql, 0) <> 1) then
    Inf('Błąd przy aktualizacji parametrów dodanej pozycji.',100);

  TstQuery(fDokGmEdv2.DS_PozDok.DataSet).Close;
  TstQuery(fDokGmEdv2.DS_PozDok.DataSet).Open('');
  if (vIdPozNew > 0) then
    TstQuery(fDokGmEdv2.DS_PozDok.DataSet).Locate('id_poz', vIdPozNew, []);
end;

procedure bbPodzielPozycje_OnClick(Sender: TObject);
begin
  if ((fDokGmEdv2.KlFun = F3) or (fDokGmEdv2.KlFun = F5))
    and OdpowiednieDokumentyPodzielPozycje
    and OdpowiednieGrupyUzytkownikow then
      PodzielPozycje;
end;
```
