# MediPlate

> AI-assisted post-diagnostic nutrition platform for Indian clinical workflows.

![Status](https://img.shields.io/badge/status-MVP-blue)
![Frontend](https://img.shields.io/badge/Frontend-React-61DAFB)
![Backend](https://img.shields.io/badge/Backend-FastAPI-009688)
![Database](https://img.shields.io/badge/Database-Firebase-FFCA28)
![AI](https://img.shields.io/badge/AI-LLM-orange)

---

# Overview

MediPlate is a doctor-first, AI-assisted nutrition platform designed to simplify post-diagnostic dietary care for Indian patients managing one or more chronic or temporary medical conditions.

Instead of relying on patients to search through conflicting dietary advice online, MediPlate enables physicians to generate personalized Indian meal plans during consultation. Patients receive a branded PDF containing a unique Patient ID and QR code that allows them to continue receiving dietary guidance through an AI assistant after leaving the clinic.

The platform is designed to **augment clinical workflows rather than replace them**. Doctors remain responsible for diagnosis and treatment, while MediPlate provides personalized dietary guidance based on the patient's diagnosed conditions.

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

Many of these conditions have conflicting dietary recommendations.

For example:

| Condition | Recommendation |
|-----------|----------------|
| Diabetes | High-fiber foods are beneficial |
| IBS | Certain high-fiber foods may worsen symptoms |

Due to limited consultation time, detailed nutritional counselling is often skipped. Patients then rely on generic internet advice, which is frequently contradictory, Western-centric, and not tailored for Indian diets or multiple simultaneous conditions.

---

## Research Motivation

MediPlate is not intended to replace physicians or automate clinical decision-making. Instead, it explores how grounded AI systems can extend physician-directed dietary care beyond the consultation while preserving clinical oversight.

The project investigates whether a physician-initiated, AI-assisted framework can provide safe, personalized, and longitudinal dietary guidance for patients with multiple co-existing medical conditions. It also explores how such systems can reduce barriers to adoption by separating clinical onboarding from continuous patient interactions.

The current repository focuses on documenting the problem, system design, and research direction. The implementation and experimental evaluation remain future work.

---

# Solution

MediPlate extends the doctor's consultation by combining structured clinical knowledge with AI-assisted dietary guidance.

The doctor performs a one-time patient onboarding during consultation. Afterward, patients can independently access personalized meal plans and dietary guidance by scanning a QR code printed on their report.

---

# High-Level Workflow

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
Patient Assistant Retrieves Profile
               │
AI Provides Personalized Dietary Guidance
```

For implementation details and system architecture, see **[DESIGN.md](DESIGN.md)**.

---

# Features

## Doctor Portal

- Secure doctor authentication
- Patient registration
- Multiple disease selection
- Permanent & temporary disease tracking
- Dietary preference selection
- AI-generated Indian meal plans
- Doctor-branded PDF generation
- QR code generation
- Unique Patient IDs

---

## Patient Assistant

- Personalized diet plans
- Diet-related question answering
- Meal image analysis
- Hindi & English support
- Automatic handling of expired temporary diseases

---

# Technology Stack

| Layer | Technology |
|--------|------------|
| Frontend | React |
| Backend | FastAPI |
| Database | Firebase |
| AI | OpenAI / Claude Compatible APIs |
| PDF Generation | ReportLab / HTML Templates |
| QR Code | Python QRCode |
| Deployment | Vercel + Render (Planned) |

---

# Project Structure

```text
MediPlate/

├── frontend/              # React application
├── backend/               # FastAPI backend
├── disease_mappings/      # Structured disease knowledge
├── prompts/               # LLM prompts
├── pdf_generator/         # PDF generation service
├── DESIGN.md
└── README.md
```

---

# Current Workflow

### Doctor

1. Login
2. Register a new patient
3. Select diagnosed conditions
4. Configure temporary disease durations
5. Choose dietary preferences
6. Generate personalized meal plan
7. Print branded PDF

---

### Patient

1. Scan QR code
2. Enter Patient ID
3. AI retrieves active conditions
4. Request diet plans
5. Ask dietary questions
6. Upload meal photos for dietary analysis

---

# AI Approach

The current MVP uses a Large Language Model (LLM) to generate complete meal plans.

Rather than relying entirely on the model's internal knowledge, MediPlate grounds responses using structured disease mappings.

The LLM receives:

- Patient demographics
- Dietary preferences
- Active diseases
- Disease-specific dietary mappings
- Clinical instructions

The system prompt instructs the model to:

- Use only the supplied disease mappings
- Resolve conflicting recommendations conservatively
- Generate practical Indian meals
- Explain dietary decisions
- Avoid unsupported medical advice
- Recommend physician consultation whenever appropriate

The architecture is intentionally modular so the diet generation service can later be replaced with a deterministic recommendation engine without changing the rest of the platform.

---

# Roadmap

## Phase 1 (Current MVP)

- Doctor Portal
- Patient Registration
- Disease Knowledge Base
- AI Meal Plan Generation
- PDF Generation
- QR Code Integration
- Patient API

---

## Phase 2

- AI Patient Assistant
- Hindi Support
- Meal Image Analysis
- Regional Cuisine Preferences

---

## Phase 3

- Doctor Branding
- Clinical Validation
- Analytics Dashboard
- Production Deployment

---

# Future Improvements

- Rule-based recommendation engine
- Structured Indian meal database
- Laboratory value integration
- CKD stage-specific recommendations
- Allergy management
- Nutritional analytics
- Hospital EMR integration
- Evidence-backed recommendations

---

# Medical Disclaimer

MediPlate is an AI-assisted dietary guidance platform intended to support physicians and patients after diagnosis.

It **does not** provide medical diagnosis, prescribe treatment, or replace professional medical advice. Patients should always consult their treating physician before making significant dietary or medical changes.

---

# License

This project is currently under active development for educational and research purposes.