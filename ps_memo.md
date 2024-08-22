## ファイルコピー用

```powershell
# ソースフォルダとターゲットフォルダを指定
$sourceFolder = "C:\path\to\source"
$targetFolder = "C:\path\to\target"

# 日付のフォルダを作成し、ファイルを移動する関数
function Move-FilesByDate {
    param (
        [string]$sourceFolder,
        [string]$targetFolder
    )
    
    # ソースフォルダ内のファイルを取得
    $files = Get-ChildItem -Path $sourceFolder -File
    
    foreach ($file in $files) {
        # ファイル名に日付が含まれているかチェック (例: 2024-07-27)
        if ($file.Name -match "\d{4}-\d{2}-\d{2}") {
            # 日付を抽出
            $date = $matches[0]
            
            # 日付のフォルダを作成 (存在しない場合のみ)
            $dateFolder = Join-Path -Path $targetFolder -ChildPath $date
            if (-not (Test-Path -Path $dateFolder)) {
                New-Item -Path $dateFolder -ItemType Directory
            }
            
            # ファイルを日付フォルダに移動
            $destination = Join-Path -Path $dateFolder -ChildPath $file.Name
            Move-Item -Path $file.FullName -Destination $destination
        }
    }
}

# 関数を呼び出してファイルを移動
Move-FilesByDate -sourceFolder $sourceFolder -targetFolder $targetFolder

```





## ping

```powershell
ping -t www.yahoo.co.jp 2>&1 | ForEach-Object { "$(Get-Date -Format "yyyy/MM/dd HH:mm:ss") $_" } | Tee-Object -FilePath ("ping_" + (Get-Date -Format "yyyyMMdd_HHmmss") + ".log")
```







