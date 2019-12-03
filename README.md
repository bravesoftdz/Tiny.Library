# Tiny.Rtti

The project has the following conceptual features:
* Standard types, including FreePascal, old and new versions of Delphi, convenient functions for managing them.
* Universal data representation, regardless of programming language and compiler version.
* Convert standard RTTI to universal data representation.
* Minimizing dependencies on heavy units like Classes, Variants, Generics, etc.
* Cross-platform functions invoke, including FreePascal and old versions of Delphi, creation of function and interface interpreters.
* Marshalling (serialization and deserialization of data) through different formatters: JSON, XML, [FlexBin](FlexBin.RUS.md) and others.
* Easy to use unit testing library that does not access the memory manager.

### Invoke benchmark

![](data/InvokeBenchmark.png)
```pascal
procedure TForm1.SomeMethod(const X, Y, Z: Integer);
begin
  Tag := X + Y + Z;
end;

procedure TForm1.Button1Click(Sender: TObject);
const
  COUNT = 100000;
var
  i: Integer;
  LStopwatch: TStopwatch;
  LContext: System.Rtti.TRttiContext;
  LMethod: System.Rtti.TRttiMethod;
  LMethodEntry: Tiny.Rtti.PVmtMethodExEntry;
  LSignature: Tiny.Invoke.TRttiSignature;
  LInvokeFunc: Tiny.Invoke.TRttiInvokeFunc;
  LDump: Tiny.Invoke.TRttiInvokeDump;
  T1, T2: Int64;
begin
  LStopwatch := TStopwatch.Create;
  LContext := System.Rtti.TRttiContext.Create;
  LMethod := LContext.GetType(TForm1).GetMethod('SomeMethod');
  LMethodEntry := Tiny.Rtti.PTypeInfo(TypeInfo(TForm1)).TypeData.ClassData.MethodTableEx.Find('SomeMethod');
  LSignature.Init(LMethodEntry^);
  LInvokeFunc := LSignature.OptimalInvokeFunc;

  LStopwatch.Start;
  for i := 1 to Count do
  begin
    LMethod.Invoke(Form1, [1, 2, 3]);
  end;
  T1 := LStopwatch.ElapsedMilliseconds;

  LStopwatch.Start;
  for i := 1 to Count do
  begin
    PPointer(@LDump.Bytes[LSignature.DumpOptions.ThisOffset])^ := Form1;
    PInteger(@LDump.Bytes[LSignature.Arguments[0].Offset])^ := 1;
    PInteger(@LDump.Bytes[LSignature.Arguments[1].Offset])^ := 2;
    PInteger(@LDump.Bytes[LSignature.Arguments[2].Offset])^ := 3;
    LInvokeFunc(@LSignature, LMethodEntry.CodeAddress, @LDump);
  end;
  T2 := LStopwatch.ElapsedMilliseconds;

  Caption := Format('System.Rtti: %dms, Tiny.Rtti: %dms', [T1, T2]);
end;
```