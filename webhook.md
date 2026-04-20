# Webhook Contact Sync – Implementation & Existing Problems

## Current Implementation

When a webhook is triggered from CRM, it sends:
- name
- email
- phone

### Matching Logic in KrispCall

1. Search contact by **phone**
   - If found → update that contact

2. If phone not found:
   - Search by **name OR email**
   - If match found → update that contact

3. If no match found:
   - Create new contact

---

## Existing Problems

### Case 1: Same Name
- CRM creates a new contact with same name but different phone
- Webhook triggers
- System matches by name
- Existing contact gets **overwritten**
- New contact is **not created**

---

### Case 2: Same Email
- CRM creates a new contact with same email but different phone
- Webhook triggers
- System matches by email
- Existing contact gets **overwritten**
- New contact is **not created**

---

### Case 3: All Fields Updated (name, email, phone)
- CRM updates all fields
- Webhook triggers
- No match found
- System **creates a new contact**
- Results in duplicate contacts

---

## Solution 1: Match Only by Phone

### Implementation
- Remove name and email matching
- Only match using phone number

### Problem
- If phone number is changed in CRM:
  - No match found
  - New contact is created instead of updating existing one

---

## Solution 2: Keep Current Logic

### Implementation
- Keep matching by phone → name/email

### Problem
- Same name/email causes **wrong overwrites**
- Cannot handle duplicate name/email scenarios properly

---
