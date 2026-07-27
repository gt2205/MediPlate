# MediPlate Design Document

> This document describes the proposed system architecture and research-oriented design decisions behind MediPlate. Nothing described here has been implemented yet. The purpose of this document is to define a framework that can later be implemented and experimentally evaluated. See [README.md](README.md) for the project overview, motivation, and current status.

---

# Design Principles

The proposed architecture of MediPlate is guided by the following principles:

* **Doctor-first:** The platform is designed to integrate into existing clinical workflows rather than replacing them.
* **Structured over free-form:** Patient information and disease data are intended to remain structured throughout the system.
* **Grounded AI:** Every recommendation would be generated using curated disease mappings instead of relying solely on an LLM's pretrained knowledge.
* **Modular AI:** The meal generation component is designed to be isolated so it can later be replaced by a deterministic recommendation engine.
* **Simple patient experience:** Patients should require no installation or account creation.
* **Evaluation before deployment:** No component that touches real patient dietary guidance should ship without the evaluation methodology in README.md having been run and reviewed first.

---

# Proposed Architecture

```text
                    +----------------------+
                    |    Doctor Portal     |
                    |    (React Frontend)  |
                    +----------+-----------+
                               |
                               | HTTPS
                               |
                    +----------v-----------+
                    |      FastAPI API     |
                    | Authentication       |
                    | Patient Management   |
                    | Diet Generation      |
                    +-----+----------+-----+
                          |          |
              +-----------+          +----------------+
              |                                       |
    +---------v---------+               +-------------v-------------+
    | Firebase Database  |               | Disease Knowledge Base    |
    | Patients           |               | JSON Mappings             |
    | Doctors            |               | Clinical Guidelines       |
    +---------+----------+               +-------------+-------------+
              |                                       |
              +-------------------+-------------------+
                                  |
                        +---------v----------+
                        | Diet Generation    |
                        | Service (LLM)      |
                        +---------+----------+
                                  |
                  +---------------+----------------+
                  |                                |
       +----------v----------+          +----------v-----------+
       | PDF Generation      |          | Patient Assistant API |
       | QR Code             |          | Chat & Vision         |
       +---------------------+          +-----------------------+
```

The FastAPI backend is proposed as the orchestration layer, coordinating requests between the frontend, database, disease knowledge base, AI service, and PDF generator. None of these components currently exist as code.

---

# Component Design (proposed)

## Doctor Portal

Intended to be the primary interface used during patient consultation.

Proposed responsibilities:

* Doctor authentication
* Patient registration
* Disease selection
* Disease duration management
* Dietary preference collection
* Meal plan generation
* PDF download

The frontend is intended to contain minimal business logic, with all validation and processing handled by the backend.

---

## Backend API

Intended to coordinate the complete workflow.

Proposed responsibilities include:

* Authentication
* Patient CRUD operations
* Active disease calculation
* Disease mapping retrieval
* Prompt construction
* LLM invocation
* PDF generation
* QR generation
* Patient data retrieval

Business logic is intended to be centralized to keep the frontend lightweight.

---

## Database

Firebase is proposed to store persistent application data.

### Doctor

```text
Doctor
--------
doctorId
name
clinic
specialization
city
contact
```

### Patient

```text
Patient
--------
patientId
doctorId
name
age
gender
phone
dietPreference
region
diseases[]
createdAt
```

Each disease would store:

```text
Disease
--------
name
type
startDate
endDate
```

This is intended to allow temporary illnesses to expire automatically.

---

# Disease Knowledge Base (proposed)

Rather than embedding medical knowledge directly into prompts, MediPlate proposes storing dietary guidance in structured JSON mappings.

Example (illustrative, not yet built or clinically reviewed):

```json
{
  "disease": "Type 2 Diabetes",
  "avoid": [
    "White Rice",
    "Sugar"
  ],
  "prefer": [
    "Moong Dal",
    "Jowar Roti"
  ],
  "watch": [
    "Banana"
  ],
  "notes": "Prefer foods with a low glycemic index."
}
```

Intended benefits:

* Easier maintenance
* Version control
* Transparent recommendations
* Reusable across different AI providers

The open question is who authors and clinically validates these mappings, and what review process they go through before they are trusted as ground truth for evaluation. This is currently unresolved and matters more than most of the engineering decisions below.

---

# Diet Generation Pipeline (proposed)

```text
Patient Profile
      │
      ▼
Retrieve Active Diseases
      │
      ▼
Load Disease Mappings
      │
      ▼
Construct System Prompt
      │
      ▼
Large Language Model
      │
      ▼
Generate Meal Plan
      │
      ▼
PDF / Patient Assistant Response
```

The prompt would include:

* Patient demographics
* Dietary preferences
* Active conditions
* Disease mappings
* Clinical instructions

The model would be instructed to:

* Use only provided disease mappings
* Resolve conflicting recommendations conservatively
* Generate practical Indian meals
* Explain important recommendations
* Avoid unsupported medical advice

“Resolve conflicting recommendations conservatively” is the design-level statement of the research question in README.md. At present it is only a prompt instruction, with no defined mechanism or evaluation for whether the model actually does this reliably. Making that mechanism explicit and testable is the main open design problem in this document.

---

# Time-Aware Disease Resolution (proposed)

Each disease would be stored with either:

* Permanent status
* Temporary duration

Whenever patient information is requested, the backend would evaluate active conditions.

Example:

```text
Stored Diseases

Diabetes
Permanent

Hypertension
Permanent

Cold
7 Days

↓

Current Date

↓

Active Diseases

Diabetes
Hypertension
```

This is intended to ensure recommendations automatically evolve as temporary illnesses resolve.

---

# Patient Assistant (proposed)

The patient assistant is proposed to be stateless.

Workflow:

```text
Patient
   │
   ▼
Patient ID
   │
   ▼
Backend API
   │
   ▼
Retrieve Active Conditions
   │
   ▼
Construct Prompt
   │
   ▼
LLM
   │
   ▼
Response
```

Proposed interactions:

* Diet generation
* Dietary Q&A
* Meal image evaluation

Patient-specific information would always be retrieved from the backend instead of being stored in the conversation.

---

# PDF Generation (proposed)

Every generated report would contain:

* Doctor branding
* Patient information
* AI-generated meal plan
* Patient ID
* QR Code
* Medical disclaimer

The PDF is intended to act as the bridge between the physical consultation and the digital assistant.

---

# Security Considerations (proposed)

Proposed security measures include:

* Authenticated doctor accounts
* Backend-only database access
* Unique patient identifiers
* HTTPS communication
* Encrypted data storage
* Audit logging

Patient information should never be directly exposed to the AI service without backend validation. None of these measures have been implemented or reviewed yet; they are design intentions.

---

# Design Decisions

| Decision                    | Rationale                                                         |
| --------------------------- | ----------------------------------------------------------------- |
| React + FastAPI             | Simple, modern full-stack architecture                            |
| Firebase                    | Managed backend with rapid development                            |
| JSON disease mappings       | Maintainable and reusable medical knowledge                       |
| LLM-generated meal plans    | Faster path to a testable prototype with flexible meal generation |
| QR-based onboarding         | Eliminates app installation for patients                          |
| Stateless patient assistant | Ensures recommendations always use the latest patient data        |

---

# Deployment Considerations

The proposed architecture is independent of the conversational interface used to interact with patients. One possible deployment strategy is to provide patients with a QR code following the clinical consultation that opens a dedicated conversational AI interface.

During the initial interaction, the patient would authenticate using a unique identifier generated during the consultation. The conversational interface would retrieve the physician-created dietary profile and clinical constraints through secure backend APIs before initiating the interaction.

This deployment model, if built, could allow patients to continue receiving grounded dietary guidance without requiring a dedicated application while keeping physicians responsible only for the initial clinical onboarding. Since ongoing AI interactions would be patient-initiated, the framework has the potential to reduce deployment barriers for healthcare providers. The exact conversational platform remains an implementation choice, intentionally decoupled from the overall system architecture.

---

# Future Architecture

The current proposal has the LLM generate complete meal plans directly.

The surrounding architecture is designed so that, if warranted by the evaluation results, only the diet generation service would need to change in later versions:

```text
Current Proposal (v1)

Patient Profile
      │
      ▼
Disease Mappings
      │
      ▼
Large Language Model
      │
      ▼
Meal Plan


Possible Future Architecture

Patient Profile
      │
      ▼
Rule Engine
      │
      ▼
Meal Database
      │
      ▼
Meal Optimizer
      │
      ▼
Large Language Model
(Natural Language Generation Only)
```

This separation is intended to let MediPlate evolve toward a more deterministic and clinically robust recommendation system, if the evaluation work shows that's necessary, while preserving the same APIs, frontend, and patient experience.

---

# Research Questions

The proposed architecture is intended to investigate the following research questions rather than demonstrate a finished software system.

## RQ1 — Grounded Dietary Guidance

Can an AI system grounded in physician-created clinical profiles and structured disease knowledge provide safe, personalized dietary recommendations for patients with multiple co-existing medical conditions?

## RQ2 — Cross-Condition Conflict Resolution

Can grounded AI consistently identify and resolve conflicting dietary recommendations arising from multiple simultaneous medical conditions while remaining clinically conservative?

## RQ3 — Clinical Workflow Integration

Can a physician-initiated, AI-assisted framework extend post-consultation dietary care without increasing physician workload or disrupting existing clinical workflows?

## RQ4 — Modular Clinical AI

Does separating structured clinical knowledge, recommendation logic, and natural language generation improve transparency, maintainability, and future clinical validation compared with relying solely on end-to-end LLM prompting?

---

# Open Design Questions

Several important engineering and research questions remain open prior to implementation.

### Knowledge Base

* Who should author and clinically validate the disease knowledge base?
* What review and versioning process should govern updates to medical knowledge?

### Recommendation Logic

* What mechanism should resolve conflicting dietary recommendations between multiple simultaneous conditions?
* Should conflict resolution rely on explicit rules, evidence-weighted prioritization, or another approach?

### Evaluation

* How should recommendation quality be evaluated against physician- or dietitian-created dietary plans?
* Which metrics best measure recommendation safety, consistency, personalization, and clinical usefulness?

### Safety and Deployment

* How should uncertainty and recommendation confidence be communicated to patients?
* What ethical approvals and governance processes would be required before evaluation involving clinicians or patient data?

### Future Architecture

* As the number of co-existing medical conditions increases, does an LLM-based generator remain sufficiently reliable, or does a deterministic recommendation engine become necessary?
