# Sanskar's Idea: KiranaGuard - The iQOO 15 That Stops a Kirana From Losing Money

**Working title:** KiranaGuard  
**One line:** **A kirana doesn't need another billing app. It needs a phone that catches money disappearing.**

A kirana owner can lose money when a vendor delivers fewer items than billed, stock expires unnoticed, or the owner simply cannot see what is actually left. Our iQOO 15 turns the shopkeeper's phone into an **offline AI loss-prevention system**: photograph a delivery, verify physical goods against the vendor bill, track expiry, block expired sales, and ask the shop questions in Hindi - without typing a word.

---

## 1. Problem Statement Alignment

| | Track | Why it fits |
| :--- | :--- | :--- |
| **Primary** | **Productivity** | Automates repetitive inventory work, verifies deliveries, manages expiry information, and answers stock questions without manual notebook entry. |
| **Secondary** | **Open Innovation** | Uses open-source/on-device AI models and keeps the core workflow functional without cloud APIs. |

The strongest positioning is **Productivity**. The Open Innovation angle supports the architecture but should not distract from the concrete business problem.

---

## 2. The Problem: Money Disappears Without a Clear Record

A typical kirana may receive dozens of deliveries and sell hundreds of packets every day. Under time pressure, the owner may trust the vendor's bill, forget expiry dates, and rely on a handwritten notebook.

That creates five practical leakage points:

| # | Money / trust leak | What happens today |
| :--- | :--- | :--- |
| 1 | **Short delivery** | Vendor bill says 24 units; only 20 physically arrive. |
| 2 | **Expired stock** | Old packets remain at the back of the shelf and are discovered too late. |
| 3 | **Forgotten returns** | Near-expiry stock that could have been returned is missed. |
| 4 | **Expired sale** | An expired packet can accidentally reach a customer. |
| 5 | **Unknown stock** | The owner has to walk to the shelf or flip notebook pages to answer simple questions. |

### The key insight
Existing billing/inventory software primarily records what the owner enters. **We verify what physically arrived.** That is the core differentiation.

---

## 3. The Idea

**The iQOO 15 becomes a loss-prevention system for the shop.**

The core product has only **three jobs**:
1. **Verify every important delivery:** Photo of goods + photo of bill → physical count vs billed count → discrepancy.
2. **Protect against expiry:** Read expiry → alert before expiry → block expired sale.
3. **Let the owner ask in Hindi:** Hindi question → intent → local database → factual answer.

---

## 4. The Core Workflow

### 4.1 Delivery: Verify What Physically Arrived
1. Vendor X arrives. Ramesh selects **Vendor X** in the app.
2. He spreads the goods on the counter and takes a delivery photo.
3. The on-device vision model identifies and counts supported products.
4. He photographs the vendor bill. OCR extracts billed quantities.
5. The app compares the physical count with the bill.

> **Bill: Maggi × 24** | **Received: Maggi × 20** | **Short: 4** | **Estimated value: ₹56**

### Important design principle: Never silently guess
If the camera cannot confidently count an item: *"I am not confident about this item. Please confirm the quantity."* The goal is to make uncertainty visible before it becomes a financial mistake.

### 4.2 Expiry Protection
During delivery, the app reads printed expiry dates, manufacturing dates, or "best before X months" information.
* **Alerts:** 30 days before (return to vendor), 7 days before (sell first), Low stock.
* **Sale Blocking:** If a selected batch is expired, the phone gives a distinct vibration and blocks the sale: **EXPIRED - SALE BLOCKED**.

### 4.3 Hindi Voice: Ask the Shop
The owner shouldn't need an English interface.
* **"Is mahine kya expire hoga?"** → *"12 Parle-G expire honge."*
* **"Vendor X ne kitna kam diya?"** → *"4 Maggi, ₹56."*

The LLM never invents numbers; it only phrases verified database facts into natural Hindi.

---

## 5. Workflow Diagram

`mermaid
flowchart TD
    subgraph IN["① DELIVERY"]
        V["Tap vendor icon"] --> P1["One photo of goods"]
        P1 --> CNT["Ultrawide: identify + count every item<br><i>Hexagon NPU</i>"]
        P1 --> EXP["3x periscope: read printed date<br>→ calculate expiry<br><i>Hexagon NPU</i>"]
        V --> BILLP["Photo of vendor's bill"]
        BILLP --> CHK{"Bill = counted?"}
        CNT --> CHK
        CHK -->|No| SHORT["⚠️ Short delivery flagged<br>₹ owed by vendor"]
        CHK -->|Yes| OK1["Matched"]
        SHORT --> SAVE
        OK1 --> SAVE
        EXP --> SAVE[("On-device DB<br>stock · batches · expiry · vendors")]
    end

    subgraph OUT["② SALE"]
        P2["Photo / voice / + of customer's items"] --> ID["Identify items + batch<br><i>NPU</i>"]
        ID --> EXPCHK{"Expired?"}
        EXPCHK -->|Yes| BLOCK["📳 Vibrate + red + blocked"]
        EXPCHK -->|No| CART["Cart → customer screen<br><i>Office Kit</i>"]
        CART --> DONE["Sale saved, stock reduced"]
    end

    subgraph ALERT["③ ALERTS"]
        A1["30 days: return to vendor"] --- A2["7 days: sell first"]
        A2 --- A3["Low stock"]
        A3 --- A4["Vendor still owes"]
    end

    subgraph ASK["④ ASK"]
        Q["Hindi voice question<br><i>Mic</i>"] --> STT["Speech-to-text<br><i>on-device</i>"]
        STT --> LAYA["Laya: what is being asked?<br>+ confidence"]
        LAYA -->|confident| LOOK["Look up local DB"]
        LAYA -->|unsure| RE["Ask owner again"]
        LOOK --> LLM["Local LLM phrases answer in Hindi"]
        LLM --> SPEAK["Spoken + on screen"]
    end

    SAVE --> ID
    SAVE --> ALERT
    SAVE --> LOOK
    DONE --> SAVE
`

---

## 6. What We Are NOT Trying to Build
To keep the prototype reliable, we are NOT building: full POS replacement, WhatsApp ordering, complex analytics, complete GST accounting, or cloud dashboards. The finale prototype proves only this: **A phone can physically verify inventory, prevent expiry-related loss, and answer the shopkeeper - offline.**

---

## 7. On-Device AI Stack & System Architecture

| Job | Model / technology | Role |
| :--- | :--- | :--- |
| Product detection/counting | YOLO11n (INT8) | Hexagon NPU: Detect supported packets |
| Product recognition | MobileCLIP / DINOv2-small | Hexagon NPU: Identify known products |
| Expiry/bill OCR | PaddleOCR-mobile / ML Kit | Read dates and quantities offline |
| Speech-to-text | Whisper (small/base, quantised) | Hindi/Hinglish voice input |
| Intent/confidence | **Laya multilingual (322M)** | Determine what the owner is asking |
| Local answer generation | Llama 3.2 1B / Gemma (INT4) | Turn verified database facts into Hindi |
| Shop database | SQLite | Local source of truth |

---

## 8. Why It Needs the iQOO 15

| iQOO 15 hardware | Real job in the app |
| :--- | :--- |
| **50 MP ultrawide camera** | Captures an entire delivery in one photo for counting. |
| **50 MP 3x periscope camera** | Reads tiny dot-matrix expiry dates sharply from a normal distance, and sweeps shelves. |
| **Snapdragon 8 Elite Gen 5 + Hexagon NPU** | Runs detection, recognition, date reading, speech, Laya and the LLM, all offline. |
| **Vapor chamber + 7000 mAh battery** | Camera + AI all day at the counter without overheating; works through power cuts. |
| **3D ultrasonic fingerprint** | Only the owner can see purchase prices, vendor dues and profit. |
| **Vibration motor (haptics)** | Distinct buzz when an expired item is in the cart, noticeable in a noisy shop. |

### iQOO Office Kit Integration
| Use | How |
| :--- | :--- |
| **Customer screen** | Bill is mirrored to a laptop/monitor facing the customer. |
| **Leak dashboard** | Laptop shows short deliveries, expiring stock, and ₹ saved. |
| **Month-end Export** | Day-wise/month-wise register moves to the laptop as Excel. |

---

## 9. Offline-First Architecture
The core workflow must work in **airplane mode**. No cloud API is required for delivery counting, bill comparison, expiry checking, sale blocking, or stock queries.

---

## 10. The 2-Minute Live Demo

**Setup:** Real FMCG packets, one short vendor delivery, real printed bill with mismatch, one expired packet.
*   **0:00:** Establish problem with a 15s Hindi video. **AIRPLANE MODE ON.**
*   **0:15 (Catch 1 - Short Delivery):** Vendor Bill says 24. Photo shows 20. Phone flags: **SHORT BY 4 - ₹56**.
*   **0:45 (Catch 2 - Expired Sale):** Customer buys 3 packets. One is expired. Phone **BUZZES**, screen blocks sale.
*   **1:10 (Catch 3 - Ask in Hindi):** "Is mahine kya expire hoga?" System answers from SQLite.
*   **1:35:** Prove it's on-device (Airplane mode still on) and show "Money protected today: ₹___".

---

## 11. Final Build Priority

### P0 - MUST WORK
1. Delivery photo & Product counting
2. Bill OCR & Physical vs Bill comparison (Shortage alert)
3. SQLite inventory
4. Expiry OCR & Expired-sale blocking
5. Hindi stock query (Airplane-mode demo)

### P1 - SHOULD WORK
6. Confidence scores & Local LLM Hindi response
7. Expiry alerts & Vendor history

### P2 - NICE TO HAVE
8. Laptop customer display (Office Kit) & Excel export

---

## 12. The One-Line Pitch

> **"Kirana owners don't need another billing app. They need to know when money disappears: our iQOO 15 photographs the goods, verifies them against the vendor bill, watches expiry, blocks expired sales, and answers the owner in Hindi - entirely on-device."**
