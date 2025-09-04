# codex_execpolicy

本函式庫的目標是將擬執行的 [`execv(3)`](https://linux.die.net/man/3/execv) 指令分類為下列狀態之一：

- `safe` 指令可被視為安全（\*）。
- `match` 指令符合策略中的規則，但是否安全須由呼叫端依據將寫入的檔案自行決定。
- `forbidden` 禁止執行。
- `unverified` 無法判定安全性：交由使用者決定。

(\*) 是否將 `execv(3)` 呼叫視為「安全」，往往需要超出 `execv()` 引數以外的額外脈絡。例如，若您允許自主代理在原始碼樹中寫檔，則判斷 `/bin/cp foo bar` 是否「安全」需視呼叫行程的 `getcwd(3)`，以及以該 `getcwd()` 解析後 `foo` 與 `bar` 的 `realpath` 而定。
因此，驗證器不回傳布林值，而是回傳結構化結果，讓用戶端據以判斷此 `execv()` 呼叫的「安全性」。

例如，檢查 `ls -l foo` 可如此執行檢查器：

```shell
cargo run -- check ls -l foo | jq
```

其會以代碼 `0` 結束，並在標準輸出印出：

```json
{
  "result": "safe",
  "match": {
    "program": "ls",
    "flags": [
      {
        "name": "-l"
      }
    ],
    "opts": [],
    "args": [
      {
        "index": 1,
        "type": "ReadableFile",
        "value": "foo"
      }
    ],
    "system_path": ["/bin/ls", "/usr/bin/ls"]
  }
}
```

重點說明：

- `foo` 被標記為 `ReadableFile`，因此呼叫端應相對於 `getcwd()` 解析 `foo`，並以 `realpath` 取得實際路徑（可能是符號連結），以判斷讀取是否安全。
- 雖然指定的可執行檔為 `ls`，`"system_path"` 提供 `/bin/ls` 與 `/usr/bin/ls` 兩個可選路徑，以避免使用者 `$PATH` 上首先出現的 `ls`。若主機上存在其一，建議在 `execv(3)` 的第一個引數使用該絕對路徑，而非 `ls`。

此外，在本系統中「安全」並不保證指令一定執行成功。例如，如果系統認定代理可讀取 `/Users/mbolin/code/codex` 底下的任何檔案，則 `cat /Users/mbolin/code/codex/README.md` 可視為「安全」，但若該檔案不存在，仍會在執行時失敗。（不過此舉仍屬「安全」，因為代理沒有讀取未獲授權的檔案。）

## 策略（Policy）

目前預設策略定義於本 crate 內的 [`default.policy`](./src/default.policy)。

本系統採用 [Starlark](https://bazel.build/rules/language) 作為檔案格式，因為不同於 JSON 或 YAML，它支援「巨集」，且不犧牲安全與可重現性。（內部使用 [`starlark-rust`](https://github.com/facebook/starlark-rust) 作為 Starlark 的實作。）

此策略包含如下「規則」：

```python
define_program(
    program="cp",
    options=[
        flag("-r"),
        flag("-R"),
        flag("--recursive"),
    ],
    args=[ARG_RFILES, ARG_WFILE],
    system_path=["/bin/cp", "/usr/bin/cp"],
    should_match=[
        ["foo", "bar"],
    ],
    should_not_match=[
        ["foo"],
    ],
)
```

此規則表示：

- `cp` 可搭配以下任一旗標（「旗標」指不帶參數的選項）：`-r`、`-R`、`--recursive`。
- 參數開頭的 `ARG_RFILES` 表示需一個或多個「可讀檔案」參數。
- 參數結尾的 `ARG_WFILE` 表示需正好一個「可寫檔案」參數。
- 為在定義旁邊加入輕量級的單元測試，`should_match` 是應符合規則的 `execv(3)` 參數示例列表，`should_not_match` 則是不應符合的示例。於載入 `.policy` 檔時會驗證這些示例。

請注意：`.policy` 檔的語言仍在演進中；我們會持續擴充其表達力，使其能涵蓋所有視為「安全」的指令，同時防堵不安全的指令。

`default.policy` 的完整性會[透過單元測試](./tests)驗證。

此外，CLI 支援 `--policy` 參數，以指定自訂 `.policy` 檔進行臨時測試。

## 輸出型別：`match`

回到 `cp` 的例子，因規則中包含 `ARG_WFILE`，因此會回傳 `match` 而非 `safe`：

```shell
cargo run -- check cp src1 src2 dest | jq
```

若呼叫端考慮允許此指令，應解析 JSON 以挑出 `WriteableFile` 參數，並判斷寫入是否安全：

```json
{
  "result": "match",
  "match": {
    "program": "cp",
    "flags": [],
    "opts": [],
    "args": [
      {
        "index": 0,
        "type": "ReadableFile",
        "value": "src1"
      },
      {
        "index": 1,
        "type": "ReadableFile",
        "value": "src2"
      },
      {
        "index": 2,
        "type": "WriteableFile",
        "value": "dest"
      }
    ],
    "system_path": ["/bin/cp", "/usr/bin/cp"]
  }
}
```

請注意：`match` 的結束代碼仍為 `0`，除非指定 `--require-safe`，此時結束代碼為 `12`。

## 輸出型別：`forbidden`

也可定義規則，使其一旦符合某指令，便標示為「禁止」。例如，我們完全不允許代理執行 `applied deploy`，因此定義如下規則：

```python
define_program(
    program="applied",
    args=["deploy"],
    forbidden="Infrastructure Risk: command contains 'applied deploy'",
    should_match=[
        ["deploy"],
    ],
    should_not_match=[
        ["lint"],
    ],
)
```

需注意，若要將規則標記為禁止，必須以 `forbidden` 關鍵字參數說明原因。該原因會出現在輸出中：

```shell
cargo run -- check applied deploy | jq
```

```json
{
  "result": "forbidden",
  "reason": "Infrastructure Risk: command contains 'applied deploy'",
  "cause": {
    "Exec": {
      "exec": {
        "program": "applied",
        "flags": [],
        "opts": [],
        "args": [
          {
            "index": 0,
            "type": {
              "Literal": "deploy"
            },
            "value": "deploy"
          }
        ],
        "system_path": []
      }
    }
  }
}
```
