

1. Protocol fundamentals (ASTM vs HL7)
2. Integration architecture
3. Data flow (bi-directional)
4. Parser & adapter design
5. Mapping to your system (very important)
6. Error handling & edge cases
7. Scalability strategy

---

# 🧠 1. WHY THIS LAYER IS CRITICAL

Lab analyzers:

* Don’t speak modern APIs
* Use **serial/TCP protocols**
* Send raw, messy, inconsistent data

👉 Your job:

> Convert unreliable machine output → clean, structured, validated data

---

# 🔬 2. PROTOCOLS YOU MUST SUPPORT

---

## 🧪 ASTM (Older but very common)

![Image](https://images.openai.com/static-rsc-4/aEof-6gZHXmt7rOeTPfRH97Wed01sEerDV3SdUP8w4Zkp2iS25k0DErSq38vMQ9VAhoV7CCzT4IM_67cr5l6V5_pGswNCfBxZ6cVT2iy_8tEC2VLnNC_n48wpUEr4VTxyheY1Iik-AJs_RiqyTUgLjUYLuG2fazKqdtAVf1s7yuvB-Wt9cvseTD9TtLLTpPF?purpose=fullsize)

![Image](https://images.openai.com/static-rsc-4/k9EumlY4y9uV3LAdN23-7s_2sri_9QMyhhRr1tWb69k0Wzgqq4isNxqSb-0tN2zGnGL8_BHuLH9VdPiLX7FJJ-HHuF-ZtHia-nHk69HSchJQLd5pCwe9n2UHZnyjDeenvAfuGQHFOCbgplvar3LTrbRQdzmWuh11NOmG6GMWZYMmc04KfMBcOh-jSCMfpOX5?purpose=fullsize)

![Image](https://images.openai.com/static-rsc-4/0FvFbG3ZQsViy6Y4KHltOTqYfm3KRMS0caQoAWjlX94we7vjzPykCUsKKzZzUv2MlFyWHb1RtvCJ0e1S1YeMEh0mfEvIIjfCJ2UOuZpUiPvMv4hr0xOL7PcK5G-KG6omoeT85TIy7RmU5iogdu48uxh2Fq7R14XDCLuI9P0w0QI3R6ztG1qt-EzVlOjfMnx-?purpose=fullsize)

![Image](https://images.openai.com/static-rsc-4/MHSg_9e5zP2hn22gjJyNwMB6hzNaA3wU7VfCajEwgtOm00dfcJJvDNXBjW8m3GU4g9pvoj2uWzc0i-kssQG3GLS0FpKRqQh0ui8iIQcZuSIL0Q7PxH_e5UbwfJ99bzfqGTsuAHLT0_3B1zytcCbAl2xXlzvwpfRcodXng1h2qPbSG0WeRfDh6j3ITsVBJYFS?purpose=fullsize)

![Image](https://images.openai.com/static-rsc-4/e5orHvTC4g8Sw_aDYzGKYLcFfZ0tyNJ5561tsdwlLzUsTFR_mrG7RLf4KGSxA_epRILcOhsqIfQ0wY070hqvUwXy2Sw-DkVKy-FcazeX37-OVRfnxP1KRE5vECuRw-nrxJKsUZLH2SMWcq6ectfp5ErSKRWMRZET9tZ144aUVMi71bHFu1cVvWrOA7cMFntj?purpose=fullsize)

![Image](https://images.openai.com/static-rsc-4/ItDq9HpEzvSOqgwUvf9fviblZrys8H4bwc9pRllx090PMqmeUZJJ2z38TU9bE7hX1TABTD2CaNt6EF8Y-o3CGG27XjUdDa5h5rWGHeW2m26JhVeEJ1t6g4wl4w_spy8EkiXwTKJypXrrTP6Pk2ipFnznmHIUC5clLSs2KRt0_6wIQ2uz5hAi9X1LE4-wR23i?purpose=fullsize)

![Image](https://images.openai.com/static-rsc-4/JLp04Hd5yk60ekgp4ctyioV7XxeZFHKOavJNQ86G8vSU6eWD8d6R7eewMiPTLxxjdLtNkVZfhRtWdDD6EyuLS9VeOV61PE6KEga-5BsnTyK5j2Y2xAsD-ICHUw_itwlZS2LWPIfomKYPvyVBVGRsOCJEAej27vzJ2kcFmL96lR63kuH6j8zwPwK_IxLTU7ej?purpose=fullsize)

### Characteristics:

* Serial (RS232) or TCP
* Message-based
* Frame control (STX, ETX, ACK, NAK)

### Message Structure:

```text
<STX>H|...<CR>
P|...<CR>
O|...<CR>
R|...<CR>
L|...<ETX>
```

---

## 🏥 HL7 (Modern hospital standard)

![Image](https://images.openai.com/static-rsc-4/tJUUf4nliSRZ_ekjVTtOBdx3Y2KFRPljDlYvUGJisxgnRfa8RSYhHFS65yLlqlpeYEN9J8u6Xx-PN5RdNEARYWpFttx-JWL-hfbLDTnBuec7wTJGhTTws6uC48BeP2Vd8DuFdyPzRuCEUQHwtBPYsn1RIJm5TFAnCploq2fLXfUCLGSkLSTkMs-2whK8u4Jj?purpose=fullsize)

![Image](https://images.openai.com/static-rsc-4/kpSYXN0rySTFz7dd5Gr1b0QOHCTT9LbHt7-yUCiEGEH_fRoeaCSAErL2H9me1ADsrNsDpyqnNSJomxOSCGbi3SmYl4vcJsavrVyQBSiE4KGmqFyycZmUvxTmNzl4U4dW2nm0Uvyc40K1dr8Q3qSq-_U-bYpURgFkxOLlFDPPPblJMy58ut2cFh0JNrOtggnT?purpose=fullsize)

![Image](https://images.openai.com/static-rsc-4/4Dlqy3ES8zasWUMsvqAdHzj9CqM6Zv8DmjoC4WW5UegrsabOr7IIodQmT2ugPOfDP6ASMgSwBhckyBu1qV6fsRvHs0xsqnFEtVl_lQYmV5mkh7fgalps7OH5JD0jjnYwJICx0FkjwoRX5W44f8lYh1W2bpU6xyAxOrNnJv-JBRyG9nqMFPcal5B05pe9I0eW?purpose=fullsize)

![Image](https://images.openai.com/static-rsc-4/mZrZhVnTeZz2LdXdy2spfvWsISasMd0OkAP2izrGHkjh-bzoFsK-pE9Qg0Ndpsrf7sP8NJHGf6J6vxxb6QLfC1KoWRQyS0EFD63XJ6ZVLiSdk0XHXRjBt5UgQpBnWXCg5k3qH0NfU_sl6jnkO9soHa_yNJNqiM-kX1JiVtJ9eY2dei69a1umpRxHesmERQ8J?purpose=fullsize)

![Image](https://images.openai.com/static-rsc-4/DluiM6sfiRTVCmJOUEY8yvFLgffxblOgOp0bGpQXxaQ4DWL3PJz_hjFysuqj1-rMx8Ml4Jn6DS5ozZ897Z-ddur7ii53RLMJ9L0E8oJ9JfyWTcpGOBgBbNVql6ZoM9K84pPV2sohOCIL3d3PUckCChzAnFJTne_I1mEr6vDqvCq5xIO562XZYMC3Sx78Ruti?purpose=fullsize)

![Image](https://images.openai.com/static-rsc-4/4vh2a9GzIEFrCDxY8binSuISYkTREhy884VDDz6FlyShVJ8ZaBN8hHDVTLGLQxgmc-QrFnVFk-Ml65LMlJETsac093YzE74OTChwQ8niTWZQ4pnz5mKndl3PkMguUo1XsNVJ2SaLWHhxM2rdEIzDHh98aVqraMpCKgwhxtJad3t_8wg6spCN76VZRdmMS-iT?purpose=fullsize)

![Image](https://images.openai.com/static-rsc-4/-KoQz0UcVRdb9ZUHnfaGrSfBxYZskxZycamhXwQu7haFxQmw4x8IcZGBA2QYWoiRrc8bqpyQKeruU5kLRoiSBuPOpMP7H3HC9Fv_UqFFtdW2iBqOUQCK6b21EGfidY9Mw_KozvY6ky_ILxVJRhDRsvjrpTIgRo9Ap1fKQ5qlr9AE5cDvp2mvqFB77jEvLsoA?purpose=fullsize)

### Common Messages:

* `ORM` → Order message
* `ORU` → Result message

### Example:

```text
MSH|^~\&|LAB|...  
PID|1||12345||Rahul Kumar  
OBR|1||...|CBC  
OBX|1|NM|HB||13.5|g/dL
```

---

# 🏗️ 3. INTEGRATION ARCHITECTURE

---

## Core Component: Instrument Integration Service

### Subcomponents:

1. **Device Connector Layer**
2. **Protocol Parser (ASTM/HL7)**
3. **Normalization Layer**
4. **Mapping Engine**
5. **Event Publisher**
6. **Command Sender (for orders)**

---

## High-Level Flow

```text
Analyzer → Connector → Parser → Normalizer → Mapper → Event → Core System
```

---

# 🔌 4. DEVICE CONNECTOR LAYER

---

## Types:

### Serial (RS232)

* COM ports
* Requires hardware interface

### TCP/IP

* Socket-based communication

---

## Responsibilities:

* Maintain connection
* Handle retries
* Buffer incoming data

---

## Example:

```text
Machine → TCP Socket → Listener Service
```

---

# 📦 5. PROTOCOL PARSER

---

## ASTM Parser

### Responsibilities:

* Frame detection (STX/ETX)
* Checksum validation
* Segment parsing

---

## HL7 Parser

### Responsibilities:

* Segment splitting (`\r`)
* Field parsing (`|`)
* Component parsing (`^`)

---

## Output (Unified Format)

```json
{
  "patient_id": "12345",
  "test_code": "CBC",
  "parameters": [
    {"code": "HB", "value": "13.5", "unit": "g/dL"}
  ]
}
```

---

# 🔄 6. NORMALIZATION LAYER

Machines are inconsistent:

| Machine | Hemoglobin |
| ------- | ---------- |
| Sysmex  | HB         |
| Beckman | HGB        |
| Others  | Hemoglobin |

👉 Normalize to:

```json
"HB"
```

---

## Strategy:

* Maintain **machine-specific mapping configs**

---

# 🧠 7. MAPPING ENGINE (CRITICAL)

This connects machine data → your DB schema.

---

## Mapping Table Example

```json
{
  "machine": "sysmex_xn1000",
  "mappings": {
    "HB": "hemoglobin",
    "WBC": "white_blood_cells"
  }
}
```

---

## Flow:

```text
Machine Code → Standard Code → DB Parameter ID
```

---

# 🔁 8. BI-DIRECTIONAL COMMUNICATION

---

## 🟢 Inbound (Results)

Machine → LIS

Event:

```json
{
  "event": "result.received",
  "sample_id": "uuid",
  "values": [...]
}
```

---

## 🔵 Outbound (Orders)

LIS → Machine

Example HL7 ORM:

```text
MSH|...
PID|...
OBR|...
```

---

# 🔗 9. MATCHING LOGIC (VERY IMPORTANT)

How do you know which sample result belongs to?

---

## Matching Strategies:

### 1. Barcode (BEST)

* Machine reads barcode

### 2. Sample ID

* Sent with order

### 3. Manual fallback

* UI for technician

---

---

# ⚙️ 10. ERROR HANDLING

---

## Common Issues:

* Partial messages
* Corrupted frames
* Duplicate results
* Missing sample ID

---

## Solutions:

### ✅ Retry logic

### ✅ Deduplication (idempotency key)

### ✅ Dead-letter queue

### ✅ Manual review queue

---

# 🧪 11. REAL DATA FLOW (END-TO-END)

---

## Scenario: CBC Analyzer

```text
1. Sample scanned
2. Analyzer runs test
3. Machine sends ASTM message
4. Connector receives
5. Parser extracts data
6. Mapping engine normalizes
7. Match sample via barcode
8. Store results
9. Emit event → workflow engine
```

---

# ⚡ 12. SCALABILITY DESIGN

---

## Problem:

Thousands of machines, high throughput

---

## Solution:

### 1. Device Gateway Service

* Handles multiple machines

### 2. Queue-based ingestion

* Kafka topic: `instrument.raw`

### 3. Workers

* Parse + map asynchronously

---

## Partition Strategy:

* By `tenant_id`
* By `machine_id`

---

# 🧠 13. CONFIGURATION SYSTEM (IMPORTANT)

Each lab has different machines.

---

## Example Config:

```json
{
  "machine_id": "abc123",
  "protocol": "ASTM",
  "connection": {
    "type": "TCP",
    "host": "192.168.1.10",
    "port": 5000
  },
  "mapping_profile": "sysmex_xn1000"
}
```

---

# 🔐 14. SECURITY

* Device authentication (IP whitelisting)
* Network isolation (VPC)
* Data encryption

---

# 🚀 15. WHAT MAKES YOUR SYSTEM 10x BETTER

---

## 🔥 Smart Features:

### ✅ Auto-calibration detection

* Detect abnormal machine behavior

### ✅ Real-time validation

* Flag impossible values instantly

### ✅ Plug-and-play onboarding

* Add new machine in minutes

### ✅ Simulation mode

* Test without real hardware

---
