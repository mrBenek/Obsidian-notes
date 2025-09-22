```csharp title:"Uruchomienie programu C# z wtyczki w Prestiż"
//TfDokSprzedazDok
{$ADDTYPE TDotNetPrestiz}
{$ADDTYPE TfDokSprzedazDok}

var
    frm : TfDokSprzedazDok;
    wDNRun: TDotNetPrestiz;
    res : string;

procedure MojProgram(Sender: TObject);
begin
  wDNRun :=  TDotNetPrestiz.Create;
  try
    try
      res := wDNRun.RunDotNetProgram('MojDotNet', '1', '');
      inf300('Odpowiedź z dotNeta : ' + res);
    except
    end;
  finally
    wDNRun.Free;
  end;
end;

begin
  if (Self is TfDokSprzedazDok) then
  begin
    if (frm = nil) then
    begin
      frm := Self as TfDokSprzedazDok;
    end;
    if (frm <> nil) then
    begin
      //if (frm.WindowId = [w_ID]) then
      begin
        // co ma zrobić wtyczka
        PluginsAddAction(Self, 'Mój progrsm', 'money_bag_24', @MojProgram);
      end;
    end;
  end;
end.

```