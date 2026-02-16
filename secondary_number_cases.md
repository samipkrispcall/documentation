# Contents

- [I. Create Contact](#i-create-contact)
  - [Case 1: CRM has One Main Phone Field + Separate Secondary Phone Fields](#case-1-crm-has-one-main-phone-field--separate-secondary-phone-fields)
  - [Case 2: CRM has a List/Array Type Phone Field](#case-2-crm-has-a-listarray-type-phone-field)
- [II. Update Contact](#ii-update-contact)
  - [Case 1: CRM has One Main Phone Field + Separate Secondary Phone Fields](#case-1-crm-has-one-main-phone-field--separate-secondary-phone-fields-1)
  - [Case 2: CRM has List/Array Type Phone Field](#case-2-crm-has-listarray-type-phone-field)
- [III. Bulk Contact Sync](#iii-bulk-contact-sync)
- [IV. Delete Contact Sync (Bulk)](#iv-delete-contact-sync-bulk)
- [V. Webhook Contact Sync](#v-webhook-contact-sync)

---

# I. Create Contact

## Case 1: CRM has One Main Phone Field + Separate Secondary Phone Fields

### ➜ Situation

**KrispCall**
- 1 Primary Number, Up to 5 Secondary Numbers

**CRM**
- 1 Main phone field (e.g., Phone)
- Multiple separate secondary fields (e.g., Mobile, Alternate, Custom Phone)

---

### ➜ Current Implementation

- Only the **Primary Number** from KrispCall is used to check if the contact already exists in CRM.
- The system searches the primary number across all CRM phone-related fields.
- If a match is found:
  - Contact creation is skipped.
  - Secondary numbers are NOT updated.
- If no match is found:
  - A new contact is created.
  - Primary number → Main phone field.
  - Secondary numbers → Secondary phone fields (if available).

---

### ➜ Limitations

- Searching contacts by secondary numbers is not implemented
- If contact already exists, secondary numbers are not merged.
- Primary and secondary numbers are not automatically organized in CRM.
```Example: If KrispCall’s primary number exists in a CRM secondary field, the system currently skips it and does not automatically arrange or merge the numbers into the proper CRM fields.```

---

## Case 2: CRM has a List/Array Type Phone Field

### ➜ Situation

**KrispCall**
- 1 Primary Number
- Up to 5 Secondary Numbers

**CRM**
- Single phone field that supports multiple numbers (array/list format)

---

### ➜ Current Implementation

- Only the **Primary Number** is used to check if the contact exists in CRM.
- The system searches if the primary number exists in the CRM phone list.
- If a match is found:
  - Contact creation is skipped.
  - Secondary numbers are NOT appended to the existing phone list.
- If no match is found:
  - A new contact is created.
  - All numbers (Primary + Secondary) are added to the phone list.

---

### ➜ Limitations

- Searching contacts by secondary numbers is not implemented
- If contact already exists, missing numbers are not merged into existing contact: it just skips.
---

# II. Update Contact

## Case 1: CRM has One Main Phone Field + Separate Secondary Phone Fields

### ➜ Current Implementation

#### Contact Search Logic

- Only the **Primary Number** from KrispCall is used for matching.
- The system searches the primary number **only in the CRM main phone field**.
- Custom/secondary phone fields are **NOT considered during update search**.

---

#### If No Match Found

- The system triggers the **Create Contact** event.
- ⚠️ However:
  - CreateContact searches across **all phone-related fields**.
  - If the number exists in a secondary/custom field:
    - Contact creation is skipped.
    - Update does NOT happen.
    - The contact remains unchanged.

#### 🔴 Resulting Issue

If KrispCall primary number exists in a CRM custom field:
- Update is switched to Create.
- Create is skipped.
- No action occurs.

---

#### If Match Found

- The existing CRM contact is fully overwritten.
- The system does **not preserve existing CRM phone numbers**.
- All phone fields are replaced with KrispCall data.

#### ➜ Example

#### KrispCall Updated Contact

```
Primary:    +11111111111
Secondary:  +12222222222
Secondary:  +13333333333
Secondary:  +14444444444
```

#### Existing CRM Contact

```
Main:       +11111111111
Custom:     +9779822222222
Custom:     +9779833333333
Custom:     +9779844444444
Custom:     +9779855555555
```

#### Final CRM After Update

```
Main:       +11111111111
Custom:     +12222222222
Custom:     +13333333333
Custom:     +14444444444
```

❌ Previously existing CRM numbers are permanently lost:

```
+9779822222222
+9779833333333
+9779844444444
+9779855555555
```

---

### ➜ Limitations
- If primary number exists in custom field → No update happens.
- Full overwrite behavior removes previous CRM phone data.
- Risk of data loss from CRM side for some cases.

---

## Case 2: CRM has List/Array Type Phone Field

### ➜ Current Implementation

#### Contact Search Logic

- The system searches for the contact using the KrispCall numbers.
- Since CRM stores numbers in an array:
  - Matching works across all stored numbers.
- Unlike Case 1, search is consistent and does not skip fields.

---

#### If No Match Found

- CreateContact event is triggered.
- All KrispCall numbers (Primary + Secondary) are added to CRM list.
- This flow works correctly.

---

#### If Match Found

- The matched contact is fully updated.
- The CRM phone array is completely replaced with KrispCall numbers.
- Existing numbers not present in KrispCall are removed.


### ➜ Example

#### KrispCall Contact

```
Primary:    +11111111111
Secondary:  +12222222222
Secondary:  +13333333333
Secondary:  +14444444444
```

#### Existing CRM Phone List

```
[
 +9779800000000,
 +9779811111111,
 +13333333333,
 +9779855555555
]
```

#### Matching

- `+13333333333` is found in CRM list.
- Contact is matched successfully.

---

#### Final CRM After Update

```
[
 +11111111111,
 +12222222222,
 +13333333333,
 +14444444444
]
```

❌ Removed CRM numbers:

```
+9779800000000
+9779811111111
+9779855555555
```

---

### ➜ Limitations
- Full replacement strategy causes data loss.

---



# III. Bulk Contact Sync

### ➜ Limitation (Case 1 specific)

- **Secondary numbers are ignored during bulk create**  
    - While processing bulk contacts, only the primary number is synced. Secondary numbers are skipped because adding them would require separate API calls. (In some cases, multiple numbers can be added with a single API call, which is not considederd as limitation.)

---

### ➜ Limitation (Both Case 1 & Case 2)

- **Only one phone number retrieved per contact**  
    - When fetching all contacts from CRM (```get_all_contacts```), only one phone number is processed per contact. Other numbers are ignored. This can lead to **duplicate contact creation** (same issue exists in V2).

---

# IV. Delete Contact Sync (Bulk)

### ➜ Limitation (Both Case 1 & Case 2)

- **Only one phone number retrieved per contact**  
    - When fetching all contacts from CRM (```get_all_contacts```), only one phone number is processed per contact. Other numbers are ignored. This can lead to **no contact deletion** if that contact number exist in other phone-fields (*case1*)/phone-list (*case2*) (same issue exists in V2).

---

# V. Webhook Contact Sync

### ➜ Situation

- Some CRM platforms send **contact data via webhook** to KrispCall.
- If **only one contact** is received per webhook request, it works fine.
- Concern when **multiple contacts** are sent in a single webhook POST request.

---

### ➜ Current Implementation

- The webhook handler returns a `Contact` object.
- The `Contact` model currently has fields only for **one primary contact**:

```python
@dataclass
class Contact(BaseDict):
    number: str
    crm_platform: str
    name: Optional[str] = None
    email: Optional[str] = None
    company: Optional[str] = None
    address: Optional[str] = None
    crm_contact_id: Optional[str] = None
    agent: Optional[str] = None
    workspace_id: Optional[str] = None
```

### ➜ Limitations
- If the webhook sends multiple contacts in the request, only one contact is processed; the rest are ignored.
- The `Contact` model does not have fields for secondary numbers, so any additional numbers from the webhook are ignored.
