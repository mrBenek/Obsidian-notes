Elsta
```pascal
procedure DodanieRezerwacjiDoZam(Sender : TObject);
var
  vIdNagl: String;
  vSql: String;

begin
  vIdNagl := inttostr(frm.QueryMain.FieldByName('Id_Nagl').AsInteger);
  if (frm.QueryMain.MarkCount > 0) then
    inf300('Proszę zaznaczyć jedną pozycję')
  else
  begin
    vSql := 'Execute procedure XXX_REZ_ZW_ZD4(' + vIdNagl + ')';
    if (ExecuteSQL(vSql, 0) <> 1) then
    begin
      Inf('Nie utworzono rezerwacji.', 100);
      Exit;
    end;
  end;    
end;
```