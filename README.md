# Home Credit LAP Qualification: AI Voice Agent

An AI voice agent built on **Retell AI** (single-prompt agent) that calls existing Home Credit customers, presents a pre-approved **Loan Against Property (LAP)** offer of up to ₹75,00,000, and runs a 7-point preliminary eligibility check before handing the lead to a senior loan expert.

## Overview

| Item | Detail |
|---|---|
| Platform | Retell AI |
| Agent type | Single-prompt voice agent |
| Languages | English, Hindi |
| Use case | LAP lead qualification |
| Agent persona | Priya (female), Home Credit |

## Repository contents

| File | Description |
|---|---|
| `system_prompt.md` | Full system prompt used in the agent |
| `README.md` | This document |

## Call flow

1. **Greeting and identification:** Verifies the customer. If busy, books a callback time.
2. **Offer presentation:** Explains the loyalty-based pre-approved offer of up to 75 lakh.
3. **Eligibility checklist (7 points):**
   1. Property type
   2. Ownership status
   3. Document availability
   4. Loan amount
   5. Occupation and income mode
   6. Market value
   7. Tenure
4. **Handoff:** Once all 7 answers are collected and all criteria are met, the customer is told a senior loan expert will call back with exact interest rates.

## Eligibility rules

| Field | Eligible | Not eligible |
|---|---|---|
| Property type | Residential, Commercial, Industrial | Agricultural |
| Ownership | Sole or Joint | None |
| Documents | Original documents available | No originals |
| Loan amount | Up to 75,00,000 | Above limit (offered the maximum first) |
| Income | Salaried or Self-Employed, received in Bank | Cash income |
| Market value | Any (recorded only) | None |
| Tenure | 3 to 15 years | Below 3 or above 15 years |

## Business logic

- **Immediate disqualification:** If any answer fails a criterion, the agent politely says the customer does not meet the criteria for this offer and ends the call.
- **Loan transfer switch:** If the customer mentions an existing loan on the property or wants to reduce their EMI, the agent says a loan-transfer specialist will contact them shortly and ends the call.
- **Loan amount limit:** For requests above 75 lakh, the agent explains the limit and asks whether to proceed with the maximum.
- **Handoff gate:** The agent moves to the final handoff only after all 7 questions are answered. Any skipped question is asked before closing.
- **Out-of-order answers:** Details volunteered early are captured and never asked again.

## Dynamic variables

| Variable | Purpose |
|---|---|
| `company_name` | Company name |
| `customer_name` | Customer being called |
| `agent_name` | Agent's name |
| `agent_gender` | Agent's gender, followed strictly, including Hindi grammar |
| `current_date`, `current_day`, `current_time` | Context for callback scheduling |
| `additional_context_from_rag` | Product knowledge for answering questions |
| `language_to_speak` | English or Hindi |
| `conversation_history` | Conversation log (supplied by Retell) |
| `customer_utterance` | Customer's latest message (supplied by Retell) |

## Edge cases handled

- Busy customer with a callback request
- Wrong person or family member answering (no offer details shared)
- Questions about interest rates (never quoted)
- "Are you a robot?" and "Is this a scam?"
- Customer asking for a human, refusing a question, going silent, or being abusive
- Unit conversions (crore/lakh, months/years, Hindi numbers)
- Customer correcting an earlier answer (eligibility re-checked)
- Do-not-call requests
- Mixed income (part bank, part cash)
- Loans on other assets that do not trigger the transfer logic

## Test calls

Each link includes the recording and transcript.

| # | Scenario | Result | Link |
|---|---|---|---|
| 1 | Happy path with out-of-order answers | Eligible, handoff | https://drive.google.com/file/d/104DFS53TqbUxseG9ewlwBbEUusjQn-k7/view?usp=share_link |
| 2 | Busy, callback requested | Callback confirmed, call ends | https://drive.google.com/file/d/1PzaG17BgEaKqk7d02eoT9sva32LFjXh0/view?usp=share_link |
| 3 | Agricultural property | Disqualified | https://drive.google.com/file/d/1VYdtVBLasgBR4k7-OtOnOOIlQpTaxPYG/view?usp=share_link |
| 4 | Cash income | Disqualified | https://drive.google.com/file/d/1zPp9tiPOyj2zycrhSjAvBncYW0bVu5oL/view?usp=share_link |
| 5 | Existing loan / EMI reduction | Transfer message, call ends | https://drive.google.com/file/d/1pMKPLIGVPoUfXmnrPpQWG2beQe21pN-Z/view?usp=share_link|
| 6 | Loan above 75 lakh | Offered 75 lakh maximum, continued | https://drive.google.com/file/d/1MxNUfzBlK2SpRvFY2HehcGXbINNvkhzF/view?usp=share_link |
| 7 | Out-of-order answers | No repeated questions | https://drive.google.com/file/d/1-WY7SJCKNooT3gwy3gQ68KOFBgNQNy1A/view?usp=share_link |
| 8 | Hindi call | Hindi with correct gender forms |https://drive.google.com/file/d/1VggfyJ74ZfT-XnlLTYq-iDEian8pz5gH/view?usp=share_link|

**Screen recording of all calls:** drive https://drive.google.com/file/d/1I2XZEIY3Y3LnrEWqe9g-9Kgydx3R9rip/view?usp=share_link

## Design decisions

- **Per-turn state tracking:** The agent reviews the conversation every turn and fills any checklist field the customer has already answered.
- **Priority rules:** Transfer logic overrides disqualification. Over-limit amounts get the maximum offered before any rejection.
- **Gaps in the brief:** The brief has no rule for market value or loan-to-value, so the agent records the value without judging it. A customer who declines the 75 lakh maximum is politely ended.
- **Privacy and safety:** No offer details are shared with anyone except the verified customer. The agent never asks for OTP, PIN, passwords, or ID numbers, and never quotes rates, fees, or approvals.
- **Voice style:** Short turns (one to two sentences), one question at a time, and no markdown or lists, since this is a spoken call.

## How to reproduce

1. Create a single-prompt agent in Retell AI.
2. Paste the contents of `system_prompt.md` into the agent instructions.
3. Set the opening line: `Hello, this is {{agent_name}} from {{company_name}}. Am I speaking with {{customer_name}}?`
4. Set the dynamic variables, for example `agent_name` = Priya, `company_name` = Home Credit, `customer_name` = Rahul Sharma, `agent_gender` = female, `language_to_speak` = English.
5. Run a web call with **Run Test**.
