# 02 安裝與接線攻略

來源：PM180-Installation-Manual (BG0521)。**PDF 頁碼 = 手冊印刷頁碼 + 4**。接線圖都是圖片，需開原始 PDF 看圖 (位置見 SKILL.md 的 PM180_SOURCE_DIR → `02_硬體安裝/PM180-Installation-Manual.pdf`)。

## 圖號 → PDF 頁碼對照
| 內容 | 圖號 | PDF 頁 |
|---|---|---|
| 外形尺寸 / 盤面安裝 / DIN 軌 | 2-1, 2-2 / 2-3 | 8 / 9 |
| 3DIR2 三線直接 2CT | 2-4 | 11 |
| 4LL3/4LN3 四線 Y 直接 4CT；3PT 4CT (內部 CT) | 2-5, 2-6 | 12 |
| 4LL3/4LN3 3PT 3CT 外部 HACS 100A (CS1S)；3OP2 開口三角 2PT 2CT | 2-7, 2-8 | 13 |
| 3LL3/3LN3 四線 Y 2PT 3CT；3OP3 開口三角 2PT 3CT | 2-9, 2-10 | 14 |
| 4LL3/4LN3 四線三角直接 3CT | 2-11 | 15 |
| DI 乾接點 / 濕接點 | 2-12, 2-13 | 16 |
| 繼電器 / 8DI+4RO | 2-14, 2-15 | 17 |
| 4AI/AO | 2-16 | 18 |
| TRM 暫態模組 | 2-17 | 19 |
| PMU (MV 線路監測範例) | 2-18 | 20 |
| DFR 故障錄波 | 2-19 | 21 |
| RS-485 (舊 CPU) / RS-485 (N3) + IRIG-B / USB / ETH | 2-20 / 2-21, 2-22, 2-23 | 22 / 23 |
| ETH RJ45 & SC、Cellular SMA | 2-23, 2-24 | 24 |
| 模組位置 (Slot) | 2-25 | 25 |
| 序列埠位置、RS-232 PC/數據機、RS-485/422 多點 | 3-1 ~ 3-8 | 26–30 |
| 更換電池 | Ch4 | 31 |

## 安裝前必讀
1. **出廠不含電池**，安裝時要裝 CR1225 3V 鋰電池 (CPU 面板電池座，一字起子拉出，注意極性，推到「喀」一聲)。
2. **裝電池後首次上電初始化約 7 分鐘**，之後正常開機約 40 秒 — 不要以為當機。
3. 電池可帶電更換；斷電狀態下 30 分鐘內換好不會遺失資料。電池低時 Event log 記錄 BATTERY LOW、前面板 MEM.BAT.LOW 紅燈。

## 電源接線
- 主電源：VOLTAGES 板端子 L/+ (火線/正)、N/- (中性/負)。50-290VAC 或 90-290VDC。
- 輔助備援電源 (AUX. P.S. 模組)：同樣 L/+、N/-。
- 線徑：電源 1.5-2.5 mm² (15-13 AWG)；電流 2.5-6 mm² (10 AWG)。
- 接地：機殼接地到盤體接地，> 2 mm² (14 AWG) 專用線。

## 電壓 / 電流輸入
- V1/V2/V3/VN：Y 接，標準 480V L-L (N1 型 690V L-L)；V4/Vref：獨立隔離 AC/DC 400V，可接站用直流電池 (VDC 額定預設 125V，範圍 10-300V)。
- I1–I4：內部 CT 5A (標準) 或 1A；或外接 SATEC HACS 100A 分裂式 CT (CS1S，40mA 二次)。端子間距 9.5mm。
  - 5A 輸入：連續 10A/20A (IEC/ANSI)，故障量測至 50A (10×In)，過載 50A 連續、250A/1s。
  - 1A 輸入：連續 2A(IEC)/4A(ANSI)，故障至 10A。
  - HACS 100A：故障至 100A (20×In)。
- DFR 模組 I1x–I4x：接保護 CT，HACS 200A → 200A@In=5A；HACS 100A → 40A@In=1A。

## 接線模式 (Basic Setup 的 Wiring Mode)
| 代碼 | 說明 | 圖 |
|---|---|---|
| 3OP2 | 三線開口三角，2PT 2CT (2 元件) | 2-8 |
| 4LN3 (預設) | 四線 Y，3PT 3 元件，讀相電壓 | 2-5/6/7/11 |
| 3DIR2 | 三線直接連接，2CT | 2-4 |
| 4LL3 | 四線 Y，3PT 3 元件，讀線電壓 | 2-5/6/7/11 |
| 3OP3 | 三線開口三角，2PT 3CT (2½ 元件) | 2-10 |
| 3LN3 / 3LL3 | 四線 Y，2PT 3CT (2½ 元件) 讀相/線電壓 | 2-9 |
| 3BLN3 / 3BLL3 | 三線 Broken Delta，2PT 3CT | — |
- 4LN3/3LN3/3BLN3 → 電壓、波形、諧波、Min/Max、需量為**相電壓**；其他模式為**線電壓**。
- 另有 Line Monitor 電流板，可直接接 MV 桿上電壓/電流感測器 (§2.2.5.2)。

## 數位輸入 (DI16)
- 每片 16 DI，光耦隔離，**所有 COM 端子內部相連**。乾接點 (內部 24VDC，開路 >16kΩ、閉合 <10kΩ) 或濕接點 24/48/125/250VDC。
- 端子間距 3.81mm、線徑至 2.5mm²。
- 用途：狀態監視、需量週期同步、時間同步 (分鐘脈衝)、SOE、故障錄波觸發、脈衝計數、TOU 費率輸入。

## 繼電器 (RLY8) 與 8DI/4RO
- 8 個 SPST Form A。8A@250VAC、5A@30VDC、0.25A@250VDC、5A@24VDC。更新 ½ 週波。
- 用途：告警、遙控、電能脈衝 (KYZ)。
- 8DI/4RO 範例 (圖 2-15) 為 125VDC 濕接點 + 10A 保險絲。

## 類比 I/O (4AI/AO)
- 4 AI + 4 AO，內建隔離電源。0-20/4-20mA (迴路負載 ≤500Ω/510Ω)、0-1mA/±1mA (≤10kΩ，200% 過載)。±10V 選項只有 4AI 模組。

## TRM / PMU / DFR
- TRM：接 V1/V2/V3/VN (+VG 對地)，量測至 2kV 對地暫態，自動偵測免設定。
- PMU：電壓接 PMU 模組 V1/V2/V3/VN，電流由背板取自 AC 輸入；IRIG-B 接 CPU 模組 1pps/CM 端子；或用 PMU 的 IEEE 1588 網口 (需經 P2P 透明時鐘交換器接 PTP grandmaster)。**相序必須正確 (ABC)，CBA 要在 Basic Setup 改相序，否則量不到頻率**。前面板 "PMU OK" LED：黃色每秒閃 = 未同步、綠色閃 = 已同步。
- DFR：從保護 CT 迴路以 HACS 夾在 I1x–I4x。

## 序列通訊埠
| 埠 | 位置 | 介面 | 備註 |
|---|---|---|---|
| COM1 | CPU 模組 | RS-232/485 (N3 版僅 RS-485)，4 端子 | 可直連 PC RS-232 (3 線)；數據機初始化字串 `ATS0=1&D0&K0&W0` |
| COM2 | 通訊模組 (IRIG-B、Cellular、TXFX…) | RS-422/485，5 端子 | 僅 8N1；多片通訊模組只有一個 COM2 |
| COM3 | 顯示器埠 DB15 | RS-485 | 接 RDM/RGM；**DB15 pin 1、8 輸出 12VDC，RS-485 線誤接會燒埠**；不用顯示器時可當一般 RS-485 |
| COM4 | RGM180 上的 IR 光學埠 | IR | |
| COM5 | Cellular/Modem/Profibus 模組內部 | — | |
- 全部光耦隔離，最高 115200 bps，RS-485 最長 1000m。
- 模組上 COM2 的切換：Open = RS-485、Closed = RS-232 (圖 2-25 標示)。

## USB / 乙太網路 / 行動網路
- USB 1.1 隔離，Modbus RTU，速度約序列埠 10 倍，需安裝 PAS 光碟內驅動。
- **電表端是 USB-A 母座 (跟電腦的孔一樣，現場確認)** → 用 **USB A 公對 A 公直通線** (≤2m)；不要用中間有晶片的「對拷線 / Data Link」。電表先上電再插線。
- ETH (CPU)：10/100Base-T RJ45，Modbus/TCP 502、DNP3/TCP 20000、IEC 104 2404、IEC 61850，5 個 socket。
- ETH2 (TXFX 模組)：10/100TX RJ45 + 100FX SC 光纖，主網路斷線時自動切換備援。
- Cellular 2G/3G：插 SIM、SMA 天線，Modbus/TCP、DNP3/TCP。

## IRIG-B
- 非調變 (pulse-width coded, DCLS)、非平衡 5V，支援 B000/B001/B004/B005 (IRIG 200-04)。
- N3 版 CPU 端子 (可拆式 4 端子) 或通訊模組 BNC。建議線材 RG58A/U 51Ω (Belden 8219)，建議 GPS 時鐘 Masterclock GPS-200A。

## 安裝完成檢查清單
1. 電池已裝、首次上電等 7 分鐘、CPU LED 1 秒亮 1 秒滅 (正常運轉)。2 閃 = Service mode，3 閃 = 嚴重錯誤 (Diagnostics mode)。
2. MAIN POWER / BACKUP POWER 綠燈。
3. 用 PAS/RDM 設定 Wiring mode、PT 比、PT 二次側、CT 一次側、頻率、相序 (先設這些再設 setpoint 與錄波觸發)。
4. 檢查相序 (RDM Status 頁 `Ph.rOt` POS/NEG/ERR)、功率方向與 PF 合理。
5. 設定通訊埠/IP，從 SCADA 端讀 1 秒值驗證。
6. 時間同步來源 (IRIG-B/SNTP/DI)；沒有 IRIG-B 就不要留在 IRIG-B。
