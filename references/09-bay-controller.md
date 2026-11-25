# 09 Bay Controller (間隔控制單元 BCU) 攻略

來源：PM180-ApplicationNote-BayControlUnit (BG0531)。原文：`fulltext/PM180-ApplicationNote-BayControlUnit.txt` (§4 有完整設定參數表)。

## 功能概要
- 監控 **2 台斷路器 (XCBR1–2) + 14 台隔離/接地開關 (XSWI1–14)**。
- 位置指示：一般 2 接點 (N.O./N.C.) 或分相 6 接點；可只接一個接點 (另一個取反)；來源可為 DI、邏輯方程、event flag、或 GOOSE 遠端指示 (ExtInd)。
  - 6 接點：三極同位置 → 該位置；任一極 bad (兩接點都閉) → bad；其餘 → intermediate。位置變化 ½ 週波內偵測，自動記 Event log。
- 每台開關最多 6 個控制輸出：合閘 1、分閘 1–2 (雙跳圈同步)、select-close 1、select-open 1–2；可用 64 個繼電器中任一 (select-close 與 select-open 可共用同一顆)。被 BCU 使用的繼電器自動設為 unlatched/normal，且不能被直接遙控。
- 沒設控制輸出 = 「虛擬開關」，只做位置監視供連鎖使用。
- 位置替代 (substitution)、非控制性跳脫指示 (AlmFlt，5 秒自動復歸)、操作次數計數 (0–9999，可重置)。
- 韌體需 ≥ 20.XX.07 (BCU app note 記載)。

## 控制參數
| 參數 | 說明 |
|---|---|
| Command Duration | Modbus/邏輯命令的預設脈衝寬度；61850 可由 CSWI$CF$Pos$pulseConfig onDur (0–60000ms) 覆蓋 |
| Pulse Duration Control Logic | 自適應脈衝：邏輯方程為 TRUE 時延長脈衝 (例：直到到位或失壓)；實際脈衝不會短於設定值 |
| Start Moving Timeout | 不動作逾時 (命令後多久要開始動)；0 = 不監督 |
| Command Termination Timeout | 整體操作監督時間，逾時即中止並回報 |
| Interlocking (Open/Close) | 連鎖邏輯方程，TRUE 才允許；不需連鎖就設 TRUE |
| Interlocking Bypass | 一次性、有逾時的連鎖旁路 (通常接 "Interlocking OFF" 開關)；用一次或逾時後失效，方程需回 FALSE 才能再充能 |

## 命令被拒的條件
1. 控制埠無權限 (被停用/封鎖，或 61850 client 時設備在 Local)；2. 開關設定不可控；3. 已有命令執行中；4. 已在目標位置；5. 位置無效；6. 位置為替代值；7. 連鎖封鎖；8. 控制輸出被邏輯封鎖。

## 控制權 (Switching Permissions)
- **REMOTE CONTROL 旗標** (STATIC EVENTS 群組)，預設 OFF；用 setpoint 讀 Local/Remote 切換開關 DI，執行 "REMOTE CONTROL" 動作。OFF 時拒絕 **IEC 61850** 控制 (Loc = 其反相)。
- **Modbus 不受 REMOTE CONTROL 影響** (可當就地操作站)。
- 多使用者時用「直接控制埠」權限：每個埠 (COM1–5、USB、Ethernet client IP) 可設 Allowed / Disabled / Blocked；Blocked 埠由 setpoint 的 "UNBLOCK CONTROL PORT" 動作授權。PAS → Bay Control Setup → Control Ports (§4.2)。

## 邏輯可用變數 (setpoint / PLC)
`XCBRn_Pos`/`XSWIn_Pos` (Intermediate/OFF/ON/Bad)、`*_OpCls`、`*_OpOpn` (執行中=ON)、`*_OpCnt`、`ExtInd1–128` (單點)、`ExtInd1:2…127:128` (雙點)、`ExtiVal1–32`、`ExtfVal1–32`、`REMOTE CONTROL`。
動作：`CLOSE SWITCH` / `OPEN SWITCH` (目標 XCBR1–2、XSWI1–14)、`REMOTE CONTROL`、`UNBLOCK CONTROL PORT` (ALL/COM1–5/USB/Ethernet)。

## IEC 61850 對應 (CTRL LD)
- XCBRn / XSWIn：`ST.Loc`、`ST.Pos` (位置)、`ST.BlkOpn` / `ST.BlkCls` (連鎖反相)、`ST.OpCnt`、`ST.AlmFlt` (XCBR)。
- CILOn (n=1–2 對應 XCBR、3–16 對應 XSWI1–14)：`ST.EnaOpn` / `ST.EnaCls`、`SV.*.subEna/subVal` (旁路)。
- CSWIn：`CO.Pos.ctlVal` (OFF=分、ON=合)、`ST.Pos`、`SV.Pos.subEna/subVal` (替代)、`CF.Pos.pulseConfig`。
- 控制模型：**direct-with-enhanced-security**，CommandTermination 回報；失敗時 LastApplError AddCause：1 Not-supported、2 Blocked-by-switching-hierarchy、4 Invalid-position、5 Position-reached、8 Blocked-by-Mode、9 Blocked-by-process、10 Blocked-by-interlocking、12 Command-already-in-execution、13 Blocked-by-health、16 Time-limit-over。
- 開關前綴預設 QA1/QA2 (XCBR)、QB1…/QE… (XSWI)，可改，改完重新匯出 CID。

## Modbus 對應
| 位址 | 內容 |
|---|---|
| 29440–29471 | 位置 Pos (每台 UINT32：00=中間、01=OFF、10=ON、11=bad)；+0 XCBR1、+2 XCBR2、+4 XSWI1 … +30 XSWI14 |
| 29568 / 29632 | OpOpn / OpCls 執行中 bitmap (bit0 XCBR1, bit1 XCBR2, bit2–15 XSWI1–14) |
| 29696–29727 | 操作次數 (R/W 可清) |
| **44458–44459** | **合閘 (ON/Close) 命令 bitmap**，寫 1 動作 |
| **44460–44461** | **分閘 (OFF/Open) 命令 bitmap** |
| 44462 / 44464 | 位置替代 啟用 / 停用 bitmap |
| 44466–44481 | 位置替代值 (UINT16 每台) |
| 44514 / 44516 | OpOpn / OpCls (alias) |
| 44518 / 44520 | EnaOpn / EnaCls 連鎖狀態 (1=允許) |
| 44522 | 替代中 bitmap |
| 44524 | AlmFlt 跳脫告警 bitmap |
| **44526–44541** | **命令終止狀態** (每台 UINT16，碼同上 AddCause，0=成功) |
| 28160 / 28288 / 28416–28609 | GOOSE 外部量 ExtiVal / ExtfVal / ExtInd |
- 密碼保護時先寫 44378 授權。

## 設定步驟 (PAS → Meter Setup → Bay Control Setup)
1. 選 Switch Name → 設屬性 (前綴、類型)、位置輸入 (DI / 邏輯 / flag / ExtInd)、控制輸出繼電器、脈衝寬度、逾時。
2. Switch Enabled = Enabled。
3. 用 Setpoint 或 PLC Configurator 寫連鎖方程、REMOTE CONTROL、旁路邏輯。
4. Control Ports 設定控制權。
5. Send + Save 到 database (前綴才會在 PAS 顯示)；若用 61850 重新匯出 CID。
6. 測試：先用虛擬開關/無輸出驗證邏輯，再接實際跳合閘迴路。
