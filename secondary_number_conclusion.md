# Multi-Number Handling – Decision Report

---

## 1. Create Contact

### Case 1: Contact not found in CRM
- Create the contact  
- Add secondary numbers if available  

---

### Case 2: Contact found, no secondary numbers in KrispCall
- Skip creation  

---

### Case 3: Contact found, secondary numbers in KrispCall but not in CRM
- Add all secondary numbers to the existing CRM contact  

---

### Case 4: Contact found, secondary numbers exist in both KrispCall and CRM

#### Scenario 1: CRM supports unlimited numbers

**Options:**
- Replace all CRM secondary numbers with KrispCall numbers  
  → *Risk: Data loss in CRM*  

- Append non-matching KrispCall secondary numbers  
  → *Risk: Total number count may exceed limits (e.g., more than 6)*  

---

#### Scenario 2: CRM has fixed/custom number fields

**Options:**
- Replace all CRM secondary numbers with KrispCall numbers  
  → *Risk: Data loss*  

- Create a new contact  
  → *Prevents data loss*

  → *Risk: Duplicate Contact Existance*

---

### Case 5: Contact found, KrispCall has secondary numbers, CRM secondary numbers unknown

**Options:**
- Replace/create CRM secondary numbers with KrispCall numbers  
  → *Risk: Data loss*  

- Create a new contact  
  → *Prevents data loss*  

---

## 2. Update Contact

### 2.1 Existing Contact Data Should Include:
- Primary number  
- Secondary numbers  
- Primary email  
- Secondary emails  

---

### 2.2 Merge Handling

#### Case 1: CRM supports unlimited numbers

**Options:**
- Replace all CRM secondary numbers with KrispCall numbers  
  → *Risk: Data loss*  

- Replace matched numbers and append non-matching KrispCall secondary numbers  
  → *Risk: Total number count may exceed limits (e.g., more than 6)*  

---

#### Case 2: CRM has fixed/custom number fields

**Options:**
- Replace all CRM secondary numbers with KrispCall numbers  
  → *Risk: Data loss*  

- Create a new contact  
  → *Prevents data loss*  

---

## **3. Bulk Contact Sync**

- No changes required.  
- Secondary numbers and secondary emails will **not be handled for now**.

---

## **4. Webhook Contact Sync**

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