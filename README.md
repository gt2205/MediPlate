# MediPlate

> A research proposal for a physician-initiated, AI-assisted framework for post-diagnostic dietary guidance in a clinical settings.

---

![Status](https://img.shields.io/badge/status-research%20proposal-lightgrey)
![Frontend](https://img.shields.io/badge/Frontend-React%20(planned)-61DAFB)
![Backend](https://img.shields.io/badge/Backend-FastAPI%20(planned)-009688)
![Database](https://img.shields.io/badge/Database-Firebase%20(planned)-FFCA28)
![AI](https://img.shields.io/badge/AI-LLM-orange)

---

# Overview

MediPlate is a research proposal for a doctor-first, AI-assisted nutrition framework aimed at simplifying post-diagnostic dietary care for Indian patients managing one or more chronic or temporary medical conditions.

Instead of relying on patients to search through conflicting dietary advice online, MediPlate proposes that physicians generate personalized Indian meal plans during consultation. Patients would receive a physician-generated PDF containing a unique Patient ID and QR code, allowing them to continue receiving dietary guidance through an AI assistant after leaving the clinic.

The platform is designed to **augment clinical workflows rather than replace them**. Doctors remain responsible for diagnosis and treatment; MediPlate's role is limited to dietary guidance grounded in the conditions a physician has already diagnosed.

**This repository currently contains the problem statement and system design. There is no implementation yet, and no results have been produced or evaluated.**

---

# The Problem

Doctors often have only **5–10 minutes** per consultation.

Patients frequently leave with multiple medical conditions such as:

- Type 2 Diabetes
- Hypertension
- IBS
- Chronic Kidney Disease
- Fatty Liver
- Temporary illnesses like fever or post-surgery recovery

Many of these conditions have conflicting dietary recommendations. For example:

| Condition | Recommendation |
|-----------|----------------|
| Diabetes | High-fiber foods are beneficial |
| IBS | Certain high-fiber foods may worsen symptoms |

Due to limited consultation time, detailed nutritional counselling is often skipped. Patients then rely on generic internet advice, which is frequently contradictory, Western-centric, and not tailored for Indian diets or multiple simultaneous conditions.

---

# Research Motivation

MediPlate is not intended to replace physicians or automate clinical decision-making. Instead, it explores how grounded AI systems can extend physician-directed dietary care beyond the consultation while preserving clinical oversight.

The project investigates whether a physician-initiated, AI-assisted framework can provide safe, personalized, and longitudinal dietary guidance for patients with multiple co-existing medical conditions, and — more specifically — whether such a system can be shown to resolve cross-condition dietary conflicts safely and consistently. It also explores how such systems can reduce barriers to adoption by separating clinical onboarding from continuous patient interactions.

---

# Proposed Solution

MediPlate proposes extending the doctor's consultation by combining structured clinical knowledge with AI-assisted dietary guidance.

The doctor would perform a one-time patient onboarding during consultation. Afterward, patients could independently access personalized meal plans and dietary guidance by scanning a QR code printed on their report.

---

# Proposed Workflow

```text
Doctor
   │
Registers Patient
   │
Selects Diagnosed Conditions
   │
Generates Diet Plan
   │
Prints PDF + QR Code
   │
──────────────────────────────
               │
               ▼
Patient Scans QR
               │
AI Assistant Retrieves Physician-Verified Profile
               │
AI Provides Personalized Dietary Guidance
```

For implementation details and system architecture, see **[DESIGN.md](DESIGN.md)**.

---

# Proposed Features

## Doctor Portal

- Doctor authentication
- Patient registration
- Multiple disease selection
- Permanent & temporary disease tracking
- Dietary preference selection
- AI-generated Indian meal plans
- Doctor-branded PDF generation
- QR code generation
- Unique Patient IDs

## Patient Assistant

- Personalized diet plans
- Diet-related question answering
- Meal image analysis
- Hindi & English support
- Automatic handling of expired temporary diseases

---

# Proposed Technology Stack

| Layer | Technology |
|--------|------------|
| Frontend | React |
| Backend | FastAPI |
| Database | Firebase |
| AI | OpenAI / Claude Compatible APIs |
| PDF Generation | ReportLab / HTML Templates |
| QR Code | Python QRCode |
| Deployment | Vercel + Render (planned) |

None of this has been implemented yet — this is the intended stack for a future implementation phase, not a description of what currently exists.

---

# Planned Repository Structure

```text
MediPlate/

├── frontend/              # React application (not yet implemented)
├── backend/               # FastAPI backend (not yet implemented)
├── disease_mappings/      # Structured disease knowledge (not yet implemented)
├── prompts/               # LLM prompts (not yet implemented)
├── pdf_generator/         # PDF generation service (not yet implemented)
├── DESIGN.md
└── README.md
```

---

# Proposed AI Approach

The proposed approach uses a Large Language Model (LLM) to generate complete meal plans, grounded in structured disease mappings rather than the model's internal knowledge alone.

The LLM would receive:

- Patient demographics
- Dietary preferences
- Active diseases
- Disease-specific dietary mappings
- Clinical instructions

The system prompt would instruct the model to:

- Use only the supplied disease mappings
- Resolve conflicting recommendations conservatively
- Generate practical Indian meals
- Explain dietary decisions
- Avoid unsupported medical advice
- Recommend physician consultation whenever appropriate

The architecture is intended to be modular, so the diet generation service could later be replaced with a deterministic recommendation engine without changing the rest of the platform. See [DESIGN.md](DESIGN.md#future-architecture) for details.

---

# Future Work

- Rule-based recommendation engine
- Structured Indian meal database
- Laboratory value integration
- CKD stage-specific recommendations
- Allergy management
- Nutritional analytics
- Hospital EMR integration
- Evidence-backed recommendations
- Clinical validation with physicians and nutrition experts

---

# Medical Disclaimer

MediPlate, as proposed, is intended to support physicians and patients after diagnosis.

It **does not** provide medical diagnosis, prescribe treatment, or replace professional medical advice. Patients should always consult their treating physician before making significant dietary or medical changes. No part of this proposal has been clinically validated.

---

# License

This project is currently under active development for educational and research purposes.