
* 使えそうなサイト
  * [文字列操作](https://docs.microsoft.com/ja-jp/powershell/scripting/learn/deep-dives/everything-about-string-substitutions?view=powershell-7.1)


* bat化
```powershell
@echo off&setlocal enabledelayedexpansion & for %%f in (%*) do (set p=!p!"\"%%f\"" ) 
powershell -NoProfile -ExecutionPolicy RemoteSigned "set-location '%CD%';$s=[scriptblock]::create((gc \"%~f0\"|?{$_.readcount -gt2})-join\"`n\");&$s" !p!&goto:eof 

# 引数を受け取りたい場合、powershell本文で下記のように書けば受け取れる。ドラッグアンドドロップされたファイル名もちゃんと格納される
$arg0 = $Args[0]

# パイプライン処理で引数の中身全て処理する場合の書き方
$Args | % { echo $_ }

```

* 上記の管理者権限昇格を自動で行う版
```powershell
@powershell -NoProfile -ExecutionPolicy RemoteSigned "if (!([Security.Principal.WindowsPrincipal][Security.Principal.WindowsIdentity]::GetCurrent()).IsInRole([Security.Principal.WindowsBuiltInRole]::Administrator)){Start-Process powershell -ArgumentList \"cd %CD%;%~f0 %*\" -Verb RunAs -WindowStyle Hidden -Wait}else{set-location '%CD%';$s=[scriptblock]::create((gc \"%~f0\"|?{$_.readcount -gt1})-join\"`n\");&$s %*}" &goto:eof 


```

* パッケージ追加（excel)
```powershell
if(!(Get-Module -ListAvailable -Name ImportExcel)){ Install-Module ImportExcel -Scope CurrentUser -Force }
```

* 高度な正規表現(マッチした要素を名前付きタグを付けて抽出）
```powershell
cat "D:\work\Log.log" -Wait -Tail 0 -Encoding UTF8 |
? {$_ -match '\[(?<Level>.*)\]\s(?<Title>.*?)\s\[(?<Tag>.*?)\]\s(?<Msg>.*)'} |
% {[PSCustomObject]$Matches | 
select Level, Tag, Title, Msg | 
ConvertTo-Json -Compress}
```

* よくあるREST API操作（登録）
```powershell
Invoke-RestMethod "http://{url}/_api/" -Method Post -ContentType "application/json; charset=utf-8" -Body '{"name":"test","num":1}'
```

* よくあるREST API操作（取得）
```powershell
Invoke-RestMethod "http://{url}/_api/"
```

* redmine等のWebツールのAPI操作
```powershell
# ユーザー情報の入力
$c = Get-Credential
# REST APIの呼び出し例①
Invoke-RestMethod http://{redmine url}/issues.json?tracker_id={id} -Credential $c
# REST APIの呼び出し例②
Invoke-RestMethod http://{redmine url}/issues.json?tracker_id={id} -Credential $c | % issues | select {$_.
author.name}, {$_.custom_fields[0].value }, {$_.custom_fields[1].value}, description | Format-Table
```

* excel
```powershell
Import-Excel .\data.xlsx -StartRow 2 | select -Skip 2 | ConvertTo-Json
```

* 行を跨いだ処理（改行の置換等）をする場合はC#のReadAllTextを使うと楽で安全。 -joinの場合、改行を連結しようとすると全てCRLFに変換されてしまったりするので注意
```powershell
[System.IO.File]::ReadAllText(".\test.csv") | % {$_ -replace "[^`r]`n", "<br>"}
```
* テキストファイルを引数で受け取ってテキスト処理してから吐き出すバッチの例
```powershell
@echo off&setlocal enabledelayedexpansion & for %%f in (%*) do (set p=!p!"\"%%f\"" ) 
powershell -NoProfile -ExecutionPolicy RemoteSigned "set-location '%CD%';$s=[scriptblock]::create((gc \"%~f0\"|?{$_.readcount -gt2})-join\"`n\");&$s" !p!&goto:eof 

# C#関数のカレントディレクトリをコマンド実行時のカレントディレクトリに変更
[System.IO.Directory]::SetCurrentDirectory((Get-Location).Path)

$count = 0;
$Args | % { % { [System.IO.File]::ReadAllText($_);$count++ } | % {$_ -replace "[^`r]`n", "<br>"} > ("test{0}.csv" -f $count) }
```

* ファイル名変更例
```powershell
$path = "c:\work\test.txt"
# シンプルに置換
$path -replace ".txt", "_modified.txt"
```

* 名前付き引数の受け取り例
```powershell
@echo off&setlocal enabledelayedexpansion & for %%f in (%*) do (set p=!p!"\"%%f\"" ) 
powershell -NoProfile -ExecutionPolicy RemoteSigned "set-location '%CD%';$s=[scriptblock]::create((gc \"%~f0\"|?{$_.readcount -gt2})-join\"`n\");&$s" !p!&goto:eof 

Param(
	[Switch]$h,
	$i,
	$o, 
	$num
)

if($h){@"
help message
-i 必須 入力ファイル
-o 必須 入力内容はTest1かTest2のみ受け付ける
-num 必須 ４桁の数値のみ受け付ける
"@
exit
}

if($Args -or !$i -or !$o -or !$num){@"
入力形式が正しくありません。-hで使い方を確認してください。
"@
exit
}


"-i " + $i
"-o " + $o
"-num " + $num
```
* groupの使い方
```powershell
Read *.log |
Group-Object Date |
%{
    $m = $_ | select -ExpandProperty Group | measure -Property TimeTaken -Average -Maximum -Minimum
    $_ | Add-Member Average $m.Average
    $_ | Add-Member Max $m.Maximum
    $_ | Add-Member Min $m.Minimum
    $_
}|
select Name, Count, Average, Max, Min|
ft -AutoSize
```

* propertyの回し方（csv内の全レコードに置換処理を行う例
```powershell
Import-Csv .\test.txt | %{
  $props = $_ | Get-Member -MemberType NoteProperty | Select -ExpandProperty Name
  foreach($prop in $props) { $_.$prop = $_.$prop -replace "`n", "<br>" } $_
} | Export-Csv -path data.csv -Encoding Default -NoTypeInformation
```


* https://ericzimmerman.github.io/KapeDocs/#!Pages\3.-Using-KAPE.md
* https://ericzimmerman.github.io/KapeDocs/#!index.md
* https://ericzimmerman.github.io/#!index.md
* https://ericzimmerman.github.io/#!documentation.md
* https://ericzimmerman.github.io/#!index.md
* https://ericzimmerman.github.io/#!index.md#Requirements_and_troubleshooting
* https://binaryforay.blogspot.com/2016/02/lecmd-v0600-released.html


