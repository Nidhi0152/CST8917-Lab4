# CST8917 Lab 4 – Real-Time Trip Event Analysis

## ✅ Overview
This project uses Azure Event Hub, Azure Functions, and Azure Logic Apps to detect and report suspicious or interesting taxi trips in real time.

---

## 🚀 Architecture
- Event Hub receives incoming trip events.
- Azure Function analyzes trip metrics (passenger count, distance, payment type).
- Logic App handles trip routing:
  - Sends Adaptive Cards to Microsoft Teams based on severity.

---

## 🔧 Azure Function Logic
The function checks:
- Distance > 10 → LongTrip
- Passenger count > 4 → GroupRide
- PaymentType == 2 → CashPayment
- PaymentType == 2 AND Distance < 1 → SuspiciousVendorActivity

It returns:
- `insights[]`
- `isInteresting`
- `summary` of flags

---

## 🔁 Logic App Steps
1. Trigger: Event received from Event Hub
2. Action: Send to Function
3. Loop over each result
4. Conditional branches:
   - If not interesting → ✅ No Issues card
   - If interesting → 🚨 Interesting or ⚠️ Suspicious cards
5. Send Adaptive Cards to Teams Channel

---

## 📥 Sample Input
```json
{
  "ContentData": {
    "vendorID": "2",
    "tripDistance": 0.5,
    "passengerCount": 1,
    "paymentType": "2"
  }
}
