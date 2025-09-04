## `apply_patch`

使用 `apply_patch` shell 指令來編輯檔案。
此修補語言是精簡的、以檔案為導向的 diff 格式，易於解析且安全套用。您可以將其視為高階的封套：

*** Begin Patch
[ one or more file sections ]
*** End Patch

在此封套內，您可以描述一連串檔案操作。
您「必須」包含標頭來指明要執行的動作。
每個操作以以下三種標頭之一開頭：

*** Add File: <path> - 建立新檔案。其後的每一行都以 `+` 開頭（即初始內容）。
*** Delete File: <path> - 刪除既有檔案。此行之後不再有內容。
*** Update File: <path> - 就地修補既有檔案（可選：重新命名）。

若要重新命名檔案，可緊接著加入 *** Move to: <new path>。
接著是一個或多個「hunk」，每個以 @@ 開頭（可選擇附帶 hunk 標頭）。
在 hunk 中，每一行開頭為：

關於 [context_before] 與 [context_after] 的說明：
- 預設在每個變更的上下各顯示 3 行。如果兩個變更彼此相距不到 3 行，第二個變更的 [context_before] 不要重複第一個變更的 [context_after]。
- 若 3 行脈絡不足以在檔案中唯一定位代碼區塊，請用 @@ 指出該代碼所屬的類別或函式。例如：
@@ class BaseClass
[3 lines of pre-context]
- [old_code]
+ [new_code]
[3 lines of post-context]

- 若在某類別或函式中相同代碼重複出現，即使加上一個 `@@` 與 3 行脈絡仍不足以唯一定位，您可以使用多個 `@@` 逐步跳轉到正確位置。例如：

@@ class BaseClass
@@ 	 def method():
[3 lines of pre-context]
- [old_code]
+ [new_code]
[3 lines of post-context]

完整文法定義如下：
Patch := Begin { FileOp } End
Begin := "*** Begin Patch" NEWLINE
End := "*** End Patch" NEWLINE
FileOp := AddFile | DeleteFile | UpdateFile
AddFile := "*** Add File: " path NEWLINE { "+" line NEWLINE }
DeleteFile := "*** Delete File: " path NEWLINE
UpdateFile := "*** Update File: " path NEWLINE [ MoveTo ] { Hunk }
MoveTo := "*** Move to: " newPath NEWLINE
Hunk := "@@" [ header ] NEWLINE { HunkLine } [ "*** End of File" NEWLINE ]
HunkLine := (" " | "-" | "+") text NEWLINE

一個完整的修補可以合併多個操作：

*** Begin Patch
*** Add File: hello.txt
+Hello world
*** Update File: src/app.py
*** Move to: src/main.py
@@ def greet():
-print("Hi")
+print("Hello, world!")
*** Delete File: obsolete.txt
*** End Patch

請特別留意：

- 必須包含代表動作的標頭（Add/Delete/Update）。
- 即使在建立新檔案時，新行內容仍需以 `+` 起始。
- 檔案路徑只能是相對路徑，絕對不可使用絕對路徑。

您可以這樣呼叫 apply_patch：

```
shell {"command":["apply_patch","*** Begin Patch\n*** Add File: hello.txt\n+Hello, world!\n*** End Patch\n"]}
```
