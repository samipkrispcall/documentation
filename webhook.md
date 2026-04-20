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

## Solution 3: Store CRM Contact ID

### Implementation
- Save CRM contact ID in KrispCall 
- save the id when creating, while updating, while deleting the contacts in CRM and monitor in our mongodb.
- Use this ID to match contacts instead of phone/name/email whether the contact is active or deleted.

### Problem
- Not all CRMs provide update/delete webhooks. This means KrispCall has no visibility into whether a contact has been modified or removed on the CRM side.

- As a result, the contact ID stored in KrispCall (e.g., in MongoDB) remains marked as active even if it has been deleted in the CRM.

- When KrispCall attempts to perform operations (update, fetch, etc.) using this outdated CRM contact ID, the CRM may return errors such as:
  - `404 Not Found` (contact no longer exists)
  - Other API errors depending on the CRM behavior

- Since there is no reliable way to detect these changes:
  - Stale or invalid contact mappings remain in the system
  - Error logs can increase significantly due to repeated failed API calls
  - Data synchronization becomes inconsistent between KrispCall and CRM
  - There is no guaranteed way to recover or clean up invalid records automatically
