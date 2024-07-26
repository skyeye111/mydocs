## メモをエクスポートする

```vb
Sub ExportNotes()
    Dim olApp As Outlook.Application
    Dim olNamespace As Outlook.NameSpace
    Dim olNotesFolder As Outlook.MAPIFolder
    Dim olItem As Object
    Dim fileNum As Integer
    Dim filePath As String
    
    ' Outlookアプリケーションオブジェクトの取得
    Set olApp = New Outlook.Application
    Set olNamespace = olApp.GetNamespace("MAPI")
    
    ' メモフォルダの取得
    Set olNotesFolder = olNamespace.GetDefaultFolder(olFolderNotes)
    
    ' エクスポートするファイルのパスを設定
    filePath = "C:\Users\maco\Desktop\memo\OutlookNotes.txt"
    
    ' テキストファイルの作成
    fileNum = FreeFile
    Open filePath For Output As #fileNum
    
    ' メモの内容をテキストファイルに書き込み
    For Each olItem In olNotesFolder.Items
        ' メモアイテムかどうかを確認
        If TypeOf olItem Is Outlook.NoteItem Then
            Dim olNote As Outlook.NoteItem
            Set olNote = olItem
            
            Print #fileNum, "Subject: " & olNote.Subject
            Print #fileNum, "Body: " & olNote.Body
            Print #fileNum, String(50, "-")
        End If
    Next olItem
    
    ' ファイルを閉じる
    Close #fileNum
    
    ' オブジェクトの解放
    Set olItem = Nothing
    Set olNotesFolder = Nothing
    Set olNamespace = Nothing
    Set olApp = Nothing
    
    MsgBox "メモがエクスポートされました。", vbInformation
End Sub
```



## メモをインポートする

```vb
Sub ImportNotes()
    Dim olApp As Outlook.Application
    Dim olNamespace As Outlook.NameSpace
    Dim olNotesFolder As Outlook.MAPIFolder
    Dim olNote As Outlook.NoteItem
    Dim fileNum As Integer
    Dim filePath As String
    Dim fileLine As String
    Dim noteSubject As String
    Dim noteBody As String
    
    ' Outlookアプリケーションオブジェクトの取得
    Set olApp = New Outlook.Application
    Set olNamespace = olApp.GetNamespace("MAPI")
    
    ' メモフォルダの取得
    Set olNotesFolder = olNamespace.GetDefaultFolder(olFolderNotes)
    
    ' インポートするファイルのパスを設定
    filePath = "C:\Users\maco\Desktop\memo\OutlookNotes.txt"
    
    ' テキストファイルを開く
    fileNum = FreeFile
    Open filePath For Input As #fileNum
    
    ' ファイルの行を読み取り、メモを作成
    Do While Not EOF(fileNum)
        ' メモのSubjectを読み取り
        Line Input #fileNum, fileLine
        noteSubject = Mid(fileLine, 10) ' "Subject: " の後の文字列
        
        ' メモのBodyを読み取り
        Line Input #fileNum, fileLine
        noteBody = Mid(fileLine, 6) ' "Body: " の後の文字列
        
        ' 次の区切り線まで読み飛ばす
        Do While Not EOF(fileNum)
            Line Input #fileNum, fileLine
            If fileLine = String(50, "-") Then
                Exit Do
            End If
        Loop
        
        ' 新しいメモを作成
        Set olNote = olNotesFolder.Items.Add(olNoteItem)
        'olNote.Subject = noteSubject ' この行がエラーになる場合に備えて、以下の代替方法を試してください
        olNote.Body = "Subject: " & noteSubject & vbCrLf & noteBody ' 代替方法：件名を本文に含める
        olNote.Body = noteBody
        
        olNote.Save
    Loop
    
    ' ファイルを閉じる
    Close #fileNum
    
    ' オブジェクトの解放
    Set olNote = Nothing
    Set olNotesFolder = Nothing
    Set olNamespace = Nothing
    Set olApp = Nothing
    
    MsgBox "メモがインポートされました。", vbInformation
End Sub

```







