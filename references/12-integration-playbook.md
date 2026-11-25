# 12 SCADA / HMI 整合建置攻略 (通用流程)

適用任何上位系統 (WinCC OA、Ignition、Node-RED、PLC、Python、Gateway…)。需求不明時先問使用者下表「需求確認」項目。

## 0. 需求確認 (建置前要問清楚)
1. 上位系統與驅動 (Modbus TCP / RTU、DNP3、IEC 104、IEC 61850 MMS)。
2. 設備數量、拓撲 (直接 ETH、RS-485 多點、經 Gateway 如 MDC-714)。
3. 需要的點：即時量 (V/I/P/Q/S/PF/Hz)、電能、需量、諧波/電能品質、DI/DO 狀態、告警、控制 (繼電器/斷路器)。
4. 更新週期 (秒級即時 / 15 分鐘電能)、是否需要事件時標 (SOE → DNP3/104/61850 較適合)。
5. 時間同步來源 (GPS IRIG-B / SNTP / PTP)。
6. 是否需要錄波檔 (COMTRADE) 或 PQ 報表 (PAS Upload Scheduler)。
7. 現場 PT/CT 比 (影響單位與縮放)、韌體版本 (是否含 61850)、模組配置。

## 1. 協定選擇建議
| 情境 | 建議 |
|---|---|
| 一般能源管理 / HMI 即時監視 | **Modbus TCP 讀 32-bit 區** (最簡單、點表固定) |
| 多台 RS-485 序列 | Modbus RTU 多點 (每台不同站號，115200 或 19200) 或經 Gateway 轉 TCP |
| 電力調度、需事件時標、不想輪詢 | IEC 60870-5-104 (spontaneous + Float 型別) |
| 北美/水務/既有 DNP 系統 | DNP3 TCP (注意無 unsolicited) |
| 數位變電站、GOOSE 連鎖、Report | IEC 61850 (需授權 + ICD/CID 工程) |
| 廣域量測 | C37.118.2 → PDC |
- 多協定可同時使用 (例：SCADA 用 104，工程師同時用 PAS 走 Modbus TCP)。

## 2. 設備端設定順序 (PAS)
1. 建 site → 連線 (USB 最快) → Upload Setups 備份原設定。
2. **Basic Setup**：接線模式、PT 比/二次、CT 一次、頻率 (台灣 60Hz)、相序 → Send。
3. Device Options：能量小數位、roll 值、Volts/Amps scale (若用 16-bit 或 DNP/104 scaled 才重要)。
4. Local Settings：時區 +480、DST 停用、同步來源。
5. Communications：IP/遮罩/閘道 (Network Setup)、序列埠協定/站號/鮑率。
6. 協定專屬設定 (DNP Class/事件、104 mapped/Class2、61850 IED/Report…)。
7. 啟用密碼保護 (選擇性) — 記錄密碼，SCADA 若要寫入需先授權。
8. Save 到 site database (交接用)。

## 3. Modbus TCP 點表建置要點
- 位址用 0-based (`references/04-modbus.md` §10 的最小點表可直接用)。
- 型別：UINT32/INT32、**word order = low word first (CDAB)**。
- 縮放：依 PT 比決定 (PT>1：V×1、kW×1；PT=1：V×0.1、W×1)；電流 ×0.01；PF ×0.001；Hz ×0.01；THD ×0.1。
- 區塊讀取，每次 ≤120 暫存器：13952(66)、14336(28)、14464(10)、14592(70，需量)、14720(18，電能)。
- 想一次讀完不連續點 → 用 User Assignable Registers 0–119。
- 輪詢週期：即時量 1–2 秒 (資料本身 1 秒更新)；電能 15 秒–1 分鐘。
- 通訊品質點：診斷旗標 44326、DI bitmap 12544、繼電器 bitmap 12800、Static event flags 12928。
- 控制：繼電器強制動作 44046 / 釋放 44050 (UINT64 bitmap)；斷路器合 44458 / 分 44460，結果看 44526+。

## 4. 經 RS-485 Gateway (如 ICP DAS MDC-714)
- PM180 COM1/COM2 設 Modbus RTU、各台不同站號、相同鮑率/8N1。
- Gateway 讀 FC03，32-bit 要讀成對暫存器並保持 CDAB 字序。
- 每個請求 ≤120 暫存器；建議分成 3–4 個區塊請求。
- 產生 MDC config.csv → 使用 `mdc-700-config` skill。

## 5. 驗收檢查
1. 與 PAS RT Data Monitor 或 RDM 顯示值比對 V/I/P/PF/Hz/kWh。
2. 正負號：送電方向 kW 應為正 (import)。
3. 切換 PT 比/改 CT 後重新驗證縮放。
4. 中斷網路測試重連、閒置逾時。
5. 事件類：觸發 DI 確認 SOE/事件時標。
6. 時間同步：確認設備時間與 SCADA 一致 (時區)。
