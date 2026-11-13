# Hybrid TPA-Agent - System Design Document (系統設計文件)

本文件詳細記錄 **Hybrid TPA-Agent (混合型技術與價格行為交易代理人系統)** 的架構設計、模組職責、資料流、核心演算法與工程決策。本專案採用 **OpenSpec** 規範驅動開發 (Spec-Driven Development) 進行設計與實作追蹤。

---

## 1. System Overview (系統概述)

本系統旨在結合**「100% 確定性演算法（空間結構辨識）」**、**「機器學習（勝率與機率預測）」**與**「大型語言模型（情境脈絡推理與動態風控）」**，為非專業交易員提供兼具數學嚴謹性、統計勝率支撐與高度可解釋性的短線高勝率交易設定（Actionable Trading Setups）。

---

## 2. System Architecture (四層式分塊架構)

系統採用嚴格解耦的四層式架構（Tiered Block Architecture），共包含八大核心子模組：

```
[Layer 1: Input & Data Layer (輸入與資料層)]
  ├── [1.1] User Preference Block (交易標的、風險承受度、資金比例)
  └── [1.2] Data Ingestion Engine (即時/歷史 OHLCV 擷取、Volume Profile 計算)
         │
         ▼
[Layer 2: Core Analytical Engines (核心運算引擎層)]
  ├── [2.1] Deterministic Price Action & S&D Zone Engine (Swing 點位、供需區標記)
  └── [2.2] Machine Learning Win-Rate Engine (XGBoost 突破/反彈勝率預測)
         │
         ▼
[Layer 3: LLM Reasoning & Domain Knowledge Layer (推理與知識層)]
  ├── [3.1] Domain Knowledge Base (價格行為學規則庫、S/R 轉換機制、流動性陷阱)
  └── [3.2] Planner & Reasoning Agent (Chain-of-Thought 多步驟情境語意推理)
         │
         ▼
[Layer 4: Risk Management & Output Layer (風控與輸出層)]
  ├── [4.1] Decision & Risk Management Engine (動態停損/停利計算、RR >= 2.0 門檻)
  └── [4.2] Structured Report Interface (結構化 JSON 與 Markdown 交易決策報告)
```

### 八大子模組詳細職責
1. **[1.1] User Preference Block**：接收使用者設定之交易標的（如 `BTCUSDT`）、時間級別（如 `15m`、`1h`）、每筆最大風險額度（如 1%~2% 本金）。
2. **[1.2] Data Ingestion Engine**：高頻抓取 OHLCV 資料，清洗缺失值並計算成交量分佈（Volume Profile），標定高成交量節點（HVN）與控制點（POC）。
3. **[2.1] Deterministic Price Action & S&D Zone Engine**：透過滾動極值演算法自動標記波高點（Swing High）與波低點（Swing Low），偵測買賣失衡起漲/起跌點，產出確定性供需區（Supply & Demand Zones）與 S/R 轉換線。
4. **[2.2] Machine Learning Win-Rate Engine**：萃取技術指標（RSI, ATR, Volume Z-score）與價格至各供需區邊界的空間幾何距離特徵，輸入 XGBoost 分類器計算反彈與突破勝率。
5. **[3.1] Domain Knowledge Base**：封裝經典價格行為學公理，包括 S/R Flip（支撐壓力互換）、流動性清掃（Liquidity Sweep）、ChoCh（結構轉變）與吞噬/Pinbar 形態。
6. **[3.2] Planner & Reasoning Agent**：整合空間座標與 ML 勝率分數，利用 CoT 提示工程比對高低時間級別結構，評估當前盤勢是否具備假突破或流動性陷阱風險。
7. **[4.1] Decision & Risk Management Engine**：以供需區外側加 ATR 緩衝作為精確停損點，對向供需區邊界作為停利點；嚴格把關數學期望值 $E > 0$、勝率 $\ge 65\%$ 與風報比 $RR \ge 2.0$。
8. **[4.2] Structured Report Interface**：輸出機器可讀之標準 JSON 契約與操盤手友善之可執行報告，支援 Human-in-the-Loop 一鍵確認。

---

## 3. Data Flow & Core Algorithm (資料流與核心演算法)

### 3.1 系統運作資料流 (Execution Flow)
1. **Trigger**：系統定時抓取標的最新完成之 K 線 Bar 資料。
2. **Deterministic Processing**：SNR 引擎平行計算空間結構，標記出活躍供需區。
3. **ML Prediction**：特徵工程萃取特徵向量，XGBoost 模型推論產出條件反彈/突破勝率 $P_{\text{win}}$。
4. **CoT Reasoning**：LLM 代理人帶入價格行為規則庫進行語意檢驗，過濾假突破雜訊。
5. **Risk Gate**：風控模組動態計算 Stop-Loss、Take-Profit 與 Risk-to-Reward Ratio ($RR$)，驗證門檻。
6. **Report Dispatch**：產出 `APPROVED` 交易策略或 `WAIT/NO_TRADE` 觀望報告。

### 3.2 核心演算法虛擬碼 (Algorithm 1)
```python
Algorithm 1: Hybrid Technical & Price Action Agent (Hybrid TPA-Agent)
Input : Asset Symbol S, Timeframe TF, Historical OHLCV Data D
Output: Actionable Trading Setup & Decision Report R

01: Initialize Agent Context, SNR Algorithm, and ML Prediction Model
02: Receive user target S and timeframe TF
03: raw_data <- DataAcquisition.fetch_ohlcv(S, TF)
04: 
05: // 步驟一：確定性演算法計算支撐壓力與供需區
06: demand_zones, supply_zones <- SNR_Engine.detect_zones(raw_data)
07: current_price <- raw_data.close[-1]
08: 
09: // 步驟二：機器學習引擎計算突破/反彈勝率
10: features <- FeatureExtractor.extract(raw_data, demand_zones, supply_zones)
11: win_probability <- ML_Engine.predict_win_rate(features)
12: 
13: // 步驟三：AI Agent 進行領域推理與風控計算
14: agent_context <- {current_price, demand_zones, supply_zones, win_probability}
15: evaluated_plan <- PlannerAgent.reason(agent_context, PriceActionKnowledgeBase)
16: 
17: IF evaluated_plan.win_probability >= 0.65 AND evaluated_plan.risk_reward_ratio >= 2.0 THEN
18:     entry_price <- current_price
19:     stop_loss   <- evaluated_plan.recommended_stop_loss
20:     take_profit <- evaluated_plan.recommended_take_profit
21:     R <- SynthesisEngine.generate_report(
22:         Status="APPROVED", 
23:         entry=entry_price, 
24:         stop_loss=stop_loss, 
25:         take_profit=take_profit, 
26:         evidence=evaluated_plan.evidence
27:     )
28: ELSE
29:     R <- SynthesisEngine.generate_report(
30:         Status="WAIT/NO_TRADE", 
31:         reason="Risk-to-Reward (< 2.0) or Win Rate (< 0.65) below threshold."
32:     )
33: END IF
34: 
35: return R
```

---

## 4. API & Data Schemas (資料格式與介面規範)

### 4.1 供需區結構化輸出 (Zone Detection JSON)
```json
{
  "symbol": "BTCUSDT",
  "timeframe": "1h",
  "current_price": 62450.0,
  "demand_zones": [
    {
      "zone_id": "DZ-01",
      "price_low": 60800.0,
      "price_high": 61250.0,
      "volume_poc": 61050.0,
      "retest_count": 1,
      "strength": "strong"
    }
  ],
  "supply_zones": [
    {
      "zone_id": "SZ-01",
      "price_low": 64200.0,
      "price_high": 64650.0,
      "volume_poc": 64400.0,
      "retest_count": 0,
      "strength": "moderate"
    }
  ]
}
```

### 4.2 結構化決策報告輸出 (Final Actionable Report JSON)
```json
{
  "status": "APPROVED",
  "timestamp": "2026-11-13T21:00:00Z",
  "symbol": "BTCUSDT",
  "action": "LONG",
  "entry_price": 61250.0,
  "stop_loss": 60550.0,
  "take_profit": 64200.0,
  "risk_reward_ratio": 4.21,
  "ml_win_probability": 0.72,
  "reasoning_evidence": [
    "價格回測 1h 強烈需求區 DZ-01，收盤形成帶長下影線之看漲 Pinbar",
    "低時間級別 (15m) 出現結構改變 (ChoCh) 與量能吸收",
    "對向第一道阻力區為 SZ-01 (64200.0)，具備 4.21 倍非對稱風報比"
  ],
  "risk_control": {
    "max_capital_risk_pct": 1.5,
    "position_sizing_units": 0.214
  }
}
```

---

## 5. Engineering Decisions & Trade-off Matrix (工程決策與權衡矩陣)

| 決策面向 | 採用架構 / 決策 | 替代方案 | 採納理由 |
| :--- | :--- | :--- | :--- |
| **數值與空間計算** | 100% 確定性 Python 演算法 | 多模態 LLM 直接視覺判讀圖表 | 徹底杜絕 LLM 的數值與幾何幻覺，保障邊界計算精確。 |
| **勝率分類預測** | XGBoost 漸進增強決策樹 | LSTM / Transformer 時序深度學習 | 針對結構化時序與幾何特徵具備極高收斂速度與抗過擬合能力，推論延遲低於 10ms。 |
| **執行機制** | Human-in-the-Loop (HITL) 輔助 | 全自動 API 高頻無人跟單 | 避免突發黑天鵝事件與 API 斷流導致重大穿倉，由操盤手保有最終確認權。 |
| **風控機制** | 供需區結構 + ATR 動態緩衝 | 固定百分比或固定點數停損 | 動態適應盤面波動率，避免在低波動期停損過寬、高波動期頻繁被洗出場。 |

---

## 6. OpenSpec Integration (OpenSpec 整合)

本架構所有模組規格均已轉換為 OpenSpec 規範，位於 `openspec/changes/hybrid-tpa-agent/`：
- `proposal.md`：專案動機、變更範圍與 6 大核心 Capabilities 定義。
- `specs/`：6 大子功能規範（`market-data`, `zone-detection`, `win-rate-prediction`, `reasoning-agent`, `risk-management`, `report-generation`），全部通過 strict 規範驗證。
- `design.md`：包含完整上下文、架構圖、決策矩陣與降級策略。
- `tasks.md`：涵蓋 6 個開發階段、17 個可驗證任務之實作檢查清單。
