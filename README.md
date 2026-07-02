# MediPlate

> AI-assisted post-diagnostic nutrition platform for Indian clinical workflows.

![Status](https://img.shields.io/badge/status-MVP-blue)
![Frontend](https://img.shields.io/badge/Frontend-React-61DAFB)
![Backend](https://img.shields.io/badge/Backend-FastAPI-009688)
![Database](https://img.shields.io/badge/Database-Firebase-FFCA28)
![AI](https://img.shields.io/badge/AI-LLM-orange)

---

## Overview

MediPlate is a doctor-first, AI-assisted nutrition platform designed to simplify post-diagnostic dietary care for Indian patients managing one or more medical conditions.

Instead of relying on patients to search for conflicting dietary advice online, MediPlate enables physicians to generate personalized Indian meal plans during consultation. Patients receive a branded PDF containing a QR code and unique Patient ID that provides continued AI-assisted dietary guidance after leaving the clinic.

The project focuses on improving the clinical workflow rather than replacing it—the doctor remains responsible for diagnosis while the AI assists with personalized nutrition recommendations.

---

## The Problem

Doctors often have only a few minutes per consultation.

Patients may simultaneously have:

- Type 2 Diabetes
- Hypertension
- IBS
- CKD
- Temporary illnesses like fever or post-surgery recovery

These conditions frequently have conflicting dietary requirements, making it difficult to provide comprehensive nutritional counselling during appointments.

Most patients eventually rely on generic internet advice, which is often inconsistent and not tailored to Indian diets or multiple medical conditions.

---

## Our Solution

MediPlate streamlines dietary care through a simple workflow:

```text
Doctor
   │
Registers Patient
   │
Selects Medical Conditions
   │
Generates Personalized Diet Plan
   │
Prints PDF with QR Code
   │
──────────────────────────────
               │
               ▼
Patient Scans QR
               │
Patient Assistant Retrieves Profile
               │
AI Provides Ongoing Dietary Guidance
```

---

## Features

### Doctor Portal

- Doctor authentication
- Patient registration
- Disease selection
- Permanent & temporary disease support
- Dietary preferences
- AI-generated meal plans
- PDF generation
- QR code generation

### Patient Assistant

- Personalized meal plans
- Diet-related Q&A
- Food image analysis
- Hindi & English support
- Automatic handling of expired temporary diseases

---

## System Architecture

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

A more detailed explanation of the architecture is available in **DESIGN.md**.

---

## Technology Stack

| Layer | Technology |
|--------|------------|
| Frontend | React |
| Backend | FastAPI |
| Database | Firebase |
| AI | OpenAI / Claude Compatible APIs |
| PDF | Report Generation |
| QR | QR Code Generation |

---

## Project Structure

```text
MediPlate/

├── frontend/
├── backend/
├── disease_mappings/
├── pdf_generator/
├── prompts/
├── DESIGN.md
└── README.md
```

---

## Current Workflow

### Doctor

1. Login
2. Register patient
3. Select medical conditions
4. Set temporary disease durations
5. Generate diet plan
6. Print PDF

### Patient

1. Scan QR code
2. Enter Patient ID
3. AI retrieves profile
4. Ask dietary questions
5. Generate new meal plans
6. Upload meal photos for analysis

---

## AI Design

The current MVP uses a Large Language Model to generate complete meal plans.

The LLM receives:

- Patient profile
- Dietary preferences
- Active diseases
- Structured disease mappings
- Clinical instructions

The model is prompted to:

- Use only the supplied disease mappings
- Resolve conflicting recommendations conservatively
- Generate practical Indian meals
- Explain dietary choices
- Avoid unsupported medical advice

Future versions may replace the LLM meal generation module with a deterministic recommendation engine while keeping the surrounding architecture unchanged.

---

## Roadmap

### Phase 1

- Doctor Portal
- Disease Knowledge Base
- Patient Registration
- Meal Plan Generation
- PDF + QR Code

### Phase 2

- Patient Assistant
- Food Image Analysis
- Hindi Support

### Phase 3

- Doctor Branding
- Regional Cuisine Support
- Clinical Validation
- Production Deployment

---

## Medical Disclaimer

MediPlate provides AI-assisted dietary guidance and is intended to support, not replace, clinical decision making.

The platform does **not** provide medical diagnosis or treatment recommendations. Patients should always consult their treating physician before making significant dietary or medical changes.

---

## Future Improvements

- Rule-based recommendation engine
- Structured Indian meal database
- Hospital EMR integration
- Laboratory value support
- Allergy management
- Nutritional analytics
- Evidence-backed recommendations

---

## License

This project is currently under active development for educational and research purposes.