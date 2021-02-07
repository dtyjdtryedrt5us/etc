
* 使えそうなサイト
  * [文字列操作](https://docs.microsoft.com/ja-jp/powershell/scripting/learn/deep-dives/everything-about-string-substitutions?view=powershell-7.1)


* bat化
```powershell
@powershell -NoProfile -ExecutionPolicy RemoteSigned "$s=[scriptblock]::create((gc \"%~f0\"|?{$_.readcount -gt1})-join\"`n\");&$s" %*&goto:eof 

# 引数を受け取りたい場合、powershell本文で下記のように書けば受け取れる。ドラッグアンドドロップされたファイル名もちゃんと格納される
$arg0 = $Args[0]

# パイプライン処理で引数の中身全て処理する場合の書き方
$Args | % { echo $_ }

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
@powershell -NoProfile -ExecutionPolicy RemoteSigned "$s=[scriptblock]::create((gc \"%~f0\"|?{$_.readcount -gt1})-join\"`n\");&$s" %*&goto:eof 

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
