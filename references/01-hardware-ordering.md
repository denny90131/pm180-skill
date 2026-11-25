# 01 硬體概觀、型號、模組與訂購碼

來源：PM180_DATASHEET (2024/07)、PM180-Brochure-2018 (BB0160)、PM180-Installation-Manual (BG0521)、PM180-Operation-Manual Ch1 (BG0525)、EDL180_Manual (BG0647)

## 產品定位
- SATEC expertmeter PM180：Class A (IEC 61000-4-30 Ed.3 2015) 電力品質分析儀 + 0.2S 計量表 + 故障錄波器 (DFR) + SOE + PMU + Bay Controller + 可程式控制器，模組化熱插拔。
- 應用：數位變電站 (IEC 61850)、故障錄波 (至 40×In)、PMU/WAMS (C37.118.1 P/M class)、擾動方向偵測、BCU、SCADA。

## 量測與準確度重點
- 4 路隔離電壓 (V1–V3 + V4/Vref 可量測直流電池)、4 路隔離電流 (I1–I4)。
- 取樣：8 通道 256 點/週波；TRM 模組 512/1024 點/週波；DI 16 點/週波 (時間戳 1ms@60Hz / 1.25ms@50Hz)。
- 有功電能 Class 0.2S (IEC 62053-22) / 0.2 (ANSI C12.20)；無功 0.5S。(CS1S 型為 Class 1)
- 諧波到 63 次 (IEC 61000-4-7)、間諧波、閃爍 (IEC 61000-4-15)、方向性諧波功率。
- 電流雙量程：標準量程 20A/10A (精密量測)、擴展量程 100A (故障)；DFR 模組可到 200A (40×In@5A)。
- 電壓：直接量測至 690V L-L (828V 上限)，經 PT；V4 輔助 AC/DC 400V (DC 300V 範圍，通常接站用電池)。
- 記憶體 256MB (約 250MB 可用)；RTC 每月漂移 ≤5 秒@25°C。
- 頻率解析度 3 位小數 (另有 4 位小數暫存器)。

## 韌體版本代碼 (判斷功能選項)
`V31.AB.XX`：
- B (PQ 標準)：4=IEEE 1159、5=EN 50160、7=GOST 13109-97、8=GOST 32144-2013
- A (61850)：0=無、1=IEC 61850 (Ed1)、2=IEC 61850 Ed2
- 例：V31.14.XX = IEEE1159 + 61850；V31.25.XX = EN50160 + 61850 Ed2
- V31.X5.28 起 EN 50160 可選 2007 或 2010 版。
- 功能最低韌體：TXFX ≥ V31.XX.8、TRM ≥ V31.XX.10、DFR ≥ V31.XX.11、Cellular ≥ V31.XX.15、8DI/4RO ≥ V31.XX.28、PMU ≥ V31.XX.29 (C37.118.2 需 ≥ V31.XX.31)、Bay Control ≥ 20.XX.07 (BCU app note 記載)。
- 讀取：Modbus 46080+20 (韌體版本 3101–3199)、+21 (build)；Telnet 指令 `I`。

## 硬體版本
- **N3 版**：新 CPU 板 (AP1451)，CPU 上內建 IRIG-B 端子 (1pps/CM)，COM1 只支援 RS-485。PMU 只能用在 N3 新 CPU。
- PM180-N1：690V 輸入、1MΩ 阻抗。

## 槽位 (Slot) 配置
共 6 槽：3 個固定 (VOLTAGES、CPU、CURRENTS) + 3 個可熱插拔選配槽 (Slot #1/#2/#3)。
規則 (重要)：
- **TXFX、Cellular(2G/3G) 必須插在 Slot #1** (緊鄰 CPU)，且各只能 1 片。
- TRM、DFR、PMU 各只能 1 片；TRM/DFR 可放任一選配槽。PMU 與 TRM 為同一組件 (PMU 含 TRM)，datasheet 註明需緊鄰電流模組。
- 16DI 與 8DI/4RO 混用時，**16DI 必須放在較低槽號**。
- 4AI/AO 與 8AI 混用時，4AI/AO 放較低槽號 (左側)。
- 多片通訊模組時只有一個 COM2 生效。
- I/O 編號依上電時偵測順序自動分配 (第一片 DI 模組 = DI1–DI16，第二片 = DI17–DI32…)。

## 選配模組一覽
| 模組 | 內容 | 上限 |
|---|---|---|
| DI16 | 16 DI，乾接點 (內部 24VDC) 或濕接點 24/48/125/250VDC，掃描 1ms@60Hz | 3 片 (48 DI) |
| RLY8 | 8 繼電器 SPST Form A，8A@250VAC、5A@30VDC、0.25A@250VDC；動作 10ms、釋放 5ms | 3 片 (24 RO) |
| DI8 RO8 / 8DIOR | 8 DI + 4 RO (EMR 或 SSR) | 3 片 |
| 4AIOx | 4 AI + 4 AO，±1mA(x=1)/0-20mA(x=2)/0-1mA(x=3)/4-20mA(x=4)，0.5%FS，更新 2 週波 | 3 片 (12AI/12AO) |
| 4AIV | 4 AI ±10V | — |
| TRM | 暫態錄波，4 電壓通道對地，至 2kV，512/1024 spc，自動偵測免設定 | 1 |
| PMU | 同步相量 (含 TRM)，IEEE C37.118.1，可選 IEEE 1588 網口 | 1 |
| DFR-CS2/CS2S/CS2SL | 故障錄波 4 電流通道至 200A (40×In)，含 4 只 HACS | 1 |
| IRIG | IRIG-B (BNC) + 第二 RS-422/485 | — |
| TXFX | 光纖乙太網路 100FX(SC) + 10/100TX 備援 + 第二 RS-422/485 | 1 (Slot1) |
| T3G / T3C | 2G/3G GSM / CDMA 數據機 + 第二 RS-422/485 (不支援 61850/60870) | 1 (Slot1) |
| BACDC / B21DC | 輔助電源 85-265VAC & 40-300VDC / 9.5-36VDC | 1 |

## 訂購碼 (Brochure 2018 order string)
`PM180 - [顯示] - [電壓] - [電流] - [頻率] - [準確度/PQ標準] - [通訊] - [測試證書] - [選配模組…]`
- 顯示：X=無顯示 (transducer)、G=5.7" 彩色觸控 RGM180、G-DIN、D=3 行 LED (RDM180)、D-DIN、M=多視窗 LED (RDM312)
- 電壓：(空)=690V 額定、U=120V 額定
- 電流：5=50A 校正 5A (0.2S)、1=10A 校正 1A (0.2S)、CS1S=100A 分裂式 HACS (Class 1)、CS1H、HACS、FLEX (3VAC 鉤表)
- 頻率：50Hz/60Hz
- 標準：A=ANSI C12.20 + IEEE1159、E=IEC 62053-22 + EN50160、G=GOST
- 通訊：(預設) Modbus RTU/TCP、DNP3、DNP/TCP、IEC 60870-5-101/104；**850 = 加 IEC 61850 (SISCO)**
- 證書：CC = ISO 17025 / ILAC 校正證書
- 選配模組最多 3 片 (見上表代碼)

## 顯示器
- RDM180 (PM180-D)：2×4 位 + 1×6 位紅色 LED，6 鍵。
- RDM312 (PM180-M)：12 值 + 1 文字視窗，7 鍵。
- RGM180 (PM180-G)：5.7" TFT 觸控 320×240，可選乙太網路 (PoE)，內含 IR 光學埠 (COM4)。
- 均接 COM3 (DB15) RS-485 Modbus RTU，距離至 500m (需外接 12VDC)，附 3m 線 AC0198。

## 電源
- 主電源：50-290VAC 50/60Hz 或 90-290VDC，10W。端子 L/+、N/-、GND。
- 輔助備援電源模組 (AUX P.S.)：50-290VAC / 40-290VDC，或 9.6-35VDC (12/24V 選項)，20W。可在大幅電壓驟降/中斷時持續記錄。

## 環境/機構
- 尺寸 220×152×210 mm，2.5kg；DIN 軌、盤面、19" 機架安裝。
- 工作溫度：datasheet -25~60°C；安裝手冊附錄 -30~70°C (以 datasheet 較新為準)。濕度 0–95% 不凝結，海拔 ≤2000m。

## EDL180 攜帶型事件與資料記錄器
- 本質為裝在攜帶箱的 PM180 (功能、設定、PAS 操作與 PM180 相同)，內建 UPS 斷電 >4 小時。
- 直接量測至 828VAC；Flex 鉤表 30/300/3000A (需電池) 或 200A 型 (免電池)，2V/3V AC 輸出。
- 前面板：AC 電源座、保險絲、電源開關、RGM 顯示、ETH、電流鉤表輸入、電壓輸入、USB-A、能量脈衝 LED、IR 埠、電池電量 LED (3 顆) + 充電 LED (紅=充電、藍=滿)。
- 安裝距離：≤600A 線路至少 0.5m、600–3000A 至少 1m。
- 接線模式與 CT 比在 PAS 設定 (見 EDL180_Manual §4.7)。
