# 04 Modbus 攻略 (RTU / ASCII / TCP)

來源：PM180-Modbus (BG0527 Modbus Reference Guide)。原文：`fulltext/PM180-Modbus.txt` (每列一行，可直接 Grep 區塊標題，例如 `Grep "^14336-14363" -A 16`)。

## 1. 協定實作
- 模式：RTU、ASCII (序列埠)、Modbus/TCP (port 502)。站號 1–247，不支援廣播 (0)；TCP 不檢查站號。
- 功能碼：**03 / 04 (同義，讀多暫存器)**、06 (寫單一)、16 (寫多)、22 (Mask write)、08 (loop-back，只支援子碼 0)。
- **一次最多 120 個暫存器** (03/04/16)，超過回例外 02。
- 例外碼：01 非法功能 (密碼保護時寫入也回 01)、02 非法位址、03 非法值、04 設備故障。
- 位址：手冊位址為 0-based **PDU 位址**。Modicon 表示法：≤9999 加 40001 (例 256 → 40257)；>9999 用 6 位數加 400001 (例 14336 → 414337)。
- TCP socket 閒置 5 分鐘自動關閉；每個 socket 獨立檔案指標。

## 2. 資料格式 (最容易出錯)
### 2.1 32-bit 整數 (建議 SCADA 一律用這組)
- 兩個連續暫存器，**第一個 = 低 16 位 (low word)，第二個 = 高 16 位**，低字永遠在偶數位址。
  - 多數 SCADA/PLC/Gateway 預設 Big-endian word (ABCD)，PM180 是 **CDAB (word swap / little-endian word order)** — 要開 word swap。
- UINT32 或 INT32 (kW、kvar、PF、kWh net 等有號)。換算：`value = (signed)high × 65536 + (unsigned)low`。
  - 例：V1 69000V → (13952)=3464、(13953)=1 → 1×65536+3464=69000。
  - 例：-789 kW → (14336)=64747、(14337)=65535 → -1×65536+64747 = -789。
- 小數以 10 的次方放大傳送，乘上表中單位 (例 頻率 5001 × 0.01Hz = 50.01Hz)。
- **非 IEEE float** (除 Synchrophasor 與 External Float 區塊為 FLOAT32)。

### 2.2 單位 (依 PT 比而變！)
| 代碼 | PT=1 | PT>1 |
|---|---|---|
| U1 電壓 | 0.1 V | 1 V |
| U2 電流 | 0.01 A | 0.01 A |
| U3 功率 | 1 W / var / VA | 1 kW / kvar / kVA |
| U4 V4 電壓 | 0.1 V (V4 PT=1) | 1 V |
| U5 能量 | 0.001/0.01/0.1/1 kWh (依 Energy Decimal Places，預設 0 → 1 kWh) | 同左 |
- PF 0.001；THD/TDD/不平衡 0.1%；諧波 0.01%；頻率 0.01Hz (另有 0.001、0.0001Hz 暫存器)；角度 0.1° (相量 0.001°)；溫度 0.1°C。
- **PT=1 (直接接線) 時功率單位是 W，PT>1 時是 kW** — 切換 PT 比後 SCADA 縮放要跟著改。

### 2.3 Scale (16-bit 縮放與範圍上限)
- Vmax = Voltage Scale (預設 828V，建議：經 PT 用 144V、直接接 828V) × PT 比
- Imax = Current Scale (預設 ANSI 4×CT 二次 = 20A；IEC 2×CT 二次 = 10A) × CT 比
- Ixmax = 100 × CT 比 (5A, CT 卡) / 200 × CT 比 (5A, DFR)；1A 時 20× / 40×
- Pmax = Vmax × Imax × 2 (PT=1 單位 W，上限 9,999,000W)；PT>1 時 /1000 kW
- 暫存器 242 = Voltage scale (1V，二次側)、243 = Current scale (0.1A)。

### 2.4 16-bit 縮放格式 (256–308、4320–10751)
- 輸出 0–9999 (raw low/high 由 240/241 設定)。還原：`工程值 = raw × (ENG_HI − ENG_LO) / (RAW_HI − RAW_LO) + ENG_LO`
- 例：PT=1、raw 1449 → 1449×828/9999 = 120.0V；CT 200A、400% → Imax=800A，raw 250 → 20.00A；功率 ±Pmax 雙向，raw 5500 → 132.6 kW。
- PF：ENG −1.000~1.000，raw 8900 → 0.78。
- 精度較差，**新案建議改讀 32-bit 區**。

### 2.5 Modulo-10000 能量 (16-bit 區 287–294、301–302)
`能量 = 高字 × 10000 + 低字`。使用時能量 roll 值要限 8 位數以免溢位。

## 3. 常用 32-bit 暫存器速查 (Section 3.4)
### 1-Second Phase Values 13952–14029
| 位址 | 參數 | 型別 | 單位 |
|---|---|---|---|
| 13952 / 13954 / 13956 | V1 / V2 / V3 (4LN3 等為相電壓) | UINT32 | U1 |
| 13958 / 13960 / 13962 | I1 / I2 / I3 | UINT32 | U2 |
| 13964 / 13966 / 13968 | kW L1 / L2 / L3 | INT32 | U3 |
| 13970 / 13972 / 13974 | kvar L1 / L2 / L3 | INT32 | U3 |
| 13976 / 13978 / 13980 | kVA L1 / L2 / L3 | UINT32 | U3 |
| 13982 / 13984 / 13986 | PF L1 / L2 / L3 | INT32 | 0.001 |
| 13988 / 13990 / 13992 | V1 / V2 / V3 THD | UINT32 | 0.1% |
| 13994 / 13996 / 13998 | I1 / I2 / I3 THD | UINT32 | 0.1% |
| 14000–14004 | I1–I3 K-Factor | UINT32 | 0.1 |
| 14006–14010 | I1–I3 TDD | UINT32 | 0.1% |
| 14012 / 14014 / 14016 | V12 / V23 / V31 | UINT32 | U1 |
| 14018–14022 | I1x–I3x (擴展/故障量程) | UINT32 | U2 |

### 1-Second Total Values 14336–14363
| 14336 Total kW (INT32) | 14338 Total kvar (INT32) | 14340 Total kVA | 14342 Total PF (INT32, 0.001) |
|---|---|---|---|
| 14344 PF lag | 14346 PF lead | 14348 kW import | 14350 kW export |
| 14352 kvar import | 14354 kvar export | 14356 3相平均 L-N V | 14358 3相平均 L-L V |
| 14360 3相平均電流 | 14362 平均 Ix | | |

### 1-Second Auxiliary Values 14464–14497
14464 I4、14466 In (中性線)、**14468 頻率 (0.01Hz)**、14470 電壓不平衡 (0.1%)、14472 電流不平衡、14476 V4、14478 I4x、14480 V4x (中性-地)、14482 內部溫度 (INT32 0.1°C)、14484 頻率 (0.001Hz)、14486 鋰電池電壓 (0.001V)、14490 頻率 (0.0001Hz)、14496 Posc%。

### Present Demands 14592–14661
14592–14596 V1–V3 需量、14598–14602 I1–I3 需量、14604 kW import block、14606 kvar import block、14608 kVA block、14610 kW import 滑動、14612 kvar import 滑動、14614 kVA 滑動、14622 kW import accumulated、14628 kW import predicted…、14656 V4、14658 I4、14660 In 需量。

### Total Energies 14720–14741 (U5)
| 14720 kWh import | 14722 kWh export | 14724 kWh net (INT32) | 14726 kWh total |
|---|---|---|---|
| 14728 kvarh import | 14730 kvarh export | 14732 kvarh net (INT32) | 14734 kvarh total |
| 14736 kVAh total | 14738 Vh total | 14740 Ah total | (+22.. kVAh imp/exp、kvarh Q1–Q4) |
- Phase Energies 14848–14889 (每相 kWh/kvarh/kVAh)。Billing Summary 14784–14815 (16 個總和暫存器)。

### Maximum Demands 18816–18859
18816–18820 V1–V3 最大電壓需量、18822–18826 I1–I3 最大電流需量、18834 最大 kW import 滑動需量、18836 kvar import、18838 kVA、18846 kW export、18848 kvar export、18854 V4、18856 I4、18858 In。

### 其他區塊位址索引 (查詳細內容請 Grep fulltext)
| 區塊 | 位址 |
|---|---|
| Setpoint status (alias / 1-32 / 33-64) | 11840 / 27648 / 27712 |
| Special inputs (電壓擾動、相序 0=錯 1=ABC 2=CBA) | 11904–11907 |
| Event flags 1-32 / 33-64 | 12160 / 12224 |
| Digital inputs DI1-32 / 33-64 / 65-96 / 97-128 (bitmap) | 12544 / 12608 / 12672 / 12736 |
| Relay outputs RO1-32 / 33-64 (bitmap) | 12800 / 12864 |
| Static event flags (bit0 相序錯…bit7 設備故障、bit8 無電壓、bit9 遠端控制) | 12928 |
| Counters #1–32 | 13056–13119 |
| Time/Date | 13184–13207 |
| 1/2-Cycle Values | 13248–13309 |
| 1-Cycle Phase / Total / Aux | 13312 / 13696 / 13824 |
| Fundamental Phasor (幅值/角度 0.1°) | 13888–13935 |
| 3-Second Powers / 10-Minute Powers | 14144 / 14272 |
| Harmonic demands | 14528 |
| Symmetrical components | 14912–14937 |
| V1/V2/V3 諧波 % (H01–H63, 0.01%) | 14976 / 15104 / 15232 (V4 16896) |
| I1/I2/I3 諧波 % | 15360 / 15488 / 15616 (I4 17280) |
| V1/V2/V3 諧波電壓 (工程值) | 15744 / 15872 / 16000 |
| I1/I2/I3 諧波電流 | 16128 / 16256 / 16384 |
| Flicker | 17088–17105 |
| Min 1-Cycle Phase / Total / Aux | 17408 / 17536 / 17664 |
| Max 1-Cycle Phase / Total / Aux | 18432 / 18560 / 18688 |
| TOU Parameters (費率) | 19456–19459 |
| Billing TOU Register #1–#8 | 19584 起每 128 (#9–#16 自 21376) |
| 0.2s / 3s / 10min / 2h RMS (PQ 統計聚合) | 25600 / 25664 / 25728 / 25792 |
| Present PQ Measurements (IEEE1159/EN50160/GOST) | 27136–27303 |
| GOOSE 外部量 ExtiVal / ExtfVal (FLOAT32) / ExtInd | 28160 / 28288 / 28416–28609 |
| 開關位置 Pos / OpOpn / OpCls / 操作次數 | 29440 / 29568 / 29632 / 29696 |
| Synchrophasor (PMU，含 FLOAT32 Re/Im) | 29824–29903 |
| Distance to fault | 30080–30093 |
| Min/Max Log (含時間戳) | 35840 起 (Section 3.5) |

## 4. 控制與狀態暫存器 (Section 3.6)
| 位址 | 功能 |
|---|---|
| 44032 / 44034 / 44036 | Event flags 1-32 設定 (1=set) / 清除 (0=clear) / 狀態 |
| 44046–44049 | 強制繼電器動作 (UINT64 bitmap，1=operate) |
| 44050–44053 | 強制繼電器釋放 (1=release) |
| 44066–44069 | 繼電器狀態 (1=closed) |
| 44102 | 清除能量 (寫 0) |
| 44103 | 清除最大需量 (0=全部,1=功率,2=V&A,3=V,4=A,5=諧波) |
| 44104 / 44105 | 清除 TOU 能量 / TOU 最大需量 |
| 44106 | 清除計數器 (0=全部, 1–32=個別) |
| 44107 | 清除 Min/Max |
| 44108 | 清除統計 (0=EN50160/GOST、6=通訊計數、7=開關操作次數) |
| 44135 / 44136 / 44137 | Setpoint 運作 / PQ recorder / Fault recorder 啟用 (0/1) |
| 44166 | UDP 跨設備觸發 index |
| 44262 / 44264 | 記憶體大小 / 剩餘 |
| 44278 | 檔案新紀錄通知 bitmap |
| 44294 / 44310 | Setpoint 1-64 狀態 / 告警閂鎖 (寫 0 清除) |
| **44326–44327** | **診斷旗標 bitmap (寫 0 清除)** |
| 44338 / 44340 | GOOSE 訂閱狀態 #32–64 / #1–31 |
| 44342 / 44343 | 目前連線埠號 / 介面 |
| 44346–44377 | 網路識別：IP、遮罩、閘道 (network byte order)、GPRS IP、IMEI、PMU 網路 |
| **44378–44379** | **授權暫存器：寫入 8 位數密碼；讀 0=允許、-1=需授權** |
| 44442 / 44450 | 使用者名稱 / 密碼 (增強安全版本) |
| 44394 / 44395 | RSSI / GPRS 狀態 |
| 44458 / 44460 | 開關 ON(合) / OFF(分) 命令 bitmap (bit0 XCBR1, bit1 XCBR2, bit2–15 XSWI1–14) |

## 5. 設定暫存器 (Section 3.7)
| 位址 | 內容 |
|---|---|
| 46080– | 設備識別：+0 序號、+2 型號 ID 18000、+4 "PM180"、+20 韌體版本 (3101–3199)、+21 build、+26 PMU 韌體 |
| 46112– | 原廠設定：輸入範圍、過載 (400=ANSI/200=IEC)、MAC |
| 46208– | Basic Setup：+0 接線模式 (F26)、+1 PT 比 (×0.1)、+2 PT 二次 (×0.1V)、+3/+4 V4、+5 CT 一次 (A)、+7 I4 CT、+17 頻率、+18 相序 (0=ABC,1=CBA)、+24 最大負載電流 |
| 46240– | 需量設定 (+0 功率需量週期分鐘、+1 滑動數、+2 同步來源、+8/+9/+10 V/A/H 秒) |
| 46256– | Device options：+0 功率計算模式、+1 能量 roll (0=10^4 … 5=10^9)、+2 能量小數位 0–3、+4 費率控制、+6 PQ 選項 (2=EN50160:2007, 5=:2010)、+14 區間能量週期、+15 IEC 61000-4-30 10 分鐘 profile |
| 46400– | 時間：+0 時區分鐘 (台灣 480)、+1 DST (0=停用)、+8 同步來源 (0=IRIG-B、0x0001–0x0080=DI 1PPM、0x1001–=DI 1PPS、0x7FFD=PTP、0x7FFE=SNTP、0x7FFF=無) |
| 46416– | 時鐘：+0,1 本地時間 UNIX 秒 (可寫入校時) |
| 46448– | 通訊埠：每埠 16 個 (COM1 46448、COM2 46464、COM3 46480、COM4 46496、COM5 46512)；+0 協定 (0=RTU,1=ASCII,2=DNP3,7=IEC60870-5)、+1 介面 (0=232,1=422,2=485)、+2 位址、+3 鮑率 (6=9600,7=19200,8=38400,9=57600,10=115200)、+4 格式 (0=7E,1=8N,2=8E)、+7 回應延遲 (預設 5ms)、+8 字元逾時 (預設 4ms) |
| 46560– | PMU 網路 (IEEE 1588 口) |
| 46576– | 設備網路：+0,1 IP、+2,3 遮罩、+4,5 閘道 (network byte order)、+14 應用協定閒置逾時、+20 TCP keepalive |
- 接線模式代碼 F26：0=3OP2、1=4LN3、2=3DIR2、3=4LL3、4=3OP3、5=3LN3、6=3LL3、8=3BLN3、9=3BLL3。

## 6. User Assignable Registers (把散落的點集中成一次讀)
- 0–119 = 可指派暫存器，120–239 = 對應表 (寫入實際位址)。120 個上限、32-bit 須對齊偶數。
- 例：讀 kWh import + 16-bit 1 秒 V1 → 寫 120=14720、121=14721、122=4672，之後讀 0–2。
- PAS：Protocol Setup → MODBUS Registers → Default (先指向 11776 "None") → 填位址 → Send。

## 7. 密碼保護寫入流程
1. 寫密碼到 44378–44379 (INT32)。
2. 執行寫入 (設定、重置、繼電器控制)。
3. 寫 0 到 44378 重新上鎖 (建議)。
- 未授權寫入 → 例外碼 01。

## 8. 檔案 (Log) 傳輸
- 32 字的請求區 + 1792 字的回應區；另有 8 字 info 請求/200 字 info 回應。
- 流程：Reset/Set-File-Position → Read-File → 讀回應區 → Acknowledge (指標前進) → 重複；檢查 record status bit 9 (EOF)。File ID：0 Event、1–16 Data log、17–24 Waveform、25 SOE、26 PQ、27 Fault、128 即時波形。
- 一般 SCADA 不建議自己實作，用 PAS Upload Scheduler 取資料再匯出 COMTRADE/PQDIF。

## 9. TCP Notification Client
- 設備主動以 FC16 寫 24 個暫存器到遠端 Modbus/TCP 伺服器 (事件/定時通知)，由 setpoint 的 "Send notification" 動作觸發；PAS → Communication Setup → TCP Notification Client Setup。

## 10. SCADA 點表範例 (建議最小集合，Modbus TCP, FC03, 32-bit CDAB)
| 點名 | 位址 (0-based) | 型別 | 換算 (PT>1) |
|---|---|---|---|
| Va / Vb / Vc | 13952 / 13954 / 13956 | UINT32 | ×1 V |
| Vab / Vbc / Vca | 14012 / 14014 / 14016 | UINT32 | ×1 V |
| Ia / Ib / Ic / In | 13958 / 13960 / 13962 / 14466 | UINT32 | ×0.01 A |
| P / Q / S | 14336 / 14338 / 14340 | INT32/INT32/UINT32 | ×1 kW/kvar/kVA |
| PF | 14342 | INT32 | ×0.001 |
| Hz | 14468 | UINT32 | ×0.01 |
| kWh imp / exp | 14720 / 14722 | UINT32 | ×1 kWh (小數位 0) |
| kvarh imp / exp | 14728 / 14730 | UINT32 | ×1 kvarh |
| V THD a/b/c | 13988 / 13990 / 13992 | UINT32 | ×0.1 % |
| I THD a/b/c | 13994 / 13996 / 13998 | UINT32 | ×0.1 % |
| DI1–32 / RO1–32 | 12544 / 12800 | UINT32 bitmap | bit n = DI(n+1) |
| 診斷旗標 | 44326 | UINT32 | 非 0 = 有診斷事件 |
- 一次讀取區塊：13952 起 66 個暫存器 (到 14017) 可涵蓋 V/I/P/Q/S/PF/THD/線電壓；14336 起 28 個；14464 起 10 個；14720 起 18 個。
- PT=1 (低壓直接接線)：電壓 ×0.1 V、功率 ×1 W (要 kW 再 ÷1000)。

## 11. Python (pymodbus 3.x) 讀取範例
```python
from pymodbus.client import ModbusTcpClient
c = ModbusTcpClient("192.168.0.203", port=502); c.connect()
def u32(regs, i): return regs[i] | (regs[i+1] << 16)          # low word first
def s32(regs, i): v = u32(regs, i); return v - (1 << 32) if v & 0x80000000 else v
r = c.read_holding_registers(13952, count=66, slave=1).registers   # pymodbus ≥3.10 改用 device_id=1
va, ia = u32(r, 0), u32(r, 6) * 0.01                         # PT>1: V 為 1V 單位
t = c.read_holding_registers(14336, count=8, slave=1).registers   # pymodbus ≥3.10 改用 device_id=1
kw, pf = s32(t, 0), s32(t, 6) * 0.001
```
