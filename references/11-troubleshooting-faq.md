# 11 故障排除與常見坑 (FAQ)

彙整自所有原廠文件。每條都附出處，需要細節再 Grep fulltext。

## 開機 / 硬體
| 症狀 | 原因 / 處理 | 出處 |
|---|---|---|
| 第一次上電很久沒反應 | 裝電池後首次初始化約 7 分鐘 (平常 40 秒)，等待即可 | Install §2.2、Ch4 |
| CPU LED 2 閃 | Service mode (韌體升級中)，所有功能停止 | OM Ch2 |
| CPU LED 3 閃 / RDM "Critic Error" | 嚴重錯誤 Diagnostics mode，查診斷碼 (OM Appendix F)，清除診斷；反覆發生聯絡代理商 | OM Ch2/Ch10 |
| MEM.BAT.LOW 紅燈、Event log "BATTERY LOW" | 換 CR1225 (可帶電換) | Install Ch4 |
| 插了模組沒反應 | 檢查槽位規則：TXFX/Cellular 必須 Slot1；16DI 要在 8DI/4RO 之前；4AI/AO 在 8AI 之前；各類 1 片限制；韌體版本是否支援 | Install §2.2/2.3 |
| DI 編號跟預期不同 | I/O 編號依上電偵測順序；換槽但順序不變則編號不變 | OM Ch6 |

## 量測值異常
| 症狀 | 原因 / 處理 |
|---|---|
| 功率、PF 全是 0 | CT 一次 × PT 比 > 10,000,000 → 功率歸零 (OM RDM Basic Setup Note 2) |
| 電壓讀數差 10 倍 | PT=1 時電壓單位 0.1V，PT>1 時 1V；改了 PT 比 SCADA 縮放要同步改 |
| 功率差 1000 倍 | PT=1 時功率單位 W，PT>1 時 kW |
| 功率為負 / PF 怪 | CT 極性或相別錯；檢查相序 (RDM Status `Ph.rOt`、Modbus 11906 相序 0=錯 1=ABC 2=CBA) |
| 電壓是線電壓不是相電壓 | 接線模式非 4LN3/3LN3/3BLN3 時為線電壓 |
| PMU 量不到頻率 | 現場 CBA 相序未在 Basic Setup 設定 |
| 故障時諧波不更新 | 設計如此 (避免 CT 飽和誤讀) |
| 16-bit 暫存器數值很粗 | 0–9999 縮放精度有限 → 改讀 32-bit 區 |
| 能量讀數跳回 0 | 能量 roll 值 (預設 10^9)；16-bit modulo-10000 暫存器需限 8 位數 |
| 能量小數 | U5 單位由 Energy Decimal Places 決定 (預設 0 → 1 kWh) |

## 通訊
| 症狀 | 原因 / 處理 |
|---|---|
| Modbus 32-bit 值很怪 (很大或亂跳) | **字序**：PM180 低字在前 → SCADA 開 word swap (CDAB)；32-bit 起始位址需偶數 |
| 讀取回例外 02 | 位址不存在或一次超過 120 個暫存器 |
| 寫入回例外 01 | 密碼保護啟用 → 先寫密碼到 44378–44379；或該設定不可寫 |
| Modicon 位址差 1 | 手冊位址為 0-based；4xxxx 表示法要 +40001 (>9999 用 +400001) |
| COM2 用 8E1 不通 | COM2 只支援 8N1 |
| N3 版 COM1 RS-232 不通 | N3 版 COM1 只有 RS-485 |
| 接 COM3 後 RS-485 埠損壞 | COM3 DB15 pin 1、8 是 12VDC 輸出，不可接 RS-485 線 |
| RDM 顯示 Con.Err. | COM3 與 RDM 參數 (位址/鮑率/格式) 不一致、線路、Service mode |
| PAS 常出現 "Communication failed" | 增加 Break Time-out (預設 10ms) / Response Time-out；PC 負載高 |
| Modbus/TCP 連線過一陣子斷 | Socket 閒置 5 分鐘自動關 → 定期輪詢或重連 |
| 超過連線數 | ETH 共 5 sockets (Modbus/DNP/104 合計)；61850 最多 4 個 MMS client |
| IEC 104 連線 2 分鐘斷 | 閒置關閉 → 主站送 TESTFR 或設 cyclic |
| IEC 104 收不到 spontaneous/cyclic | 沒設 Client IP；或事件點未 mapped (只有 1–4095 mapped 點有事件)；或另一主站已在收 |
| IEC 104 傳送停止 | 未確認 ASDU 超過 k (預設 12) → 主站要回 S-frame，或設 0 |
| DNP3 沒有 unsolicited | 不支援，必須輪詢 Class 1/2/3 |
| DNP3 事件突然全清 | 改了事件點數量會清空緩衝並重設預設事件點 |
| DNP 16-bit 值怪 | 16-bit AI scaling 預設啟用 → 反算或改讀 32-bit 變數 |
| 61850 client 無法遙控 (AddCause=2) | REMOTE CONTROL 旗標未經 setpoint 設 ON，或控制埠被封鎖 |
| 61850 停止運作 (約 30/60 天後) | 臨時授權到期 → 買永久授權 (提供序號) |
| 下載 61850 設定後 IP 沒變 | 設計如此，IP 在 Communication Setup → Network Setup 改 |
| GOOSE 訂閱值為 0 | 未收到/逾時/dataset 或 confRev 不符 → 查 sbsLGOS 或 Modbus 44338/44340 |
| C37.118 無法開新串流 | 5 個串流槽被佔 (UDP client 異常離線) → Stop UDP Streams = YES；Telnet `enet`→`proto` 查 |
| 韌體升級失敗 | 只能用 Modbus 協定；序列埠用 115200；需輸入密碼；Cellular 升級後 IP 可能改變 |

## 時間
| 症狀 | 原因 / 處理 |
|---|---|
| 診斷碼 24/25 一直出現 | 時間同步來源設 IRIG-B 但沒接或 GPS 失鎖；此告警無法遮蔽 → 改成未用 DI 或 SNTP |
| 時間差 8 小時 / DST 亂跳 | 預設時區 -300 分 + 美國 DST → 台灣設 +480、DST 停用 |
| SNTP 不同步 | Local Settings 的同步來源要選 SNTP，且 SNTP Client Enabled = YES、伺服器 IP 改為內網 NTP |
| IRIG-B 斷訊後 | 5 分鐘後改用內部 RTC，恢復後自動重新取 GPS 時間；事件記錄在 Event log |
| IEC 104 時標 IV=1 | 失電後未經 C_CS_NA_1 校時 |

## 錄波
| 症狀 | 原因 / 處理 |
|---|---|
| 沒有波形/資料記錄 | 記憶體未分割給該檔；Setpoint 未設觸發動作；PQ/Fault recorder 未啟用 (Modbus 44136/44137) |
| 故障錄波沒觸發 | 各故障觸發需個別啟用；DI 外部觸發要勾 Fault Recorder；同時多個 DI 只有第一個觸發 |
| DI 事件重複 | 消彈跳時間太短 (1–100ms，8 個一組共用) |
| 需要 COMTRADE | PAS Export 或 Upload Scheduler 自動轉檔；61850 檔案服務 `\COMTRADE\` |

## PMU / C37.118.2
| 症狀 | 原因 / 處理 |
|---|---|
| Reporting Rate 改了，PMU tester 收到的速率不變 (例如設 30 收到 10、設 60 收到 30) | tester (commanded client) 收到的是**串流 Data Rate**，不是 Reporting Rate。舊值若剛好是新 Reporting Rate 的約數就不會被自動修正 → PAS C37.118.2 Setup 的 Client-server **Data Rate 1/2** (Modbus 陣列 17 的 +43/+53，spont 為 +49/+51) 一併改。2026-11 現場實例：Reporting 60、四條串流都是 30 |
| 快速診斷 | Modbus TCP 寫 47480=[17,0] 再讀 47480 起 58 個暫存器，可一次看 +13 reporting rate 與各串流 rate/ID；SV Ed2 sample rate 用 [22,232] |
