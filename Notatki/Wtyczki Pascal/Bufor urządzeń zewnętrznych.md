```pascal

var
  IdUzytkownik: Integer;

function PobIdUzytk : Boolean;
var
   vId : string;
begin
   result := false;
   IdUzytkownik := -1;
   vId := Trim(GetFromQuerySQL('SELECT ID_UZYTKOWNIK FROM uzytkownik WHERE login = ' + #39 + GetUser + #39, 1));
   if length(vId) > 0 then
   begin
      IdUzytkownik := StrToInt(vId);
      result := true;
   end else
      inf('Błąd przy określaniu użytkownika.', 100);
end;

  vSql := 'select U.oznnrwydruzyt FROM UZYTKOWNIK U WHERE U.id_uzytkownik = ' + IntToStr(IdUzytkownik);
vNrUzytk := Trim(GetFromQuerySQL(vSql, 0));

vUwagi := GetUser + '_' + DateTimeToStr(Now);

vSql := 'execute procedure URZZEWNAGL_ADD('
  + '1, '
  + QuotedStr(cKodUrzZew_ZOO_UKC) + ', '
  + QuotedStr(vNrUzytk) + ', '
  + 'null, ' + IntToStr(vIdKontrah) + ', null, 2, '
  + 'current_date, ' + QuotedStr('ZA') + ', '
  + QuotedStr(cPrefiksNrDok_ZOO_UKC) + '||GEN_ID(XXX_GEN_ZOO_UKC, 1), '
  + 'null, null, null, '
  + '0, '
  + 'null, null, null, null, ' + QuotedStr(vUwagi) + ')';

if (ExecuteSQL(vSql, 0) = 1) then
begin
  vSql := 'select max(u.id_urzzewnagl) from urzzewnagl u where u.odb_uwagi = ' + QuotedStr(vUwagi);

  vIdUrzZewNagl := Trim(GetFromQuerySQL(vSql, 0));

  vSql := 'update urzzewnagl set odb_uwagi = '''' where odb_uwagi = ' + QuotedStr(vUwagi);
  ExecuteSQL(vSql, 0);
  if (vIdUrzZewNagl = '') then
  begin
    Inf('Nie pobrano nowo utworzonego nagłówka.', 100);
    Exit;
  end;
end else
begin
  Inf('Nie utworzono nagłówka urządzenia zewnętrznego.', 100);
  Exit;
end;

for vRow := 4 to Sheet.UsedRange.EntireRow.Count do
begin
  vIndeks := Sheet.Cells[vRow, 2].Text;
  vIlosc := Trim(Sheet.Cells[vRow, 6].Text);

  vUzupelnionaIlosc := False;
  if (vIlosc <> '') then
    if (StrStToCurr(vIlosc) > 0) then
      vUzupelnionaIlosc := True;

  if ((Trim(vIndeks) <> '') and vUzupelnionaIlosc) then
  begin
    if (vDataDostawy = '') then
      vSql := 'execute procedure URZZEWPOZ_ADD('
        + vIdUrzZewNagl + ', '
        + QuotedStr(vIndeks) + ', '
        + 'cast(' + CurrToStrSt(StrStToCurr(vIlosc)) + ' as numeric(18,4)), '
        + 'null, null, null)'
    else
      vSql := 'execute procedure XXX_ZO_XLS_URZZEWPOZ_ADD('
        + vIdUrzZewNagl + ', '
        + QuotedStr(vIndeks) + ', '
        + 'cast(' + CurrToStrSt(StrStToCurr(vIlosc)) + ' as numeric(18,4)), '
        + IntToStr(vIdKontrah) + ', ' + vDataDostawy + ')';

    if (ExecuteSQL(vSql, 0) <> 1) then
    begin
      Inf('Nie utworzono pozycji urządzenia zewnętrznego.', 100);
      Exit;
    end;
  end;
end;

RealizujDokumentyOUZ(StrToInt(cIdUrzZew_ZOO_UKC), StrToInt(vIdUrzZewNagl));

vSql := 'select u.id_nagl from urzzewnagl u where u.id_urzzewnagl = ' + vIdUrzZewNagl;

vIdNagl := Trim(GetFromQuerySQL(vSql, 0));
if (vIdNagl = '') then
begin
  Inf('Nieokreślony dokument.', 100);
  Exit;
end;
```