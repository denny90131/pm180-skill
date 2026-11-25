# PM180 Skill（Claude Code）

SATEC expertmeter **PM180** 多功能電力品質分析儀／變電站自動化單元（含 EDL180 攜帶型記錄器）的 Claude Code skill 與技術知識庫。

安裝後，只要在對話中提到 PM180、SATEC、PAS、Modbus 暫存器、DNP3 點表、IEC 60870-5-104 IOA、IEC 61850 ICD 等相關問題，Claude 會自動載入本 skill，依攻略檔與原廠文件全文回答，並標註出處。

> ⚠️ 本 repo 含原廠手冊全文（`fulltext/`）與 ICD 檔（`assets/icd/`），版權屬 SATEC，**請維持 Private，勿公開散布**。

## 涵蓋範圍

- 硬體／訂購碼／模組（DI16、RLY8、4AIO、TRM、DFR、PMU、TXFX、IRIG-B）
- 安裝接線、CT/PT 設定、接線模式（4LN3、3OP2…）
- RDM 面板、Telnet、PAS 設定、錄波、TOU、韌體升級、診斷碼
- 通訊協定：Modbus RTU/TCP、DNP3、IEC 60870-5-101/104、IEC 61850（含 GOOSE/Report）、IEEE C37.118.2
- Bay Controller 開關控制、PLC Configurator
- 故障排除 FAQ、SCADA/HMI 整合步驟

## 目錄結構

```
pm180/
├─ SKILL.md          skill 入口：觸發描述、工作流程、主題路由表、一頁速查
├─ references/       中文攻略重點（01–12 依主題分檔 + doc-index.md 文件索引）
├─ fulltext/         原廠 PDF 以 pdftotext -raw 轉出的全文（供 Grep 查暫存器/點號）
└─ assets/icd/       IEC 61850 ICD 範本（Ed1：PM180_2003_Rev7、Ed2：PM180_2007B_Rev7）
```

| 攻略檔 | 主題 |
|---|---|
| `01-hardware-ordering.md` | 型號、訂購碼、模組、規格、EDL180 |
| `02-installation-wiring.md` | 安裝、接線、COM 埠、IRIG-B、電池 |
| `03-setup-pas-operation.md` | RDM、Telnet、PAS、錄波、TOU、韌體、診斷碼 |
| `04-modbus.md` | Modbus 暫存器、格式、縮放、字序 |
| `05-dnp3.md` | DNP3 點表、Class／事件 |
| `06-iec60870-5.md` | IEC 101/104 IOA、ASDU |
| `07-iec61850.md` | 61850 授權、LN、Dataset、Report、GOOSE |
| `08-pmu-c37118.md` | PMU 同步相量、PTP／IRIG-B |
| `09-bay-controller.md` | 斷路器／隔離開關控制、連鎖 |
| `10-plc-configurator.md` | FBD／LD 邏輯 |
| `11-troubleshooting-faq.md` | 通訊不通、讀值異常、常見坑 |
| `12-integration-playbook.md` | SCADA／HMI 整合建置步驟 |
| `doc-index.md` | 原始 PDF ↔ fulltext ↔ 主題對照 |

## 安裝／還原

```bash
gh repo clone denny90131/pm180-skill ~/.claude/skills/pm180
```

Windows 實際路徑為 `C:\Users\<使用者>\.claude\skills\pm180`。放好後重開 Claude Code 即可生效。

> 若需看接線圖等圖片內容，`SKILL.md` 內的 `PM180_SOURCE_DIR` 指向原始 PDF 資料夾，換電腦時請更新該路徑。

## 更新

```bash
cd ~/.claude/skills/pm180
git add -A
git commit -m "更新說明"
git push
```

## 相關 skill

- `mdc-700-config`：用 ICP DAS MDC-700 把多台 PM180 的 RS-485 Modbus RTU 集中成 Modbus TCP。
