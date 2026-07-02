# MediPlate Design Document

## Overview

MediPlate is a doctor-first, AI-assisted post-diagnostic nutrition platform designed for Indian clinical workflows. It helps physicians provide personalized dietary guidance for patients managing one or more chronic or temporary medical conditions without increasing consultation time.

Unlike traditional diet or calorie-tracking applications, MediPlate begins inside the clinic. During consultation, the doctor registers the patient, selects diagnosed conditions, and generates a personalized Indian diet plan. The patient receives a branded PDF containing a unique Patient ID and QR code, allowing continued dietary guidance through an AI assistant after leaving the clinic.

The MVP uses a Large Language Model (LLM) to generate complete meal plans using structured disease mappings. The architecture is intentionally modular so the diet generation engine can later be replaced with a deterministic recommendation system without requiring changes to the rest of the application.

---

# Design Goals

The primary objectives of MediPlate are:

- Reduce the amount of dietary counselling required during consultations.
- Generate personalized Indian meal plans for patients with multiple medical conditions.
- Handle both permanent and temporary diseases.
- Support multilingual interactions (English and Hindi).
- Minimize patient onboarding friction through QR-based access.
- Keep the AI component modular for future improvements.
- Provide a scalable platform suitable for clinics and hospitals.

---

# System Architecture

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

The FastAPI backend acts as the central orchestration layer, coordinating communication between the frontend, database, disease knowledge base, and AI services.

---

# Core Components

## Doctor Portal

The doctor portal serves as the primary interface used during consultations.

Responsibilities include:

- Doctor authentication
- Patient registration
- Disease selection
- Duration selection for temporary conditions
- Dietary preference collection
- Meal plan generation
- PDF generation

The portal is designed to complete the entire workflow within a few minutes.

---

## Backend API

The backend coordinates every system component.

Responsibilities include:

- Authentication
- Patient management
- Disease validation
- Active disease resolution
- LLM prompt construction
- PDF generation
- QR code generation
- Patient API endpoints

It acts as the single source of truth for all business logic.

---

## Patient Database

Firebase stores all application data.

Example entities include:

### Doctor

- Doctor ID
- Name
- Clinic
- Contact Information

### Patient

- Patient ID
- Personal Details
- Dietary Preferences
- Active Diseases
- Temporary Disease Expiry Dates

Generated diet plans may also be stored for future retrieval.

---

## Disease Knowledge Base

Every supported disease has a structured JSON document containing dietary guidance.

Each mapping includes:

- Foods to Avoid
- Preferred Foods
- Foods Requiring Caution
- Clinical Notes

Example:

```json
{
  "disease": "Type 2 Diabetes",
  "avoid": [...],
  "prefer": [...],
  "watch": [...],
  "notes": "..."
}
```

The LLM is grounded using these mappings rather than relying solely on its internal knowledge.

---

# Doctor Workflow

1. Doctor logs into the portal.
2. A new patient is registered.
3. Diagnosed conditions are selected.
4. Temporary diseases receive expiry durations.
5. Dietary preferences are recorded.
6. The backend retrieves disease mappings.
7. A prompt is constructed.
8. The LLM generates a personalized meal plan.
9. A branded PDF containing a QR code and Patient ID is generated.

---

# Patient Workflow

1. Patient receives the printed PDF.
2. QR code opens the AI assistant.
3. Patient enters their Patient ID.
4. Backend retrieves active conditions.
5. Expired temporary conditions are automatically removed.
6. The assistant answers dietary questions, generates meal plans, or evaluates uploaded food images.

---

# Diet Generation Pipeline

```text
Patient Profile
        +
Disease Mapping JSON
        ↓
Prompt Construction
        ↓
Large Language Model
        ↓
Conflict-aware Meal Plan
        ↓
PDF / Chat Response
```

The LLM receives:

- Patient demographics
- Dietary preferences
- Active diseases
- Disease mappings
- Clinical instructions

The system prompt instructs the model to:

- Use only the supplied disease mappings.
- Generate practical Indian meals.
- Resolve conflicting recommendations conservatively.
- Explain important dietary choices.
- Avoid unsupported medical advice.
- Recommend physician consultation whenever uncertainty exists.

Although the MVP allows the LLM to generate complete meal plans, the architecture allows this module to be replaced by a deterministic recommendation engine in future versions without affecting the surrounding system.

---

# Time-Aware Disease Resolution

Each disease is stored with either:

- Permanent status
- Temporary duration

Whenever patient information is requested, the backend automatically filters expired temporary conditions.

Example:

```text
Patient

Diabetes        Permanent
Hypertension    Permanent
Cold            7 Days
Post Surgery    21 Days

↓

Current Date Evaluation

↓

Active Conditions Returned
```

This ensures dietary recommendations remain current without requiring manual updates.

---

# Patient Assistant

The patient assistant supports three interaction modes.

### 1. Diet Plan Generation

Patients can request meal plans for any duration.

Examples:

- 3-day plan
- 7-day plan
- Weekly plan

---

### 2. Dietary Questions

Examples include:

- Can I eat rice?
- Which dal is best?
- Can I eat curd at night?

The assistant remains restricted to dietary guidance related to the patient's active medical conditions.

---

### 3. Meal Image Analysis

Patients may upload food photographs.

The assistant:

- Identifies visible dishes
- Compares foods against active disease mappings
- Assigns a meal score
- Suggests healthier alternatives

---

# Security

The system is designed around basic healthcare data protection principles.

Current measures include:

- Authenticated doctor accounts
- Unique patient identifiers
- HTTPS communication
- Encrypted database storage
- Backend-only access to patient records
- Audit logging for major operations

The AI assistant never stores independent patient state and always retrieves current information from the backend using the Patient ID.

---

# Current Limitations

The current MVP intentionally prioritizes rapid development.

Known limitations include:

- Meal plans are generated entirely by the LLM.
- Clinical recommendations depend on curated disease mappings.
- Laboratory values are not yet incorporated.
- Medication interactions are outside project scope.
- The platform is intended for dietary guidance and not medical diagnosis.

---

# Future Improvements

The current architecture supports several future enhancements without major redesign.

Planned improvements include:

- Rule-based recommendation engine
- Structured Indian meal database
- Laboratory value support
- CKD stage-specific recommendations
- Allergy management
- Nutritional analysis
- Hospital EMR integration
- Doctor analytics dashboard
- Evidence scoring for recommendations
- Recommendation validation before LLM response

---

# Design Decisions

Several architectural decisions were made to simplify the MVP while supporting future expansion.

| Decision | Reason |
|----------|--------|
| Doctor-first workflow | Fits existing clinical practice |
| QR-based patient onboarding | No app installation required |
| Structured disease mappings | Grounds LLM responses |
| LLM meal generation | Fast MVP development |
| Firebase | Simple managed backend |
| FastAPI | Lightweight API framework |
| Modular AI service | Easy future replacement |

---

# Future Architecture

Although the MVP uses an LLM for complete meal generation, the surrounding architecture remains unchanged if a deterministic recommendation engine is introduced.

```text
Current MVP

Patient Profile
        ↓
Disease Mapping
        ↓
Large Language Model
        ↓
Meal Plan


Future Version

Patient Profile
        ↓
Rule Engine
        ↓
Meal Database
        ↓
Meal Optimizer
        ↓
Large Language Model
(Language Generation Only)
```

This separation allows MediPlate to evolve toward a more clinically deterministic system while preserving the same APIs, frontend, and patient experience.