# Data Model

## 1. Document Overview

### 1.1 Purpose

This document defines the logical and Salesforce data model for the Customer Service & Case Management Platform.

The data model is designed to support centralized case management, customer association, categorization, prioritization, queue assignment, SLA tracking, escalation, resolution, reporting, security, and future integration requirements.

The model follows a standard Salesforce-first approach by using standard objects where appropriate and introducing custom fields or objects only when required by the business requirements.

### 1.2 Scope

The data model covers:

* Customer and contact information
* Service case management
* Case categorization
* Case priority and status
* Queue assignment
* SLA tracking
* Escalation
* Resolution and closure
* Audit information
* Reporting requirements
* Future integration considerations

---

# 2. Data Model Principles

The following principles will guide the implementation:

1. Prefer standard Salesforce objects and fields where they satisfy the requirement.
2. Create custom fields only when standard Salesforce functionality is insufficient.
3. Avoid unnecessary custom objects.
4. Maintain clear relationships between customer, contact, and service cases.
5. Design the model for reporting and operational visibility.
6. Support automation without creating unnecessary data duplication.
7. Consider bulk processing and scalability for high-volume case creation.
8. Apply appropriate object-level, field-level, and record-level security.
9. Keep the model extensible for future integrations.

---

# 3. Core Data Model

The core model will use the following Salesforce objects:

| Object  | Type     | Purpose                                                    |
| ------- | -------- | ---------------------------------------------------------- |
| Account | Standard | Represents the customer organization or customer entity    |
| Contact | Standard | Represents an individual associated with an Account        |
| Case    | Standard | Represents a customer service request, complaint, or issue |
| User    | Standard | Represents an individual Salesforce user/agent             |
| Queue   | Standard | Represents a group responsible for handling Cases          |

### High-Level Relationship

```text
Account
   |
   | 1 : Many
   |
 Contact
   |
   | 1 : Many
   |
 Case
   |
   +---- Owner → User / Queue
   |
   +---- Service Category
   |
   +---- Priority
   |
   +---- SLA Information
   |
   +---- Resolution Information
```

---

# 4. Account

## 4.1 Purpose

The Account represents the customer or organization associated with one or more service cases.

The standard Salesforce Account object will be used.

## 4.2 Important Standard Fields

| Field           | Type     | Purpose                                       |
| --------------- | -------- | --------------------------------------------- |
| Account Name    | Text     | Customer or organization name                 |
| Account Number  | Text     | Customer reference where applicable           |
| Phone           | Phone    | Primary customer contact number               |
| Email           | Email    | Customer communication email where applicable |
| Billing Address | Address  | Customer address                              |
| Type            | Picklist | Customer classification where required        |

Additional standard Account fields may be enabled only if required by the implementation.

---

# 5. Contact

## 5.1 Purpose

The Contact represents an individual customer or person associated with an Account.

The standard Salesforce Contact object will be used.

## 5.2 Important Standard Fields

| Field      | Type   | Purpose                     |
| ---------- | ------ | --------------------------- |
| First Name | Text   | Customer first name         |
| Last Name  | Text   | Customer last name          |
| Email      | Email  | Customer email              |
| Phone      | Phone  | Customer phone number       |
| Account    | Lookup | Associated customer/account |

Contacts can be associated with Cases so that service agents can identify the individual who raised or is affected by the request.

---

# 6. Case

## 6.1 Purpose

The standard Salesforce Case object will be the central object for the platform.

Each Case represents a customer service request, complaint, issue, or support interaction.

## 6.2 Standard Case Fields

The implementation will use standard Salesforce Case fields wherever possible.

| Field              | Type        | Purpose                          |
| ------------------ | ----------- | -------------------------------- |
| Case Number        | Auto Number | Unique Case identifier           |
| Subject            | Text        | Short description of the request |
| Description        | Long Text   | Detailed description             |
| Status             | Picklist    | Current Case lifecycle status    |
| Priority           | Picklist    | Case priority                    |
| Origin             | Picklist    | Source of Case                   |
| Account Name       | Lookup      | Associated customer              |
| Contact Name       | Lookup      | Associated contact               |
| Owner              | User/Queue  | Current Case owner               |
| Created Date       | Date/Time   | Case creation timestamp          |
| Last Modified Date | Date/Time   | Last modification timestamp      |

---

# 7. Custom Case Fields

The following custom fields are proposed to support business requirements that are not fully covered by standard Case fields.

| Field              | API Name                | Type           | Purpose                                       |
| ------------------ | ----------------------- | -------------- | --------------------------------------------- |
| Service Category   | `Service_Category__c`   | Picklist       | Categorizes the service request               |
| SLA Target         | `SLA_Target__c`         | Date/Time      | Stores the calculated SLA target              |
| SLA Status         | `SLA_Status__c`         | Picklist       | Identifies current SLA condition              |
| Escalation Reason  | `Escalation_Reason__c`  | Picklist/Text  | Records reason for escalation                 |
| Resolution Summary | `Resolution_Summary__c` | Long Text Area | Stores resolution details                     |
| Resolution Date    | `Resolution_Date__c`    | Date/Time      | Records when the Case was resolved            |
| Escalated          | `Escalated__c`          | Checkbox       | Indicates whether the Case has been escalated |

These fields will be validated during implementation and may be adjusted if standard Salesforce functionality can satisfy the requirement.

---

# 8. Service Category

## 8.1 Initial Categories

The initial portfolio implementation will use the following categories:

| Service Category        | Description                                   | Assignment Queue        |
| ----------------------- | --------------------------------------------- | ----------------------- |
| Technical Support       | Technical or system-related requests          | Technical Support Queue |
| Billing & Payments      | Billing, payment, or invoice-related requests | Billing Support Queue   |
| General Service Request | General customer service requests             | General Support Queue   |

The category will be implemented as a Case picklist field:

`Service_Category__c`

## 8.2 Future Categories

The model should allow additional service categories to be introduced without requiring major changes to the Case data model.

---

# 9. Case Priority

The standard Case `Priority` field will be used.

Initial values:

| Priority | Description                                                |
| -------- | ---------------------------------------------------------- |
| High     | Request requiring urgent attention                         |
| Medium   | Request requiring normal priority handling                 |
| Low      | Request that can be handled within a longer service window |

Priority may be assigned manually, through Flow automation, or through future business rules.

---

# 10. Case Status

The Case lifecycle will use the following statuses:

| Status           | Purpose                                                        |
| ---------------- | -------------------------------------------------------------- |
| New              | Case has been created but not yet assigned                     |
| Assigned         | Case has been assigned to an appropriate queue or agent        |
| In Progress      | Agent is actively working on the Case                          |
| Pending Customer | Additional information or action is required from the customer |
| Escalated        | Case requires higher-level attention                           |
| Resolved         | Service issue has been resolved                                |
| Closed           | Case has completed the final closure process                   |

The exact implementation will be validated against Salesforce Case status configuration during development.

---

# 11. Case Ownership

Case ownership will use Salesforce's standard Case Owner functionality.

A Case can be owned by:

* An individual Salesforce User
* A Salesforce Queue

### Initial Queues

| Queue                   | Responsibility           |
| ----------------------- | ------------------------ |
| Technical Support Queue | Technical Support Cases  |
| Billing Support Queue   | Billing & Payments Cases |
| General Support Queue   | General Service Requests |

The initial assignment logic will be:

```text
Service Category
       |
       +-- Technical Support
       |       → Technical Support Queue
       |
       +-- Billing & Payments
       |       → Billing Support Queue
       |
       +-- General Service Request
               → General Support Queue
```

---

# 12. SLA Data

SLA information will be stored against the Case.

## 12.1 Initial SLA Design

The initial portfolio design uses:

| Priority | Initial SLA Target |
| -------- | ------------------ |
| High     | 4 business hours   |
| Medium   | 1 business day     |
| Low      | 3 business days    |

These are initial design values for the portfolio implementation and can be changed through configuration.

## 12.2 SLA Fields

### SLA Target

`SLA_Target__c`

Stores the calculated target date/time.

### SLA Status

`SLA_Status__c`

Initial values:

* Not Started
* On Track
* At Risk
* Breached
* Met

### SLA Calculation

The target should be calculated based on:

```text
Case Priority
       +
Case Created Date/Time
       +
Applicable SLA Duration
       =
SLA Target
```

Business-hour calculations should be considered during implementation rather than simply adding calendar hours.

---

# 13. Escalation Data

Escalation is associated with the Case rather than being represented as a separate custom object in the initial design.

The Case will contain:

* Escalated indicator
* Escalation reason
* Current Case owner
* SLA status
* Priority
* Case status

This allows escalation information to remain directly available to service agents and reporting users.

Future implementations may introduce a separate escalation history object if detailed multi-level escalation tracking becomes a requirement.

---

# 14. Resolution Data

The following Case fields will support resolution:

| Field              | Purpose                               |
| ------------------ | ------------------------------------- |
| Resolution Summary | Describes how the issue was resolved  |
| Resolution Date    | Records resolution date/time          |
| Status             | Indicates Resolved or Closed          |
| Escalated          | Indicates whether escalation occurred |

Before a Case can be closed, required resolution information should be validated.

---

# 15. Audit and History

Salesforce standard audit fields will be used wherever possible.

Important standard audit information includes:

* Created By
* Created Date
* Last Modified By
* Last Modified Date

Field History Tracking may be enabled for important Case fields such as:

* Status
* Priority
* Owner
* Service Category
* SLA Status

This supports operational auditability and troubleshooting.

---

# 16. Relationships

## 16.1 Account → Contact

One Account can have multiple Contacts.

```text
Account
   |
   +-- Contact
   +-- Contact
   +-- Contact
```

## 16.2 Account → Case

One Account can have multiple Cases.

```text
Account
   |
   +-- Case
   +-- Case
   +-- Case
```

## 16.3 Contact → Case

A Contact can be associated with multiple Cases.

```text
Contact
   |
   +-- Case
   +-- Case
```

## 16.4 Case → Owner

A Case has one current owner, which can be either:

* User
* Queue

---

# 17. Conceptual Data Model

```text
                    ┌──────────────────┐
                    │     Account      │
                    │------------------│
                    │ Account Name     │
                    │ Phone            │
                    │ Email            │
                    └────────┬─────────┘
                             │
                 ┌───────────┴───────────┐
                 │                       │
                 ▼                       ▼
        ┌─────────────────┐     ┌──────────────────┐
        │     Contact     │     │       Case       │
        │-----------------│     │------------------│
        │ First Name      │     │ Case Number      │
        │ Last Name       │     │ Subject          │
        │ Email           │     │ Description      │
        │ Phone           │     │ Status           │
        └────────┬────────┘     │ Priority         │
                 │              │ Service Category │
                 │              │ SLA Target       │
                 │              │ SLA Status       │
                 └─────────────►│ Resolution       │
                                │ Escalation       │
                                │ Owner            │
                                └────────┬─────────┘
                                         │
                              ┌──────────┴──────────┐
                              ▼                     ▼
                       ┌──────────────┐      ┌──────────────┐
                       │    Queue     │      │     User     │
                       │--------------│      │--------------│
                       │ Support Team │      │ Service Agent│
                       └──────────────┘      └──────────────┘
```

---

# 18. Reporting Considerations

The data model must support reporting on:

* Total Cases
* Open Cases
* Cases by Status
* Cases by Priority
* Cases by Service Category
* Cases by Queue
* Cases by Agent
* SLA At Risk Cases
* SLA Breached Cases
* Resolved Cases
* Average Resolution Time
* Escalated Cases
* Case volume trends

The model therefore keeps operational attributes directly available on the Case wherever practical.

---

# 19. Security Considerations

The data model will support Salesforce security mechanisms including:

* Object-level permissions
* Field-level security
* Permission Sets
* Role hierarchy
* Sharing settings
* Queue membership
* Record-level access

Sensitive customer information should only be accessible to users who require it for their responsibilities.

Security configuration will be documented separately as part of the security design and implementation.

---

# 20. Integration Considerations

External systems may send service requests to Salesforce through a REST-based integration.

Expected flow:

```text
External System
      |
      | JSON
      ▼
Salesforce REST Interface
      |
      ▼
Integration / Apex Processing
      |
      ▼
Validation & Mapping
      |
      ▼
Case
```

External data should be mapped to existing Account, Contact, and Case fields wherever possible.

Integration-specific fields will only be introduced when a real requirement exists.

---

# 21. Scalability Considerations

The Case object is expected to handle potentially high transaction volumes.

The implementation should therefore:

* Use standard Salesforce objects where possible.
* Avoid unnecessary data duplication.
* Keep automation bulk-safe.
* Avoid SOQL/DML inside loops.
* Use bulkified Apex where Apex is required.
* Design Flows to handle bulk operations appropriately.
* Avoid unnecessary synchronous processing.
* Consider asynchronous processing for suitable high-volume operations.

---

# 22. Data Model to Automation Mapping

| Data Element           | Primary Automation                    |
| ---------------------- | ------------------------------------- |
| Service Category       | Record-Triggered Flow                 |
| Priority               | Flow / Business Rules                 |
| Queue Assignment       | Record-Triggered Flow                 |
| SLA Target             | Flow / Apex where required            |
| SLA Status             | Flow / Scheduled Automation           |
| Escalation             | Flow / Apex where complexity requires |
| Resolution Validation  | Flow                                  |
| Case Closure           | Flow / Validation Rule                |
| Complex Processing     | Apex                                  |
| External Case Creation | REST + Apex                           |

The implementation will follow a declarative-first approach and use Apex only where it provides clear value.

---

# 23. Data Model Validation

Before development begins, the following items will be validated:

* Standard Case fields required
* Custom Case fields required
* Final service categories
* Priority values
* Case status values
* SLA targets
* SLA business-hour requirements
* Escalation rules
* Required Account and Contact information
* Security requirements
* Integration field mapping
* Reporting requirements

---

# 24. Traceability

| Business Requirement               | Data Model Component                 |
| ---------------------------------- | ------------------------------------ |
| BR-001 Centralized Case Management | Case                                 |
| BR-002 Customer Information        | Account, Contact                     |
| BR-003 Case Categorization         | Service Category                     |
| BR-004 Case Prioritization         | Priority                             |
| BR-005 Automated Assignment        | Owner, Queue                         |
| BR-006 SLA Monitoring              | SLA Target, SLA Status               |
| BR-007 Escalation                  | Escalation fields, Case Status       |
| BR-008 Case Validation             | Case fields and validation           |
| BR-009 Case Lifecycle              | Case Status                          |
| BR-010 Reporting                   | Case operational fields              |
| BR-011 Security                    | Object/Field/Record security         |
| BR-012 Auditability                | Standard Audit Fields, Field History |

---

# 25. Implementation Sequence

The data model will be implemented in the following sequence:

1. Review standard Account fields.
2. Review standard Contact fields.
3. Configure Case standard fields.
4. Create required custom Case fields.
5. Configure Service Category picklist.
6. Configure Case Status values.
7. Configure Case Priority values.
8. Create support queues.
9. Configure SLA-related fields.
10. Configure escalation and resolution fields.
11. Configure field history tracking.
12. Validate relationships.
13. Validate security requirements.
14. Validate the model against functional requirements.
15. Begin automation development.

---

# 26. Open Data Model Decisions

The following items remain configurable and will be finalized during implementation:

* Final Case categories
* Final Case status values
* SLA business hours
* SLA target calculations
* Escalation thresholds
* Required customer fields
* Additional reporting fields
* Integration-specific fields
* Whether detailed escalation history requires a separate object
* Whether SLA configuration should eventually be moved to Custom Metadata

---

## 27. Conclusion

The proposed data model uses Salesforce standard objects as the foundation and introduces a limited number of custom Case fields to support service categorization, SLA management, escalation, resolution, and reporting.

The model is designed to remain simple enough for maintainability while supporting automation, security, integration, reporting, and future scalability.

The next implementation stage after data-model approval is the **Security Model**, followed by Salesforce configuration and development.
