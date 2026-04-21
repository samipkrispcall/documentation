# Multi-Number Handling – Decision Report

## **1. Create Contact**

- Use **both primary and secondary numbers** for contact searching.

### **1.1 If no match is found:**
- Create a new contact (including all secondary phone numbers and secondary emails).

### **1.2 If a match is found:**
- How to handle if the number is found in the primary field of the CRM?
    - *(To be discussed)*
- How to handle if the number is found in a secondary/custom field of the CRM?
    - *(To be discussed)*

---

## **2. Update Contact**

### **2.1 Existing contact data should include:**
- Primary number  
- Secondary numbers  
- Primary email  
- Secondary emails  

### **2.2 Merge Handling:**
- Decide which numbers/emails to keep and which to remove from the combined data (KrispCall + CRM).  
  - *(To be discussed)*

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