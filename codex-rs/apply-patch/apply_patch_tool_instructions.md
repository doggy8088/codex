## `apply_patch`

Use the `apply_patch` shell command to edit files.
您的 patch language is a stripped‑down, file‑oriented diff format designed 到 be easy 到 parse 和 safe 到 apply. 您 can think 的 它 作為 a high‑level envelope:

*** Begin Patch
[ one 或 more file sections ]
*** End Patch

Within 那個 envelope, 您 get a sequence 的 file operations.
您 MUST include a header 到 specify the action 您 are taking.
Each operation starts 使用 one 的 three headers:

*** Add File: <path> - 建立 a new file. Every following line is a + line (the initial contents).
*** Delete File: <path> - remove an existing file. Nothing follows.
*** 更新 File: <path> - patch an existing file 在 place (optionally 使用 a rename).

May be immediately followed 透過 *** Move 到: <new path> 如果 您 want 到 rename the file.
Then one 或 more “hunks”, each introduced 透過 @@ (optionally followed 透過 a hunk header).
Within a hunk each line starts 使用:

為 instructions 在 [context_before] 和 [context_after]:
- 透過 default, show 3 lines 的 code immediately above 和 3 lines immediately below each change. 如果 a change is within 3 lines 的 a previous change, do NOT duplicate the first change’s [context_after] lines 在 the second change’s [context_before] lines.
- 如果 3 lines 的 context is insufficient 到 uniquely identify the snippet 的 code within the file, use the @@ operator 到 indicate the class 或 function 到 哪個 the snippet belongs. 為 instance, we might have:
@@ class BaseClass
[3 lines 的 pre-context]
- [old_code]
+ [new_code]
[3 lines 的 post-context]

- If a code block is repeated so many times in a class or function such that even a single `@@` statement and 3 lines of context cannot uniquely identify the snippet of code, you can use multiple `@@` statements to jump to the right context. For instance:

@@ class BaseClass
@@ 	 def method():
[3 lines 的 pre-context]
- [old_code]
+ [new_code]
[3 lines 的 post-context]

The full grammar definition is below:
Patch := Begin { FileOp } End
Begin := "*** Begin Patch" NEWLINE
End := "*** End Patch" NEWLINE
FileOp := AddFile | DeleteFile | UpdateFile
AddFile := "*** Add File: " path NEWLINE { "+" line NEWLINE }
DeleteFile := "*** Delete File: " path NEWLINE
UpdateFile := "*** 更新 File: " path NEWLINE [ MoveTo ] { Hunk }
MoveTo := "*** Move 到: " newPath NEWLINE
Hunk := "@@" [ header ] NEWLINE { HunkLine } [ "*** End 的 File" NEWLINE ]
HunkLine := (" " | "-" | "+") text NEWLINE

A full patch can combine several operations:

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

它 is important 到 remember:

- 您 must include a header 使用 您的 intended action (Add/Delete/更新)
- You must prefix new lines with `+` even when creating a new file
- File references can only be relative, NEVER ABSOLUTE.

您 can invoke apply_patch like:

```
shell {"command":["apply_patch","*** Begin Patch\n*** Add File: hello.txt\n+Hello, world!\n*** End Patch\n"]}
```
