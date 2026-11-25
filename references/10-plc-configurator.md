# 10 PLC Configurator (IEC 61131-3 FBD / LD) 攻略

來源：PM180-PLC-Configurator-Guide。原文：`fulltext/PM180-PLC-Configurator-Guide.txt`。

## 定位
- SATEC 的 IEC 61131-3 子集編輯器，**把 FBD/LD 邏輯編譯成 PM180 的 Control Setpoints**。其他設定仍用 PAS。
- 限制：只有 1 個 task，每 20ms 執行 (硬體限制)；類比比較用非標準 `*_THRESHOLD` 函數模擬 setpoint 門檻。
- 可與 PAS 共用 setpoint：裝置屬性 "Settings range for the logic generator" 指定 PLC Configurator 可用的 setpoint 範圍 (例 `1,2,10-16,20-30`，預設 1–64)，其餘留給 PAS。

## 安裝
- 需 **Java 8 64-bit**、Windows 7 以上 64-bit。
- 解壓縮 → 執行 `bin/scadastudio.exe`。

## 資料型別
BOOL、LREAL、TIME (ms)、DATE (= PM180 Packed date)、TOD (= Packed time)、DT (= Timestamp)、DP (雙點：IS 中間 / OFF / ON / BS 壞)。

## 內建函數
AND、OR、NOT、GE、EQ、GE_THRESHOLD、LE_THRESHOLD、TON (延遲 ON)、TOF (延遲 OFF)。(見原文 §2.3)

## 操作流程
1. File → New Project → SATEC → SATEC PLC Project → 名稱/路徑 → Finish。
2. 專案右鍵 Add device → 設 Device properties (通訊介面、位址、setpoint 範圍)。
3. 必須有一個名為 **MAIN** 的 function block (主程式)。
4. 新增 FB → 加變數 (設定位址＝PM180 參數，例如 DI、繼電器、量測值、XCBRn_Pos…) → 加 work area。
5. FBD：拖入內建元件、連線、時間延遲 (TON/TOF)、setpoint 變數的門檻上下限。LD：rails、contact、negated contact、coil、negated coil → 可轉成 setpoints。
6. 通訊：USB (Modbus RTU)、序列 COM1 (RS-232/485)、Ethernet (CPU 10/100TX，直通線)。
7. **Send configuration to the device** (上傳前會讀取設備 CT/PT 比計算常數)。**類比門檻一律用 SI 單位 (V、A、W…)，否則邏輯會錯**。
8. 也可從設備下載既有設定回專案。

## 錯誤排除
| 訊息 | 處理 |
|---|---|
| Address is not set / Unknown address | 圖上變數未設位址 → Variable properties 設定 (§3.7.3) |
| Connection error | 按 Return，檢查介面與連線參數 (可先用 PAS 連線確認正確設定) |
| No MAIN module | 把一個 FB 改名為 MAIN 或新增名為 MAIN 的 FB |
| Unconnected inputs | FBD 有未連接的腳位 (紅圈) |
