---
name: pm180
description: SATEC expertmeter PM180 多功能電力品質分析儀 / 變電站自動化單元 (含 EDL180 攜帶型記錄器) 的完整攻略與技術知識庫。凡是使用者提到 PM180、SATEC、expertmeter、EDL180、RDM180/RDM312/RGM180 顯示器、PAS 軟體、PLC Configurator，或詢問此設備的接線/CT PT 設定/接線模式 (4LN3、3OP2…)、模組 (DI16、RLY8、4AIO、TRM、DFR、PMU、TXFX、IRIG-B)、通訊 (Modbus RTU/TCP 暫存器位址、DNP3 點表、IEC 60870-5-101/104 IOA、IEC 61850 ICD/CID/GOOSE/Report、IEEE C37.118.2 同步相量)、電力品質/故障錄波/SOE/TOU 設定、Bay Controller 開關控制、韌體升級、診斷碼與故障排除時，務必使用本 skill，即使使用者沒有明說「skill」。
---

# PM180 詳細攻略 (SATEC expertmeter PM180)

本 skill 已內建所有原廠文件的**全文文字檔**與 ICD 範本，可獨立運作，不依賴原始 PDF 資料夾。

## 原始資料夾位置 (可變動)

```
PM180_SOURCE_DIR = C:\Users\denny.chuang\OneDrive - Intelligent Cloud Plus Corp\桌面\PM180
```

- 只有在需要**看圖** (接線圖、端子圖、PAS 畫面截圖) 時才需要開原始 PDF；文字/表格內容一律先查 `fulltext/`。
- 若上述路徑不存在 (資料夾被搬移)：先用 Glob/搜尋找 `PM180-Installation-Manual.pdf` (常見位置：使用者 OneDrive、桌面、文件)，找到後**更新本檔上面這一行**，找不到再問使用者新路徑。
- 原始資料夾結構見 `references/doc-index.md`。

## 使用方式 (給 Claude 的工作流程)

1. 先依下表讀對應的 `references/*.md` 攻略檔 (已整理成中文重點 + 關鍵數值)。
2. 攻略檔不夠細時，用 **Grep 搜尋 `fulltext/` 原文** (英文關鍵字，例如 `Grep pattern="1-Second Total Values" path=<skill>/fulltext/PM180-Modbus.txt -A 20`)。fulltext 為 `pdftotext -raw` 轉出，表格每列一行，可靠度高。
3. 回答時標註出處 (文件名 + 章節)，暫存器位址/點號一律以 fulltext 原文為準，不要憑記憶。
4. 圖片類內容 (接線圖) 告訴使用者對應 PDF 檔名與頁碼/圖號，必要時用 Read 開 PDF 該頁。
5. 使用者問「建置攻略/規劃」類問題但需求不明時 (例如上位系統、協定選擇、點位數量、是否需 61850 授權)，先詢問需求；技術規格問題直接查文件回答。

## 主題路由表

| 使用者問題類型 | 讀這個檔 | 原文 (fulltext/) |
|---|---|---|
| 型號、韌體版本代碼、模組、訂購碼、規格、EDL180 | `references/01-hardware-ordering.md` | PM180_DATASHEET, PM180-Brochure-2018, EDL180_Manual |
| 安裝、電源、CT/PT 接線、接線模式、DI/RO/AI 接線、COM 埠、IRIG-B、電池 | `references/02-installation-wiring.md` | PM180-Installation-Manual |
| RDM 面板選單、Telnet、PAS 操作、基本設定、時間同步、DI/繼電器/Setpoint、錄波器、TOU、韌體升級、診斷碼 | `references/03-setup-pas-operation.md` | PM180-Operation-Manual |
| Modbus 暫存器、資料格式、縮放、32-bit 字序、控制/設定暫存器、檔案傳輸 | `references/04-modbus.md` | PM180-Modbus |
| DNP3 點表、Class 0/事件、縮放 | `references/05-dnp3.md` | PM180-DNP |
| IEC 60870-5-101/104 IOA、ASDU、Interrogation、計量傳送 | `references/06-iec60870-5.md` | PM180-IEC-60870-5 |
| IEC 61850 授權、LD/LN、Dataset、Report、GOOSE、SV、ICD/CID 匯出 | `references/07-iec61850.md` | PM180-IEC-61850, PM180-IEC-61850-PICS, assets/icd/*.icd |
| PMU 同步相量、IEEE C37.118.2、PTP/IRIG-B | `references/08-pmu-c37118.md` | PM180-Application-Note-PMU, PM180-IEEE-C37-118-2 |
| Bay Controller 斷路器/隔離開關控制、連鎖 | `references/09-bay-controller.md` | PM180-ApplicationNote-BayControlUnit |
| PLC Configurator (FBD/LD 邏輯) | `references/10-plc-configurator.md` | PM180-PLC-Configurator-Guide |
| 通訊不通、讀值怪、錯誤碼、常見坑 | `references/11-troubleshooting-faq.md` | (綜合) |
| SCADA/HMI 整合建置步驟 (通用) | `references/12-integration-playbook.md` | (綜合) |
| 文件清單、BG 文件編號、原始 PDF 對照 | `references/doc-index.md` | — |

## 一頁速查 (最常用的事實)

- **出廠通訊**：所有序列埠 COM1–COM3、IR(COM4) = Modbus RTU、19200 bps、8N1、位址 1。COM2 只支援 8N1。N3 版 CPU 的 COM1 只有 RS-485。
- **TCP 埠**：Modbus/TCP 502、DNP3/TCP 20000、IEC 60870-5-104 2404、Telnet 23、C37.118.2 TCP 4712 / UDP 4713；最多 5 條同時連線。乙太網路不檢查站號 (任意 Unit ID 皆回應)。
- **ICD 範本預設 IP**：192.168.0.203 / 255.255.255.0 / GW 192.168.0.1 (實機 IP 以 Telnet `net`/`ip` 或 PAS Network Setup 確認)。
- **密碼**：預設 0，保護預設關閉；Telnet 一定要輸入密碼 (預設 0)。Modbus 寫入密碼到 44378-44379。
- **Modbus**：功能碼 03/04/06/16/22/08；一次最多 120 個暫存器；32-bit 值 **低字在前 (low word first, 偶數位址起)** → SCADA 端通常要設「Word swap / CDAB」。
- **最常用 32-bit 暫存器 (1 秒值，UINT32/INT32)**：V1/V2/V3 = 13952/13954/13956；I1/I2/I3 = 13958/13960/13962；Total kW/kvar/kVA/PF = 14336/14338/14340/14342；Frequency (0.01Hz) = 14468；kWh import/export = 14720/14722；kvarh import/export = 14728/14730；kVAh = 14736。單位依 PT 比：PT=1 → 電壓 0.1V、功率 1W；PT>1 → 電壓 1V、功率 1kW；電流恆為 0.01A；PF 0.001。
- **接線模式代碼**：0=3OP2、1=4LN3(預設)、2=3DIR2、3=4LL3、4=3OP3、5=3LN3、6=3LL3、8=3BLN3、9=3BLL3。
- **CT 一次側 × PT 比 ≤ 10,000,000**，超過則功率與功因歸零。
- **首次裝電池上電初始化約 7 分鐘**，之後每次開機約 40 秒。電池 CR1225 3V。
- **IRIG-B 未接卻選 IRIG-B 同步** → 診斷碼 24/25 持續告警，改選未使用的 DI 或 SNTP。
- **IEC 61850 需授權**：出廠為 30 天臨時授權 (可延長一個月)，永久授權需提供序號向代理商購買。Ed1 ICD = PM180_2003_Rev7.icd、Ed2 = PM180_2007B_Rev7.icd。
- **61850 遙控**：REMOTE CONTROL 旗標預設 FALSE，必須用 setpoint 邏輯設為 ON，否則 CSWI/boGGIO 控制會被拒 (AddCause=2)。
- **韌體升級**：只能走 Modbus (RTU/ASCII/TCP)，PAS → Monitor → Flash Downloader；USB 約 20 秒、115200 序列埠約 9–10 分鐘。
- **PAS 一定先存 Basic Setup** 到 site database，其他設定以它為參考。

## 相關 skill

- 若要用 ICP DAS MDC-700 (MDC-714 等) 把多台 PM180 的 RS-485 Modbus RTU 集中成 Modbus TCP，搭配 `mdc-700-config` skill 產生 config.csv；PM180 端的點表用 `references/04-modbus.md`。
