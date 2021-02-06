# powershell

## test
```powershell
@powershell -NoProfile -ExecutionPolicy RemoteSigned "$s=[scriptblock]::create((gc \"%~f0\"|?{$_.readcount -gt1})-join\"`n\");&$s" %*&goto:eof 

# C#関数のカレントディレクトリをコマンド実行時のカレントディレクトリに変更
[System.IO.Directory]::SetCurrentDirectory((Get-Location).Path)

$count = 0;
$Args | % { % { [System.IO.File]::ReadAllText($_);$count++ } | % {$_ -replace "[^`r]`n", "<br>"} > ("test{0}.csv" -f $count) }
```
