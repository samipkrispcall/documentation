# Multi-Number Handling – Decision Report

---

## 1. Search Contact by Number
- Contact search should be performed only using the primary number.
- If a call is made using a secondary number, it should first be mapped to its corresponding primary number, and the search should then be performed using that primary number.

---

## 2. Create Contact

### Case 1: Contact not found in CRM
- Create the contact  
- Add secondary numbers if available  

### Case 2: Contact found, no secondary numbers in KrispCall
- Skip creation

### Case 3: Contact found, secondary numbers in KrispCall but not in CRM
- Add all secondary numbers to the existing CRM contact

### Case 4: Contact found, secondary numbers exist in both KrispCall and CRM

#### Scenario 1: CRM supports unlimited numbers
- Append non-matching KrispCall secondary numbers  
  → *Risk: Total number count may exceed limits (e.g., more than 6)*

#### Scenario 2: CRM has fixed/custom number fields 
- Append non-matching KrispCall secondary numbers for remaining empty custom field slots
- Remaining phone number info will be provided using note event.

---

## 3. Update Contact

### 3.1 Existing Contact Data Should Include:
- Primary number  
- Secondary numbers  
- Primary email  
- Secondary emails  


### 3.2 Merge Handling

#### Case 1: CRM supports unlimited numbers
- Replace matched numbers and append non-matching KrispCall secondary numbers  
  → *Risk: Total number count may exceed limits (e.g., more than 6)*  

#### Case 2: CRM has fixed/custom number fields
- Replace matched numbers and append non-matching KrispCall secondary numbers for remainging empty custom field slots.
  → *Prevents data loss*
  note: remaining secodnary number details in note may not be feasible as it is triggered every time we update that contact. 

---

## 4. Webhook Contact Sync

- Enhance the `Contact` model to support multiple numbers and emails:

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

    secondary_numbers: Optional[List[str]] = None
    secondary_emails: Optional[List[str]] = None
```

## Current Implementation Requirements

### 1. Contact Search Logic
- Always perform contact search using the primary number.
- If a call comes through a secondary number, it must first be mapped to the corresponding primary number before performing the search.
- If the number does not exist in either the primary or secondary records, the search should be performed using the provided number.

### 2. Contact Data Structure Expectations
- Existing contact data must consistently include:
  - Primary number  
  - Secondary numbers  
  - Primary email  
  - Secondary emails  

### 3. Model Enhancement
- Update the `Contact` model to support multiple values by adding:
  - `secondary_numbers`  
  - `secondary_emails`

- Creation and update of contacts through webhook must also reflect changes to secondary numbers and secondary emails in KrispCall