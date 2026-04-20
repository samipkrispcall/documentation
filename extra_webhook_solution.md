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
