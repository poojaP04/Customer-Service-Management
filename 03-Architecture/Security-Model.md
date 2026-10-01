# Security Model

## 1. Document Overview

### 1.1 Purpose

This document defines the security model for the Customer Service & Case Management Platform.

The security design ensures that users can access the customer and Case information required for their responsibilities while restricting unauthorized access to data, fields, records, and administrative capabilities.

The design uses Salesforce's standard security framework and follows a least-privilege approach.

### 1.2 Scope

The security model covers:

* User access
* Profiles
* Permission Sets
* Object-level security
* Field-level security
* Record-level security
* Role hierarchy
* Organization-wide defaults
* Queue access
* Case access
* Auditability
* Administrative access

---

# 2. Security Principles

The implementation will follow these principles:

1. Grant users only the access required for their responsibilities.
2. Prefer Permission Sets for additional access instead of creating unnecessary profiles.
3. Restrict sensitive fields through Field-Level Security.
4. Use record-level security to control access to Cases.
5. Use queues to support team-based Case ownership.
6. Avoid granting Modify All Data unless administrative access is genuinely required.
7. Keep administrative permissions separate from operational permissions.
8. Validate security through positive and negative test cases.
9. Review security whenever new functionality or fields are introduced.

---

# 3. Salesforce Security Layers

The solution will use multiple Salesforce security layers.

```text id="s5m7kq"
                    Salesforce Security
                           │
          ┌────────────────┼────────────────┐
          │                │                │
          ▼                ▼                ▼
      Org Access      Object Access    Record Access
          │                │                │
          ▼                ▼                ▼
       Login/IP        CRUD/FLS       OWD/Sharing
          │                │                │
          └────────────────┼────────────────┘
                           │
                           ▼
                    User Permissions
                           │
                           ▼
                    Permission Sets
```

Security will be implemented as layered controls rather than relying on a single mechanism.

---

# 4. User Personas

The initial solution will support the following user personas:

| Persona                  | Primary Responsibility                                          |
| ------------------------ | --------------------------------------------------------------- |
| Service Agent            | Handles assigned Cases and performs service activities          |
| Service Manager          | Monitors Cases, escalations, SLA performance, and team workload |
| Salesforce Administrator | Configures and maintains the Salesforce solution                |

These personas represent functional responsibilities within the portfolio solution.

---

# 5. Access Model

## 5.1 Service Agent

A Service Agent should be able to:

* View relevant customer information
* Create Cases
* View Cases they are permitted to access
* Update Cases they own or are assigned to
* Update Case status
* Update priority where permitted
* Update service category where permitted
* Add resolution information
* Work with assigned queues
* View relevant reports

A Service Agent should not have unrestricted administrative access.

---

## 5.2 Service Manager

A Service Manager should be able to:

* View Cases across the relevant service teams
* Monitor Case workload
* Monitor SLA status
* Review escalated Cases
* Reassign Cases where required
* Review reports and dashboards
* Monitor team performance

A Service Manager should not automatically receive Salesforce configuration permissions.

---

## 5.3 Salesforce Administrator

The Salesforce Administrator should be able to:

* Configure Salesforce
* Manage users and permissions
* Configure objects and fields
* Manage automation
* Manage queues
* Configure reports and dashboards
* Deploy metadata
* Troubleshoot application issues

Administrative access should be restricted to the appropriate administrative user.

---

# 6. Profiles

Profiles will provide the baseline access required for each type of user.

The implementation will avoid creating unnecessary custom profiles.

Where the Salesforce environment supports suitable standard profiles, they may be used as the starting point.

Additional access should preferably be provided using Permission Sets.

### Security Design

```text id="1u2zpc"
Profile
   │
   │ Baseline Access
   ▼
Permission Set
   │
   │ Additional Required Access
   ▼
User
```

---

# 7. Permission Sets

Permission Sets will be used to provide additional access without unnecessarily modifying baseline profiles.

Potential Permission Sets include:

| Permission Set      | Purpose                                            |
| ------------------- | -------------------------------------------------- |
| Case Agent Access   | Additional Case management capabilities            |
| Case Manager Access | Manager-level Case visibility and management       |
| Integration Access  | Permissions required for integration functionality |
| Reporting Access    | Additional reporting capabilities where required   |

Permission Set names will be finalized during implementation.

---

# 8. Object-Level Security

Object-level security determines whether a user can access a Salesforce object and what actions they can perform.

The primary objects are:

* Account
* Contact
* Case

### Example Access Model

| Object  | Service Agent    | Service Manager | Administrator |
| ------- | ---------------- | --------------- | ------------- |
| Account | Read             | Read/Edit       | Full          |
| Contact | Read             | Read/Edit       | Full          |
| Case    | Create/Read/Edit | Read/Edit       | Full          |

The exact permissions will be validated during Salesforce configuration.

---

# 9. CRUD Permissions

CRUD represents:

* Create
* Read
* Update
* Delete

The solution will apply the minimum CRUD access necessary.

### Service Agent

A Service Agent generally requires:

* Account: Read
* Contact: Read
* Case: Create, Read, Edit

Delete access should not be granted unless there is a demonstrated business requirement.

### Service Manager

A Service Manager generally requires:

* Account: Read/Edit where required
* Contact: Read/Edit where required
* Case: Create, Read, Edit

### Administrator

The Salesforce Administrator requires appropriate configuration and administrative access.

---

# 10. Field-Level Security

Field-Level Security will protect fields that should not be editable by every user.

Potential fields requiring controlled access include:

* SLA Target
* SLA Status
* Escalation Reason
* Resolution Date
* Escalated
* Integration-related fields

For example, a Service Agent may be able to view an SLA Target but should not necessarily be able to manually modify the calculated value.

### Example

```text id="3v4qga"
SLA Target
    │
    ├── Service Agent → Read
    │
    ├── Service Manager → Read
    │
    └── Administrator → Read/Edit
```

Final field-level permissions will be validated during implementation.

---

# 11. Record-Level Security

Record-level security determines which individual records a user can access.

The solution will use Salesforce mechanisms such as:

* Organization-Wide Defaults
* Role Hierarchy
* Sharing Rules where required
* Queue ownership
* Manual or programmatic sharing only when necessary

The objective is to avoid exposing all Cases to every operational user by default.

---

# 12. Organization-Wide Defaults

Organization-Wide Defaults (OWD) will establish the baseline record access.

The initial design will consider a restrictive Case access model so that Case visibility can be expanded through appropriate ownership and sharing mechanisms.

### Initial Design Consideration

| Object  | Initial Access Approach     |
| ------- | --------------------------- |
| Account | Controlled baseline access  |
| Contact | Controlled baseline access  |
| Case    | Private/restricted baseline |

The final OWD settings will be confirmed after validating the required Service Agent and Service Manager access model.

---

# 13. Role Hierarchy

The role hierarchy may be structured as:

```text id="5a6k3w"
Salesforce Administrator
          │
          ▼
   Service Manager
          │
          ▼
    Service Agent
```

The role hierarchy can provide managers with access to records owned by users below them where appropriate.

The role hierarchy should not be used as the only security mechanism.

---

# 14. Queue Security

Queues will be used for team-based Case ownership.

Initial queues:

* Technical Support Queue
* Billing Support Queue
* General Support Queue

Queue membership determines which users can work with Cases assigned to the queue.

### Example

```text id="q6x2r9"
Technical Support Queue
        │
        ├── Agent A
        ├── Agent B
        └── Agent C
```

A Case assigned to the queue can then be accepted or assigned to an individual agent according to the operational process.

---

# 15. Case Access Model

The intended Case access flow is:

```text id="z4w8hc"
Case Created
     │
     ▼
Automated Assignment
     │
     ▼
Queue
     │
     ▼
Authorized Service Agent
     │
     ▼
Case Processing
     │
     ▼
Resolution / Closure
```

Users should only be able to perform actions permitted by their object, field, and record-level permissions.

---

# 16. Case Field Security

Important Case fields will have controlled access.

| Field              | Agent     | Manager | Administrator |
| ------------------ | --------- | ------- | ------------- |
| Subject            | Edit      | Edit    | Edit          |
| Description        | Edit      | Edit    | Edit          |
| Priority           | Edit      | Edit    | Edit          |
| Service Category   | Edit      | Edit    | Edit          |
| SLA Target         | Read      | Read    | Edit          |
| SLA Status         | Read      | Read    | Edit          |
| Escalation Reason  | Edit      | Edit    | Edit          |
| Resolution Summary | Edit      | Edit    | Edit          |
| Resolution Date    | Read/Edit | Edit    | Edit          |
| Escalated          | Read/Edit | Edit    | Edit          |

These permissions are an initial design and will be tested against the actual automation behavior.

---

# 17. Automation and Security

Automation must respect Salesforce security requirements.

The implementation will consider:

* Record access
* Field access
* User permissions
* Queue ownership
* Sharing behavior
* Flow execution context
* Apex execution context

Apex code will be designed to avoid unintentionally exposing or modifying records beyond the intended business scope.

Where appropriate, Apex will explicitly consider:

* Sharing behavior
* CRUD permissions
* Field-Level Security

---

# 18. Integration Security

External integrations will not use hard-coded credentials.

The architecture will use Salesforce Named Credentials for external authentication.

### Integration Security Flow

```text id="9n8v0x"
External System
      │
      │ HTTPS / JSON
      ▼
Salesforce Integration Endpoint
      │
      ▼
Named Credential
      │
      ▼
Authenticated Request
      │
      ▼
Validation
      │
      ▼
Case Creation / Update
```

Integration-specific permissions will be isolated from normal Service Agent permissions.

---

# 19. Authentication Considerations

Salesforce authentication policies will be used where applicable.

Potential controls include:

* Strong password requirements
* Session security
* Login restrictions
* Multi-factor authentication
* Profile-based login hours where required
* IP restrictions where appropriate

The exact controls depend on the Salesforce environment and available configuration.

---

# 20. Auditability

The solution will use Salesforce audit capabilities to support traceability.

Important audit information includes:

* Created By
* Created Date
* Last Modified By
* Last Modified Date
* Field History Tracking

Field History Tracking should be considered for:

* Case Status
* Case Priority
* Case Owner
* Service Category
* SLA Status

This allows administrators and managers to understand important Case changes.

---

# 21. Data Protection

The implementation should avoid storing unnecessary sensitive information.

The solution will:

* Collect only required customer information.
* Restrict sensitive fields.
* Avoid unnecessary duplication of customer data.
* Use Salesforce security controls for access.
* Avoid exposing credentials in Apex or configuration files.
* Use Named Credentials for external authentication.

---

# 22. Security Testing

Security testing will include both positive and negative scenarios.

### Positive Tests

Examples:

* Service Agent can create a Case.
* Service Agent can update an assigned Case.
* Service Manager can view relevant team Cases.
* Administrator can configure Case fields.
* Authorized users can access queue-owned Cases.

### Negative Tests

Examples:

* Service Agent cannot access administrative configuration.
* Service Agent cannot modify protected SLA fields where access is read-only.
* Unauthorized users cannot access restricted Cases.
* Users cannot access integration credentials.
* Users cannot delete Cases without the required permission.

---

# 23. Security Test Matrix

| Test ID | Scenario                                           | Expected Result |
| ------- | -------------------------------------------------- | --------------- |
| SEC-001 | Agent creates Case                                 | Allowed         |
| SEC-002 | Agent updates assigned Case                        | Allowed         |
| SEC-003 | Agent accesses unauthorized Case                   | Restricted      |
| SEC-004 | Agent edits protected SLA field                    | Restricted      |
| SEC-005 | Manager views team Cases                           | Allowed         |
| SEC-006 | Agent accesses Setup configuration                 | Restricted      |
| SEC-007 | Unauthorized user accesses integration credentials | Restricted      |
| SEC-008 | Authorized queue member accesses queue Case        | Allowed         |
| SEC-009 | User without delete permission attempts deletion   | Restricted      |
| SEC-010 | Administrator configures Case security             | Allowed         |

---

# 24. Security and Reporting

Reports and dashboards must also respect Salesforce record-level security.

A user viewing a report should only see records they are authorized to access unless a specific report/dashboard configuration provides broader visibility for an authorized role.

Manager dashboards should therefore be configured carefully to avoid unintended exposure of Case information.

---

# 25. Security and LWC

The custom Case Workspace LWC must respect Salesforce security.

The component should:

* Use Salesforce-supported data access mechanisms.
* Respect object and field permissions.
* Avoid exposing fields that the current user cannot access.
* Avoid bypassing Salesforce sharing controls.
* Display only information relevant to the current user's access.

The LWC should not be used as a mechanism to bypass Salesforce security.

---

# 26. Security and Apex

Apex will be implemented with security in mind.

The development approach should include:

* Appropriate sharing declarations.
* Bulk-safe implementation.
* Validation of user access where required.
* Avoidance of unnecessary system-level access.
* Appropriate CRUD/FLS consideration.
* Controlled data exposure.

Apex should only be used when declarative Salesforce capabilities are insufficient for the requirement.

---

# 27. Security Change Management

Security changes should be treated as controlled application changes.

Any new:

* Object
* Field
* Permission Set
* Queue
* Sharing Rule
* Profile permission
* Apex capability
* Integration permission

should be reviewed against the security model before deployment.

---

# 28. Security Traceability

| Requirement                 | Security Control                                    |
| --------------------------- | --------------------------------------------------- |
| BR-002 Customer Information | Account/Contact permissions and FLS                 |
| BR-005 Automated Assignment | Queue ownership and queue membership                |
| BR-006 SLA Monitoring       | SLA field security                                  |
| BR-007 Escalation           | Case access and controlled escalation fields        |
| BR-011 Security             | Profiles, Permission Sets, CRUD, FLS, sharing       |
| BR-012 Auditability         | Audit fields and Field History Tracking             |
| FR-015 Security             | Layered Salesforce security                         |
| FR-022 Audit                | Salesforce audit capabilities                       |
| FR-018 LWC                  | Security-aware data access                          |
| FR-020 Apex                 | Secure Apex implementation                          |
| FR-019 Integration          | Named Credentials and controlled integration access |

---

# 29. Implementation Sequence

Security configuration will be implemented in the following sequence:

1. Identify user personas.
2. Review baseline profiles.
3. Define Permission Sets.
4. Configure object permissions.
5. Configure Field-Level Security.
6. Define Case record access.
7. Configure Organization-Wide Defaults.
8. Configure role hierarchy.
9. Create and configure queues.
10. Configure sharing mechanisms where required.
11. Configure audit/history tracking.
12. Review Flow security behavior.
13. Review Apex security.
14. Review LWC security.
15. Validate integration security.
16. Execute security test cases.
17. Document final security configuration.

---

# 30. Open Security Decisions

The following items will be finalized during Salesforce configuration:

* Final profiles to be used
* Permission Set names
* Case OWD
* Account and Contact OWD
* Final role hierarchy
* Sharing Rules
* Queue membership
* Field-Level Security for custom fields
* Manager visibility requirements
* Integration user permissions
* Exact authentication policies available in the target org

---

# 31. Conclusion

The security model uses Salesforce's standard layered security architecture to control access at the organization, object, field, record, and user levels.

The solution follows a least-privilege approach and uses Profiles, Permission Sets, OWD, Role Hierarchy, Sharing, Queues, Field-Level Security, and audit capabilities according to their respective responsibilities.

Security will be validated during development and testing rather than treated as a final configuration step.
