# 文件索引 (原始 PDF ↔ fulltext ↔ 主題)

原始資料夾 (PM180_SOURCE_DIR，見 SKILL.md) 結構：

```
PM180\
├─ README.md                       資料夾說明 (索引)
├─ 01_產品總覽\
│   ├─ PM180_DATASHEET.pdf          (13 頁, 2024/07)
│   └─ PM180-Brochure-2018.pdf      (16 頁, BB0160, 含訂購碼)
├─ 02_硬體安裝\
│   └─ PM180-Installation-Manual.pdf (42 頁, BG0521)  ← 接線圖 (PDF頁=印刷頁+4)
├─ 03_操作設定\
│   ├─ PM180-Operation-Manual.pdf    (195 頁, BG0525 Rev.A7)
│   └─ PM180-PLC-Configurator-Guide.pdf (51 頁)
├─ 04_通訊協定\
│   ├─ PM180-Modbus.pdf              (234 頁, BG0527)
│   ├─ PM180-DNP.pdf                 (55 頁, BG0524)
│   ├─ PM180-IEC-60870-5.pdf         (89 頁, BG0591)
│   ├─ PM180-IEC-61850.pdf           (33 頁, BG0523)
│   ├─ PM180-IEC-61850-PICS.pdf      (55 頁)
│   └─ PM180-IEEE-C37-118-2.pdf      (16 頁)
├─ 05_應用說明\
│   ├─ PM180-Application-Note-Phasor-Measurement-Unit-PMU-Tech-App-Note.pdf (17 頁)
│   └─ PM180-ApplicationNote-BayControlUnit.pdf (19 頁, BG0531)
├─ 06_IEC61850_ICD\
│   ├─ PM180_IEC61850_ED1-1.zip  → PM180_2003_Rev7.icd  (Edition 1)
│   └─ PM180_IEC61850_ED2.zip    → PM180_2007B_Rev7.icd (Edition 2)
└─ 07_周邊_EDL180\
    └─ EDL180_Manual.pdf             (10 頁, BG0647)
```

| 文件 | fulltext 檔 | 主要內容 | 攻略檔 |
|---|---|---|---|
| Datasheet | PM180_DATASHEET.txt | 功能、規格、模組、標準 | 01 |
| Brochure 2018 | PM180-Brochure-2018.txt | 應用、規格、**訂購碼** | 01 |
| Installation Manual | PM180-Installation-Manual.txt | 安裝、接線、模組、通訊接線、電池、規格 | 02 |
| Operation Manual | PM180-Operation-Manual.txt | 裝置描述、RDM、Telnet、PAS、程式設定、錄波、TOU、協定設定、控制、監視、檔案、COMTRADE、附錄 (類比輸出參數、setpoint 動作、監視參數清單、EN50160 log、Data scales、診斷碼) | 03 |
| PLC Configurator Guide | PM180-PLC-Configurator-Guide.txt | IEC 61131 FBD/LD | 10 |
| Modbus Reference | PM180-Modbus.txt | 協定、格式、完整暫存器表、檔案格式 | 04 |
| DNP3 Reference | PM180-DNP.txt | DNP 實作、點表、設定、Device profile | 05 |
| IEC 60870-5 Reference | PM180-IEC-60870-5.txt | 101/104 實作、IOA 表、互通性表 | 06 |
| IEC 61850 Reference | PM180-IEC-61850.txt | 設定影響、預設 dataset/RCB、PAS 61850 設定 | 07 |
| IEC 61850 PICS | PM180-IEC-61850-PICS.txt | PICS / PIXIT / MICS / TICS | 07 |
| IEEE C37.118.2 | PM180-IEEE-C37-118-2.txt | 同步相量通訊、時間同步、設定 | 08 |
| PMU App Note | PM180-Application-Note-…-PMU-….txt | PMU 安裝、量測、時間同步、通訊、規格 | 08 |
| BCU App Note | PM180-ApplicationNote-BayControlUnit.txt | BCU 運作、61850/Modbus、設定 | 09 |
| EDL180 Manual | EDL180_Manual.txt | 攜帶型記錄器 | 01 |

## Point ID 跨協定換算規則 (重要捷徑)
Modbus 表中每個量測都有 Point ID (例 1 秒 V1 = 0x1100)：
- **IEC 60870-5 IOA = Point ID + 16384** (0x4000)
- **DNP3 Extended 點號 = Point ID + 32768** (0x8000)
- Modbus 32-bit 位址不是線性換算，請查表 (區塊起始見 04-modbus.md)。
