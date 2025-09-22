```pascal titile:"Aktualizacja cechy pozycji i nagla"
function AktualizujCechePoz(const ACecha: string; const AIdPoz: Integer; AIdCechaDokk, AWartosc: string): Boolean;
var
  vSql: string;
begin
  Result := True;

  vSql := 'update wystcechpoz'
       + ' set wartosc = ' + QuotedStr(AWartosc)
       + ' where id_poz = ' + IntToStr(AIdPoz)
       + ' and id_cechadokk = ' + AIdCechaDokk;
         
  if (ExecuteSQL(vSql, 0) <> 1) then
  begin
    Result := False;
    DodajDoLogu('Błąd przy aktualizacji cechy.' + ACecha);
  end;
end;

function AktualizujCechePoz(const ATransaction: TstTransaction; const ACecha: string; const AIdPoz: Integer; AIdCechaDokk, AWartosc: string): Boolean;
var
  vSql: string;
begin
  Result := True;

  DodajDoLogu('Aktualizacja cechy: ' + ACecha + ', Wartość: ' + AWartosc);

  vSql := 'update wystcechpoz'
       + ' set wartosc = ' + QuotedStr(AWartosc)
       + ' where id_poz = ' + IntToStr(AIdPoz)
       + ' and id_cechadokk = ' + AIdCechaDokk;
  try
    ATransaction.WykonajSQL(vSql);
  except
    Result := False;
    DodajDoLogu('Błąd przy aktualizacji cechy.');
  end;
end;

function AktualizujCecheDok(const ACecha: string; const AIdNagl: Integer; AIdCechaDokk, AWartosc: string): Boolean;
var
  vSql: string;
begin
  Result := True;

  DodajDoLogu('Aktualizacja cechy: ' + ACecha + ', Wartość: ' + AWartosc);

  vSql := 'update wystcechnagl'
       + ' set wartosc = ' + QuotedStr(AWartosc)
       + ' where id_nagl = ' + IntToStr(AIdNagl)
       + ' and id_cechadokk = ' + AIdCechaDokk;

  if (ExecuteSQL(vSql, 0) <> 1) then
  begin
    Result := False;
    DodajDoLogu('Błąd przy aktualizacji cechy.');
  end;
end;


if not AktualizujCecheDok('EDI_ORDERNUMBER', Result, cIdCechaDokk_Nagl_EDI_ORDERNUMBER, AOrderNumber) then Exit;

if not AktualizujCechePoz(vTransaction, 'EDI_LINENUMBER', vIdPoz, cIdCechaDokk_Poz_EDI_LINENUMBER, ALineNumber) then Exit;

if not AktualizujCechePoz('EDI_LINENUMBER', vIdPoz, cIdCechaDokk_Poz_EDI_LINENUMBER, ALineNumber) then Exit;
```