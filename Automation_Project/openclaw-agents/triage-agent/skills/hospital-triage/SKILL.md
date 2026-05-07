\# Hospital Triage Skill



\## Overview

This workflow checks emergency patients and assigns priority levels based on symptoms and vital signs.



\---



\## Workflow Steps



\### 1. Red Flag Check

Check for serious symptoms:

\- chest pain

\- not breathing

\- unconscious

\- seizure

\- stroke

\- severe bleeding



If detected:

\- Set Priority = P1

\- Skip directly to notification step



\---



\### 2. Vital Signs Check

Check:

\- Heart Rate > 120 bpm

\- SpO2 < 92%



If true:

\- Set Priority = P1



\---



\### 3. LLM Triage Assessment

Send:

\- patient complaint

\- vital signs



LLM returns:

\- priority

\- reasoning

\- recommended action

\- confidence score



\---



\### 4. Confidence Validation

If confidence < 65:

\- Mark for manual review

\- Notify admin



\---



\### 5. Token Generation

Generate patient token.



Example:

\- TRG-1234



Also calculate estimated wait time.



\---



\### 6. Database Storage

Store:

\- name

\- age

\- phone

\- complaint

\- vitals

\- priority

\- token

\- confidence



Database:

\- Airtable

\- PostgreSQL



\---



\### 7. Notification Routing



\#### P1

\- Immediate doctor alert via SMS



\#### P2

\- WhatsApp notification



\#### P3

\- Standard queue confirmation



\---



\## Expected Output



```json

{

&#x20; "token": "TRG-1234",

&#x20; "priority": "P1",

&#x20; "estimatedWaitMinutes": 5,

&#x20; "triageNote": "Red flag detected: chest pain",

&#x20; "confidence": 95

}

```

