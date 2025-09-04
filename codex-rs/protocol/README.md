# codex-protocol

此 crate 定義 Codex CLI 使用之協定的「型別」，包含 `codex-core` 與 `codex-tui` 之間通訊的「內部型別」，以及搭配 `codex mcp` 使用的「外部型別」。

此 crate 應維持最小相依。

理想情況下，本 crate 應避免包含「實質商業邏輯」。若有需求，可在其他 crate 透過 `Ext` 類型的 traits 為型別擴充功能。
