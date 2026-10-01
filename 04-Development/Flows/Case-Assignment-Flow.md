# Case Assignment Flow

## 1. Overview

### 1.1 Purpose

The Case Assignment Flow automatically assigns a Salesforce Case to the appropriate support queue based on the Case's Service Category.

The objective is to ensure that newly created Cases are routed to the correct service team without requiring manual assignment by a Service Agent.

### 1.2 Business Requirement

**BR-005 — Automated Assignment**

Cases should be automatically assigned to the appropriate support team based on their service category.

### 1.3 Functional Requirements

The Flow supports:

* FR-003 Case Category
* FR-005 Automated Queue Assignment
* FR-006 Case Lifecycle
* FR-015 Security
* FR-021 Declarative Automation

### 1.4 User Stories

The Flow supports:

* US-008 — Automatically Assign Cases based on Service Category
* US-009 — Assign Technical Support Cases
* US-010 — Assign Billing & Payments Cases
* US-011 — Assign General Service Requests

---

# 2. Business Scenario

A customer creates a service Case.

The Case contains a Service Category such as:

* Technical Support
* Billing & Payments
* General Service Request

Instead of requiring an agent or administrator to manually identify the responsible team, Salesforce automatically assigns the Case to the corresponding queue.

Example:

```text
New Case
   │
   ▼
Service Category = Technical Support
   │
   ▼
Technical Support Queue
```

---

# 3. Assignment Matrix

| Service Category        | Queue                   |
| ----------------------- | ----------------------- |
| Technical Support       | Technical Support Queue |
| Billing & Payments      | Billing Support Queue   |
| General Service Request | General Support Queue   |

This mapping represents the initial portfolio design.

Additional categories can be added later.

---

# 4. Technical Approach

The solution will use a:

**Record-Triggered Flow**

### Why Flow?

Flow is appropriate because the requirement involves:

* Evaluating Case field values.
* Applying straightforward business rules.
* Assigning a Case owner.
* Avoiding unnecessary custom Apex code.

The requirement does not initially require complex calculations or advanced server-side processing.

Therefore, Flow is preferred over Apex for this use case.

---

# 5. Flow Type

### Flow Type

**Record-Triggered Flow**

### Object

**Case**

### Trigger

The Flow should run when a Case is created.

The Flow may also be configured to run when the Service Category changes if reassignment after categorization is required.

The initial implementation should keep the trigger conditions focused to avoid unnecessary Flow execution.

---

# 6. Flow Entry Conditions

The initial Flow should evaluate Cases where:

* The Case is being created.
* `Service_Category__c` is populated.

If the Flow is later expanded to support reassignment, the entry criteria can include relevant updates to `Service_Category__c`.

---

# 7. Flow Logic

The proposed logic is:

```text
                    Case Created
                         │
                         ▼
             Service Category populated?
                    /          \
                  No            Yes
                  │              │
                  ▼              ▼
              End Flow       Evaluate Category
                                  │
                    ┌─────────────┼─────────────┐
                    │             │             │
                    ▼             ▼             ▼
             Technical       Billing &      General
              Support         Payments       Service Request
                    │             │             │
                    ▼             ▼             ▼
              Technical       Billing       General
                Queue          Queue          Queue
                    │             │             │
                    └─────────────┼─────────────┘
                                  ▼
                              End Flow
```

---

# 8. Flow Elements

The initial Flow should contain the following elements:

| Order | Element                   | Type           | Purpose                      |
| ----- | ------------------------- | -------------- | ---------------------------- |
| 1     | Start                     | Start          | Trigger when Case is created |
| 2     | Evaluate Service Category | Decision       | Determine assignment queue   |
| 3     | Assign Technical Queue    | Assignment     | Set Case Owner               |
| 4     | Assign Billing Queue      | Assignment     | Set Case Owner               |
| 5     | Assign General Queue      | Assignment     | Set Case Owner               |
| 6     | Update Case               | Update Records | Persist the new owner        |
| 7     | End                       | End            | Complete processing          |

The exact element structure may be simplified if the Flow can directly update the Case through the chosen design.

---

# 9. Queue Configuration

Before the Flow is activated, the following queues must exist:

### Technical Support Queue

Purpose:

Handles Cases categorized as Technical Support.

### Billing Support Queue

Purpose:

Handles Cases categorized as Billing & Payments.

### General Support Queue

Purpose:

Handles Cases categorized as General Service Request.

Queue membership will be configured separately as part of security and Salesforce setup.

---

# 10. Owner Assignment

Salesforce Case ownership will be updated to the appropriate Queue.

Example:

```text
Case.OwnerId
      │
      ▼
Technical Support Queue
```

The Flow should assign the Queue's Salesforce record ID rather than hard-coding an invalid or environment-specific value.

Queue IDs are Salesforce-org-specific, so the implementation should retrieve the appropriate Queue record during Flow configuration.

---

# 11. Decision Logic

The Decision element will evaluate:

`$Record.Service_Category__c`

### Outcome 1 — Technical Support

Condition:

```text
Service_Category__c = "Technical Support"
```

Action:

Assign Case to **Technical Support Queue**.

---

### Outcome 2 — Billing & Payments

Condition:

```text
Service_Category__c = "Billing & Payments"
```

Action:

Assign Case to **Billing Support Queue**.

---

### Outcome 3 — General Service Request

Condition:

```text
Service_Category__c = "General Service Request"
```

Action:

Assign Case to **General Support Queue**.

---

### Default Outcome

If no supported category is found:

```text
No Assignment
     │
     ▼
Continue / End
```

The Case should not be assigned incorrectly simply because a new or unsupported category exists.

---

# 12. Recommended Flow Design

The initial implementation should use a simple design:

```text
START
  │
  ▼
DECISION
  │
  ├── Technical Support
  │       ↓
  │   Assignment
  │       ↓
  │   Update Case
  │
  ├── Billing & Payments
  │       ↓
  │   Assignment
  │       ↓
  │   Update Case
  │
  └── General Service Request
          ↓
      Assignment
          ↓
      Update Case
```

The Flow should remain easy to understand and maintain.

---

# 13. Before-Save vs After-Save Consideration

Case Owner assignment is a record update requirement.

The Flow implementation should be evaluated to determine whether the OwnerId can be set efficiently during the triggering transaction or whether an after-save Flow is required.

The implementation should prefer the simplest supported design that avoids unnecessary additional database operations.

The final choice will be validated in the Salesforce Flow Builder.

---

# 14. Recursion Consideration

The Flow must avoid repeatedly triggering itself because of its own Case update.

If the Flow performs an update to the Case, entry conditions should be designed carefully.

For example:

* Trigger on Case creation for the initial implementation.
* Avoid triggering solely because Owner changes.
* If category-change reassignment is added later, restrict the Flow to meaningful category changes.

---

# 15. Error Handling

Potential issues include:

* Queue does not exist.
* Queue is inactive or incorrectly configured.
* Service Category contains an unsupported value.
* Required Case data is missing.
* Flow encounters a runtime error.

The Flow should avoid silently assigning a Case to the wrong team.

Unexpected Flow errors should be investigated using Salesforce Flow error details and debug tools.

---

# 16. Security Considerations

The Flow must operate within the intended Salesforce security architecture.

Considerations include:

* Case access
* Queue access
* User permissions
* Service Category field access
* Owner field behavior

Service Agents should not gain additional administrative privileges merely because the Flow assigns Case ownership.

---

# 17. Test Scenarios

The Flow will be tested using positive, negative, and boundary scenarios.

### Test Case CA-001

**Scenario:** Technical Support Case

**Given:**

A new Case has Service Category = Technical Support.

**When:**

The Case is created.

**Then:**

The Case Owner should be Technical Support Queue.

---

### Test Case CA-002

**Scenario:** Billing Case

**Given:**

A new Case has Service Category = Billing & Payments.

**When:**

The Case is created.

**Then:**

The Case Owner should be Billing Support Queue.

---

### Test Case CA-003

**Scenario:** General Service Request

**Given:**

A new Case has Service Category = General Service Request.

**When:**

The Case is created.

**Then:**

The Case Owner should be General Support Queue.

---

### Test Case CA-004

**Scenario:** Missing Service Category

**Given:**

A new Case does not have a Service Category.

**When:**

The Case is created.

**Then:**

The Flow should not assign the Case to an incorrect queue.

---

### Test Case CA-005

**Scenario:** Unsupported Category

**Given:**

A Case contains a category that is not mapped by the Flow.

**When:**

The Flow executes.

**Then:**

The Case should follow the default outcome without incorrect queue assignment.

---

### Test Case CA-006

**Scenario:** Existing Case Update

**Given:**

An existing Case is updated without changing Service Category.

**When:**

The Case is saved.

**Then:**

The Flow should not unnecessarily reassign the Case.

---

# 18. Bulk Considerations

Although Flow is declarative, the design must consider bulk execution.

The Flow should:

* Avoid unnecessary Get Records operations.
* Avoid unnecessary Update Records operations.
* Avoid unnecessary repeated processing.
* Use appropriate entry criteria.

If a large number of Cases are created through an integration or data load, the Flow should behave predictably.

---

# 19. Debugging Approach

During development, Flow Builder's Debug functionality will be used to validate:

1. Case creation.
2. Service Category value.
3. Decision outcome.
4. Queue lookup.
5. Owner assignment.
6. Case update.
7. Final Case Owner.

Example:

```text
Input Case
    ↓
Service Category
    ↓
Decision Outcome
    ↓
Queue
    ↓
OwnerId
    ↓
Saved Case
```

---

# 20. Deployment Considerations

The Flow metadata will be stored in the Salesforce project under:

```text
force-app/
└── main/
    └── default/
        └── flows/
```

The Flow should be deployed only after:

* Queue configuration is available.
* Required Case fields exist.
* Flow tests pass.
* Security behavior is validated.

---

# 21. Naming Convention

The Flow should use a clear Salesforce metadata name.

Recommended:

**`Case_Assignment_Flow`**

Label:

**Case Assignment Flow**

The name should remain stable once the Flow becomes part of the project source.

---

# 22. Acceptance Criteria Traceability

| Acceptance Criteria | Flow Coverage                                   |
| ------------------- | ----------------------------------------------- |
| AC-008.1            | Assignment rules evaluated                      |
| AC-009.1            | Technical Support → Technical Support Queue     |
| AC-010.1            | Billing & Payments → Billing Support Queue      |
| AC-011.1            | General Service Request → General Support Queue |

---

# 23. Definition of Done

The Case Assignment Flow is complete when:

* Required Case field exists.
* Required queues exist.
* Flow is configured.
* Decision logic works.
* Each category routes correctly.
* Missing/unsupported categories do not route incorrectly.
* Existing Cases are not unnecessarily reassigned.
* Flow debugging succeeds.
* Functional test cases pass.
* Security behavior is validated.
* Flow metadata is saved in the project.
* Documentation is updated.

---

# 24. Implementation Status

| Item                   | Status      |
| ---------------------- | ----------- |
| Business Requirement   | Defined     |
| Functional Requirement | Defined     |
| User Stories           | Defined     |
| Acceptance Criteria    | Defined     |
| Flow Design            | Completed   |
| Salesforce Field       | Not Created |
| Queues                 | Not Created |
| Flow                   | Not Created |
| Flow Testing           | Not Started |
| Deployment             | Not Started |

---

# 25. Conclusion

The Case Assignment Flow provides a simple declarative solution for routing Cases to the appropriate support team based on Service Category.

The implementation deliberately uses Flow instead of Apex because the initial requirement is a straightforward field-based routing rule.

The Flow will be built only after the required Case field and queues have been configured and validated.
