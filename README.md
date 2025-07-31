# CST8917 Lab 4 – Real-Time Trip Event Analysis

## Overview
This project implements a real-time event-driven system to monitor taxi trip data for suspicious or interesting patterns. Using Azure services such as Event Hub, Azure Functions, and Logic Apps, the system detects anomalies like cash payments, large passenger groups, short-distance fraud, and vendor irregularities.

### Use Cases
- Spotting suspiciously short rides paid with cash
- Identifying high passenger count trips
- Flagging vendors with irregular patterns

---

##  Architecture

```
Event Hub (trip-events)
    ↓
Azure Logic App (trigger on batch of events)
    ↓
Azure Function (trip-analysis-fn → analyze_trip)
    ↓
Logic App "For Each" output
    ├─ If not interesting →  Post "Trip Analyzed – No Issues" card to Microsoft Teams
    ├─ If interesting AND suspicious →  Post "Suspicious Trip" card
    └─ If interesting but not suspicious →  Post "Interesting Trip" card
```

---

##  Azure Resources

| Resource Type       | Name                      |
|---------------------|---------------------------|
| Resource Group      | `cst8917lab4`             |
| Event Hub Namespace |  Event Hub (e.g., `trip-events`) |
| Azure Function App  | `trip-analysis-fn`        |
| Function Name       | `analyze_trip`            |
| Logic App           | Your Logic App            |
| Microsoft Teams     | Adaptive Cards (via Logic App) |

---

##  Azure Function Logic

The Azure Function is triggered by HTTP via Logic App. It receives a batch of trip events, analyzes each one, and returns metadata flags.

###  Logic:
```python
if distance > 10:
    insights.append("LongTrip")
if passenger_count > 4:
    insights.append("GroupRide")
if payment == "2":
    insights.append("CashPayment")
if payment == "2" and distance < 1:
    insights.append("SuspiciousVendorActivity")
```

###  Returned Fields:
- `vendorID`, `tripDistance`, `passengerCount`, `paymentType`
- `insights` → List of detected tags
- `isInteresting` → True if any insights were found
- `summary` → Short description of issues found or "Trip normal"

---

##  Logic App Flow

### Trigger
- Type: **Event Hub (batch)**
- Event Hub: `trip-events` with Consumer Group `$Default`

### Actions
1. **Call Azure Function `analyze_trip`**
   - Passes batch payload to Function

2. **For Each loop** on Function results:
   - Add `Condition`:  
     Expression:  
     ```text
     @equals(items('For_each')?['isInteresting'], true)
     ```

   - **If True (Interesting)**:
     - Inner Condition:  
       ```text
       @contains(items('For_each')?['insights'], 'SuspiciousVendorActivity')
       ```
       - True → Post  Suspicious Trip Card
       - False → Post Interesting Trip Card

   - **Else (Not Interesting)**:
     - Post "Trip Analyzed – No Issues" card

### Microsoft Teams Integration
- All cards are Adaptive Cards posted using "Post adaptive card in chat or channel".
- Cards contain facts: Vendor ID, Distance, PaymentType, PassengerCount, and Summary or Insights.

---

##  Example Input & Output

### Sample Input (Event Payload to Event Hub):
```json
{
  "vendorID": "2",
    "tpepPickupDateTime": 1528119858000,
    "tpepDropoffDateTime": 1528121148000,
    "passengerCount": 1,
    "tripDistance": 1.24,
    "puLocationId": "186",
    "doLocationId": "230",
    "startLon": null,
    "startLat": null,
    "endLon": null,
    "endLat": null,
    "rateCodeId": 1,
    "storeAndFwdFlag": "N",
    "paymentType": "1",
    "fareAmount": 13.5,
    "extra": 0,
    "mtaTax": 0.5,
    "improvementSurcharge": "0.3",
    "tipAmount": 2.86,
    "tollsAmount": 0,
    "totalAmount": 17.16
}
```

### Sample Output (from Azure Function):
```json
[
  {
    "vendorID": "2",
    "tripDistance": 0.4,
    "passengerCount": 1,
    "paymentType": "2",
    "insights": [
      "CashPayment",
      "SuspiciousVendorActivity"
    ],
    "isInteresting": true,
    "summary": "2 flags: CashPayment, SuspiciousVendorActivity"
  }
]
```

---

##  Teams Output

Based on the result:
- **Suspicious activity** triggers a  card.
- **Interesting but not suspicious** triggers a  card.
- **Normal trip** triggers a card.

---
##  Challenges Faced

###  Issue with Teams Card Labeling
One challenge encountered was that the Microsoft Teams Adaptive Cards were posting the same “Interesting Trip” card even when the activity was suspicious (e.g., cash payment with distance < 1 mile). This occurred despite the Azure Function correctly identifying and returning different `insights` and `isInteresting` values.

---

##  Demo Video

Watch the system in action:  
🎥 [YouTube Demo Link](https://youtu.be/WzOVzLbAnrY)

---

##  Project Structure (GitHub Repo)

```
.
├── logic-app/
│   └── logicappworkflow.json
├── function/
│   └── analyze_trip.py
├── screenshots/
│   └── trip-monitoring-logic-flow.png
├── README.md
```
