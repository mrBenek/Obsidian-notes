```csharp title:test
public static bool PrintRB(int id, string printerName, string jsonParams)
{
    if (id <= 0)
        return false;

    var wRes = UniData.RestCom.PostCall(
        string.Format("TstBaseMethods/\"PrintRB\"/{0}/{1}/", id.ToString(), printerName), jsonParams);

    if (IsRestError())
    {
        throw new System.ArgumentException("Błąd REST: " + ErrorMessage, "PrintRB");
    }

    return true;
}
```