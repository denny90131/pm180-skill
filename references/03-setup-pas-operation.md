# 03 設定與操作攻略 (RDM / Telnet / PAS)

來源：PM180-Operation-Manual (BG0525 Rev.A7)。原文搜尋：`fulltext/PM180-Operation-Manual.txt`。

## 1. 運作模式與 LED
| 模式 | CPU LED | 說明 |
|---|---|---|
| Operational | 亮 1 秒、滅 1 秒 | 正常 |
| Energy Test | — | 能量累計不受影響、setpoint 與錄波停止，用於校驗脈衝 LED (預設 0.1 Wh/pulse，二次側值) |
| Service | 2 閃 + 1 秒滅 | 韌體升級用，所有功能停止 |
| Diagnostics | 3 閃 + 1 秒滅 | 嚴重錯誤，功能停止 (通訊埠仍可用)，RDM 顯示 "Critic Error" |
- MAIN POWER / BACKUP POWER 綠燈：對應電源有電。kWh/kvarh 紅燈：能量脈衝。

## 2. 出廠預設值 (常被問)
| 項目 | 預設 |
|---|---|
| COM1–COM4 | Modbus RTU、19200、8N1、位址 1 (COM1 介面 RS-232 舊 CPU / RS-485 N3；COM2/3 RS-485) |
| 密碼 | 0；密碼保護預設關閉 (8 位數字，所有埠共用) |
| Wiring | 4LN3；PT=1.0；PT 二次 120V；CT 一次 5A；頻率 50Hz (北美 60)；相序 ABC |
| 需量 | 功率需量 15 分鐘 block、滑動視窗 1 個；V/A/諧波需量 900 秒 |
| 時區 | -300 分 (美東)；DST 啟用，美國規則 → **台灣請改 +480、DST 停用** |
| 時間同步 | GPS IRIG-B (無訊號時自動用內部 RTC，但會報診斷) |
| 能量位數 | 9 位 (roll 10^9)、小數位 0 |
| 電壓/電流 scale | 828V / 2×或4× CT 二次 (IEC/ANSI) |
| SNTP | 停用，輪詢 600s，伺服器 192.36.143.151 / 130.149.17.21 |
| Setpoint #1 | 出廠設為每 15 分鐘觸發 Data log #1、#2 |

## 3. RDM180 面板操作
- 顯示鍵：MIN/MAX、PQ (閃爍/總諧波)、ENERGY、↑↓翻頁；同時按↑↓回第一頁。
- 選單：按 SELECT 進 MAIN 選單 → `StA` (狀態)、`OPS` (只看設定)、`CHG` (修改設定，可能要密碼)。
- 選單內：SELECT 選視窗 (閃爍)、ENTER 確認、ESC 返回、↑↓改值 (按住 5 秒跳下一位)。
- CHG 子選單：
  - `diSP` 顯示器自身 (更新率、自動捲動、主/二次電壓、`Addr`/`bAud`/`dAtA` 須與 PM180 COM3 相同)
  - `rtc` 時鐘 (時間前 "E"=GPS 鎖定、"U"=失鎖；DST；`OFFSET` 時區分鐘)
  - `bASc` 基本設定：`ConF` 接線模式、`Pt` PT 比 (1.0–6500.0)、`Pt.SEc` PT 二次 (10–690V)、`Pt.4`/`Pt.4SEc` V4、`Ct` I1–I3 CT 一次 (1–10000A)、`Ct.4`、`Ct.E` 擴展輸入 CT、`U.dC.` VDC 額定、`FrEq`、`PH.Ord.`、`Ld C` 最大負載電流 (TDD 用)
  - `dnd` 需量：`P.dP` 功率需量週期 1/2/5/10/15/20/30/60 分、`n.P.dP` 滑動視窗數 1–15、`d.Snc.In` 同步來源 (Clc 或 DI)、`A.dP`/`U.dP`/`H.dP` 秒
  - `Port.1`~`Port.3`：`Prot` (rtu/ASCII/dnP3)、`rS` (232/422/485)、`Addr` (Modbus 1–247、DNP 1–255)、`bAud` (300–115200)、`dAtA` (7E/8n/8E)
  - `OPtS`：功率計算模式 (rEAc / nAct)、能量 roll 值、`test` 能量測試模式
  - `AccS`：`PASS` 改密碼、`CtrL` On/OFF 密碼保護
  - `rESEt`：A.dnd、P.dnd、dnd、Enrg、Lo.Hi、tOU.d、tOU.E、Cnt/Cnt1–16、diAG — 選好後**按住 ENTER 約 5 秒**直到 "do" 變 "done"。
- 未設密碼保護時，在顯示頁按住 SELECT + ENTER 5 秒可直接清除該頁計數/最大值/診斷。
- RDM 顯示 "Con.Err." = 與 COM3 通訊失敗 → 查線、是否 Service mode、COM3 與 RDM 參數是否一致。

## 4. Telnet (透過 ETH，port 23)
- 指令：`h`/`?` 說明、`I` 韌體資訊、`password`、`time [hh:mm:ss]`、`date [dd/mm/yy]`、`com[port]` 序列埠設定、`net` 目前網址、`ip` 網路設定、`log`、`stat` 網路統計、`diag` 診斷、`clrdiag` 清除診斷、`reset` 重啟。
- **一定要登入密碼 (預設 0)**，不論保護是否啟用。改 IP 或 reset 後連線會斷。
- C37.118.2 串流檢查：`enet` → `proto`。PMU 1588 網口另有 Telnet，密碼 `1588`，指令 `ptp`。

## 5. PAS 軟體
### 安裝與連線
- Windows (至少 XP)，USB 需安裝 PAS 光碟內 USB 驅動。
- Tools → Configuration → Instrument Setup：建立 **Site database** (存放於 PAS 安裝目錄 `Sites`)，選 Model = PM180、標準電流輸入選項、記憶體大小；**Current over-range：20A (ANSI) 設 400%、10A (IEC) 設 200%**。
- 連線方式：Serial (Protocol、Response Time-out、Break Time-out 預設 10ms、Retries)、Internet (IP + Host Port：502 Modbus、20000 DNP3、2404 IEC 60870-5)、Cellular (Dial-up)、USB。
- 經 ETH/Cellular 時站號任意。

### 設定流程 (重要)
1. **先建立並 Save Basic Setup** (General Setup) 到 site database — 其他設定以它為基準。
2. 離線編輯：Meter Setup 選單 → 各頁籤 → Save as… 存檔。
3. 下載：勾選工具列 On-line → 個別頁籤 Send，或 Meter Setup → Download Setups (全部)。
4. 上傳：On-line → Meter Setup → Upload Setups。
5. 複製設定到其他 site：Meter Setup → Copy to…

### 主要設定頁籤 (Meter Setup)
| 選單/頁籤 | 內容 |
|---|---|
| Communications Setup → Serial Ports / Network Setup / Dial-Up Networking / SNTP Client / ExpertPower Client / TCP Notification Client / IEEE C37.118.2 / PMU IEEE 1588 Port | 通訊參數 (IP 在 Network Setup) |
| General Setup → Basic Setup | 接線、PT/CT、頻率、相序、需量 |
| General Setup → Advanced Setup (Device Options) | 功率計算模式 S=f(P,Q) 或 Q=f(S,P)、能量 roll/小數位、TDD 最大負載電流、能量 LED、Volts/Amps Scale |
| General Setup → Transformer Correction | CT/PT 比誤差 (0.900–1.100) 與相角誤差 (±600 分) |
| General Setup → Local Settings | 時區、DST、時間同步來源 (GPS IRIG-B / SNTP / DI1–48) |
| General Setup → Digital Inputs / Relay Outputs / Analog Inputs / Analog Outputs / Counters / Periodic Timers / Control/Alarm Setpoints | I/O 與邏輯 |
| Memory/Log → Log Memory / Event / SOE / Data Log / Waveform / PQ Recorder / Fault Recorder / Fault Locator | 錄波 |
| Energy/TOU → Billing/TOU Registers / Daily Profiles / Calendar | 費率 |
| Protocol Setup → MODBUS Registers / DNP Options / DNP Class 0 / DNP Event Setpoints | 協定 |
| IEC 60870-5 Setup / IEC 61850 Setup | 協定 |
- Monitor 選單：RT Data Monitor (33 組 data set，每組 40 參數)、Device Control (Remote Relay Control、Event Flags、Device Diagnostics)、RTC、Reset、Flash Downloader、Administration (Update License、Master Reset)。

### 功率計算模式
- S=f(P,Q)：S=√(P²+Q²)，適用低諧波 (V THD<5%、I THD<10%)。
- Q=f(S,P)：S=V×I，N=√(S²−P²) (非有功功率)，適用高諧波。

### 費率 (Tariff) 控制
TOU 行事曆自動、通訊寫入費率暫存器、或 1–3 個連續 DI 輸入費率碼 (2 費率需 1 個 DI、3–4 需 2 個、5–8 需 3 個)。

## 6. 數位輸入 / 繼電器
- DI：極性 NORMAL/INVERTING、Pulse/KYZ 模式、**消彈跳 1–100ms，以 8 個為一組共用** (DI1–8、DI9–16…)、勾 SOE Log、勾 Fault Recorder (開→閉觸發 External Trigger)。
- 繼電器：Unlatched / Latched / Pulse / KYZ；Polarity Normal 或 Inverting (failsafe)；Retentive (僅 latched)；脈衝寬 10–1000ms (½ 週波倍數)；Pulse source (kWh IMP/EXP/TOT、kvarh、kVAh、需量區間…)；kWh/Pulse；SOE Log；Block。
- 不存在的繼電器 = 虛擬繼電器，可當 setpoint 之間的旗標使用。共 16 (到 64) 個 Event flags 可遠端設定。
- 嚴重錯誤時所有繼電器釋放、遠端命令清除。

## 7. Control Setpoints (內建邏輯控制器)
- 64 組 setpoint，每組最多 4 個觸發條件 (OR/AND，**無優先順序，依序覆蓋**，建議同類運算子集中)、最多 4 個動作，掃描週期 ½ 週波。
- 類比觸發用 Operate limit + Release limit (遲滯，一次側單位)；也可 Delta、rDelta。
- 延遲：Operate / Release delay 0–10000 秒 (脈衝型事件不能用延遲)。
- 動作：繼電器動作/釋放、計數器、Data log、Waveform log、Event log、SOE、EXT TRIGGER (UDP 跨設備觸發 16/64 通道)、Send notification (TCP 通知客戶端)、CLOSE/OPEN SWITCH、REMOTE CONTROL 等 (見 Operation Manual Appendix B，觸發參數清單見 Appendix C)。
- 大型邏輯可用 PLC Configurator (FBD/LD) 自動分配到 setpoint，見 `10-plc-configurator.md`。

## 8. 錄波器 (Chapter 7)
- 256MB 需先分割給 28 個檔：Event log、16 Data logs、8 Waveform logs、SOE、PQ log、Fault log。Wrap-around 或 Non-wrap。
- 出廠預設：
  - Data log #1：1 週波 V1–V3、I1–I3、kW、kvar、kVA、PF、V/I THD (16 參數)；#2：需量、kWh/kvarh/kVAh、I4、V4。由 Setpoint #1 每 15 分鐘觸發。
  - Data log #9/#10：EN50160 合規統計與諧波調查 (EN50160 版本)。
  - Data log #11：IEC 61000-4-30 10 分鐘 profile (1345 參數)。
  - Data log #13 故障 RMS 趨勢、#14 PQ RMS 趨勢；#15/#16 TOU 月/日 profile。
- Waveform：8 個檔，取樣率 32/64/128/256 spc，最多 20 週波觸發前；每筆記錄 512 點。`每事件筆數 = spc × 週波數 / 512`。
- PQ recorder：IEEE 1159 (標準) / EN 50160 / GOST，自動分類事件、ITIC 曲線、擾動方向。
- Fault recorder：內建故障偵測器 (各觸發需個別啟用，門檻 % of 額定、遲滯預設 5%) 或任一 DI 外部觸發。
- Fault Locator：需輸入線路型式、長度、正/零序阻抗 (Ω/km，最多 4 段)、保護/斷路器 DI…
- PAS 檢視：Event log、SOE、PQ log、ITIC、IEEE1159/EN50160 報表、Fault log、Data log 趨勢、Waveform (RMS/頻率/頻譜)、同步波形。
- 匯出 COMTRADE / PQDIF：工具列 Export，或 Upload Scheduler 自動轉檔。

## 9. 維護
- **清除/重置**：PAS Monitor → Reset (能量、需量、Min/Max、計數器、檔案)；RDM rESEt 選單；Modbus 44102–44108。
- **校時**：Monitor → RTC → Set (有 GPS 時不需要)。
- **韌體升級**：只支援 Modbus RTU/ASCII/TCP；序列埠建議 115200。On-line → Monitor → Flash Downloader → 選檔 → 輸入密碼。USB 約 20 秒、115200 序列約 9–10 分鐘；完成後約 10 秒燒錄重啟。Cellular 臨時 IP 可能改變。
- **61850 授權**：Monitor → Administration → Update License。重置 61850 設定：Administration → Master Reset → Reset IEC 61850 Configuration。

## 10. 診斷碼 (Appendix F)
| 碼 | 說明 | 類別/原因 |
|---|---|---|
| 0 | 不可恢復系統失效 | 嚴重 |
| 1 | 永久故障 (反覆失效) | 嚴重 |
| 2 | 記憶體/資料錯誤 | 硬體 |
| 3 | 硬體看門狗重置 | 硬體 |
| 4 | DSP/取樣故障 | 硬體 |
| 5 | CPU exception | 硬體 |
| 7 | 軟體看門狗逾時 | 硬體 |
| 8 | 斷電/上電 | 失電 |
| 9 | 暖啟動/重置 | 通訊外部重啟或韌體升級 |
| 10 | 設定重置 | 設定損毀已以預設值取代 → 查 Event log 確認哪個設定被重置 |
| 11 | RTC 故障 (嚴重) | 時間遺失 |
| 12 | 設定故障 (嚴重) | 原廠/校正/基本設定損毀 |
| 14 | 擴充記憶體故障 | 硬體 |
| 15/16/17 | CPU / AC 板 / I/O 板 EEPROM 故障 | 硬體 |
| 20/21/22 | C library / RTOS kernel / Task 錯誤 | 硬體/韌體 |
| 24 | IRIG-B 訊號遺失 | 偵測到訊號自動清除 |
| 25 | IRIG-B 時間未鎖定 (GPS 失去衛星) | 鎖定後自動清除 |
- 三類：(1) 非嚴重自動重置 (如 IRIG-B)；(2) 非嚴重需手動清除 (通常伴隨重啟與設定修復)；(3) 嚴重 — 釋放所有輸出、停止運作直到清除。
- 檢視/清除：PAS Monitor → Device Control → Device Diagnostics → Clear；RDM StA 診斷頁；Telnet `diag`/`clrdiag`；Modbus 44326-44327 (寫 0 清除)。
- "DEVICE FAULT" 事件旗標可用 setpoint 驅動 failsafe 告警繼電器。
- **IRIG-B 告警無法遮蔽** → 沒接 IRIG-B 就把同步來源改成未用的 DI 或 SNTP。
