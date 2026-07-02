# MediPlate Design Document

> This document describes the internal architecture and engineering decisions behind MediPlate. For an overview of the project, motivation, features, and setup instructions, refer to **README.md**.

---

# Design Principles

The architecture of MediPlate is guided by the following principles:

- **Doctor-first:** The platform integrates into existing clinical workflows rather than replacing them.
- **Structured over free-form:** Patient information and disease data remain structured throughout the system.
- **Grounded AI:** Every recommendation is generated using curated disease mappings instead of relying solely on an LLM's pretrained knowledge.
- **Modular AI:** The meal generation component is isolated so it can later be replaced by a deterministic recommendation engine.
- **Simple patient experience:** Patients should require no installation or account creation.

---

# Overall Architecture

```text
                    +----------------------+
                    |    Doctor Portal     |
                    | (React Frontend)     |
                    +----------+-----------+
                               |
                               | HTTPS
                               |
                    +----------v-----------+
                    |     FastAPI API      |
                    | Authentication       |
                    | Patient Management   |
                    | Diet Generation      |
                    +-----+----------+-----+
                          |          |
              +-----------+          +----------------+
              |                                   |
    +---------v---------+             +-----------v-----------+
    | Firebase Database |             | Disease Knowledge Base|
    | Patients          |             | JSON Mappings         |
    | Doctors           |             | Clinical Guidelines   |
    +---------+---------+             +-----------+-----------+
              |                                   |
              +-------------------+---------------+
                                  |
                       +----------v-----------+
                       | Diet Generation      |
                       | Service (LLM)        |
                       +----------+-----------+
                                  |
                 +----------------+----------------+
                 |                                 |
      +----------v----------+          +-----------v-----------+
      | PDF Generation      |          | Patient Assistant API |
      | QR Code             |          | Chat & Vision         |
      +---------------------+          +-----------------------+
```

The FastAPI backend serves as the orchestration layer, coordinating requests between the frontend, database, disease knowledge base, AI service, and PDF generator.

---

# Component Design

## Doctor Portal

The Doctor Portal is the primary interface used during patient consultation.

Responsibilities:

- Doctor authentication
- Patient registration
- Disease selection
- Disease duration management
- Dietary preference collection
- Meal plan generation
- PDF download

The frontend contains minimal business logic, with all validation and processing handled by the backend.

---

## Backend API

The backend coordinates the complete workflow.

Primary responsibilities include:

- Authentication
- Patient CRUD operations
- Active disease calculation
- Disease mapping retrieval
- Prompt construction
- LLM invocation
- PDF generation
- QR generation
- Patient data retrieval

Business logic is intentionally centralized to keep the frontend lightweight.

---

## Database

Firebase stores persistent application data.

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

Each disease stores:

```text
Disease
--------
name
type
startDate
endDate
```

This allows temporary illnesses to expire automatically.

---

# Disease Knowledge Base

Rather than embedding medical knowledge directly into prompts, MediPlate stores dietary guidance in structured JSON mappings.

Example:

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

Benefits:

- Easier maintenance
- Version control
- Transparent recommendations
- Reusable across different AI providers

---

# Diet Generation Pipeline

The meal generation service follows the pipeline below.

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

The prompt includes:

- Patient demographics
- Dietary preferences
- Active conditions
- Disease mappings
- Clinical instructions

The model is instructed to:

- Use only provided disease mappings.
- Resolve conflicting recommendations conservatively.
- Generate practical Indian meals.
- Explain important recommendations.
- Avoid unsupported medical advice.

---

# Time-Aware Disease Resolution

Each disease is stored with either:

- Permanent status
- Temporary duration

Whenever patient information is requested, the backend evaluates active conditions.

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

This ensures recommendations automatically evolve as temporary illnesses resolve.

---

# Patient Assistant

The patient assistant is intentionally stateless.

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

Supported interactions:

- Diet generation
- Dietary Q&A
- Meal image evaluation

Patient-specific information is always retrieved from the backend instead of being stored in the conversation.

---

# PDF Generation

Every generated report contains:

- Doctor branding
- Patient information
- AI-generated meal plan
- Patient ID
- QR Code
- Medical disclaimer

The PDF acts as the bridge between the physical consultation and the digital assistant.

---

# Security Considerations

Current security measures include:

- Authenticated doctor accounts
- Backend-only database access
- Unique patient identifiers
- HTTPS communication
- Encrypted data storage
- Audit logging

Patient information is never directly exposed to the AI service without backend validation.

---

# Design Decisions

| Decision | Rationale |
|----------|-----------|
| React + FastAPI | Simple, modern full-stack architecture |
| Firebase | Managed backend with rapid development |
| JSON disease mappings | Maintainable and reusable medical knowledge |
| LLM-generated meal plans | Faster MVP with flexible meal generation |
| QR-based onboarding | Eliminates app installation for patients |
| Stateless patient assistant | Ensures recommendations always use the latest patient data |

---

# Future Architecture

The current MVP allows the LLM to generate complete meal plans.

The surrounding architecture has been designed so that only the diet generation service needs to change in future versions.

```text
Current MVP

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


Future Architecture

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

This separation allows MediPlate to evolve toward a more deterministic and clinically robust recommendation system while preserving the existing APIs, frontend, and patient experience.