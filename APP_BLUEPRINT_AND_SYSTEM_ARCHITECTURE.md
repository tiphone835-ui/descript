# Deprescript App: Master System Architecture & Product Blueprint (2026)

## 1. Product Vision & Value Proposition
**Deprescript** is an AI-powered Clinical Decision Support System (CDSS) and medication optimization platform designed for geriatricians, physicians, and clinical pharmacists. It automates the detection of Fall-Risk-Increasing Drugs (FRIDs), unmasks hidden drug-drug, drug-disease, and drug-food interactions (including high-risk fixed-dose combinations in India), and generates patient-specific tapering roadmaps aligned with the 2025/2026 International Deprescribing Guidelines.

---

## 2. User Journey & Feature Specifications

### 2.1 Splash Screen & Motion Branding
* **Visual Presentation:** Clean modern dark/light mode with medical blue and emerald accents.
* **Animation Sequence:** Circular pulse waveform $\rightarrow$ App Logo Reveal $\rightarrow$ "Deprescript AI: Evidence-Based Medication Optimization" subtitle $\rightarrow$ Smooth transition to authentication.
* **Duration:** 1.5 seconds (skippable on tap for rapid clinical access).

### 2.2 Clinician Authentication
* **Sign-in Methods:** Mobile OTP, Doctor Registration Number (NMC / State Medical Council in India), Biometric PIN, or Demo Guest Access.
* **Role Selection:**
  - *Attending Physician / Geriatrician:* Full clinical prescription modification authority.
  - *Clinical Pharmacist:* Medication reconciliation and interaction auditing.
  - *Nurse / Care Coordinator:* Bedside fall scoring and vital sign entry.

### 2.3 Home Dashboard (Central Hub)
The dashboard features 6 intuitive, high-contrast touch cards:
1. **Deprescribing Engine (Flagship):** Multi-modal prescription intake and automated analysis.
2. **All FRIDs Explorer:** Categorized search across all 25 classes and 174+ generic entities.
3. **Clinical Guidelines:** Searchable summaries of UWA 2025, AGS Beers, STOPP/START v3, and ICMR-AIIMS guidelines.
4. **Manual Interaction Checker:** Multi-drug matrix analyzing Drug-Drug, Drug-Disease, and Drug-Food/Hydration conflicts.
5. **Clinical Team & Credibility:** Profiles, photos, degrees, and affiliations of the advisory board.
6. **Patient History & Database:** Archive of past clinical audit encounters with longitudinal search.

### 2.4 Multi-Modal Input & AI Processing Layer
Doctors select their preferred input method:
* **Option A: OCR Prescription Camera:** Camera capture or gallery upload of handwritten/printed prescriptions.
* **Option B: Text Input / Paste:** Direct typing or pasting of clinic notes and pharmacy lists.
* **Option C: Voice Dictation:** Real-time speech-to-text allowing doctors to dictate: *"Patient is taking Pan-D once daily, Glimepiride 2mg, and Telmisartan 40mg with recurrent morning dizziness."*

#### The AI Normalization & Correction Engine
* **Phonetic & Typo Correction:** Fixes handwriting OCR misreads (e.g., *"clonaz 0.5"* $\rightarrow$ *Clonazepam 0.5mg*, *"amlo 5"* $\rightarrow$ *Amlodipine 5mg*).
* **Indian Brand-to-Generic Resolution:**
  - *Pan-L / Pantocid-L* $\rightarrow$ **Pantoprazole (40mg) + Levosulpiride (75mg)** (Flags Levosulpiride for Drug-Induced Parkinsonism).
  - *Pan-D / Pantocid-D* $\rightarrow$ **Pantoprazole (40mg) + Domperidone (30mg)** (Flags Domperidone for extrapyramidal symptoms & QTc prolongation).
  - *Stemetil* $\rightarrow$ **Prochlorperazine** (Flags high-potency dopamine antagonism).
  - *Vertin* $\rightarrow$ **Betahistine**.
  - *Dynapar QPS / Voveran* $\rightarrow$ **Diclofenac**.
* **Human-in-the-Loop Verification:** Displays extracted entities with confidence chips before final processing.

### 2.5 Clinical Assessment & Decision Matrix
Integrates user-entered patient factors:
* **Hemodynamics:** Supine vs. 3-minute Standing BP (Flags Orthostatic Hypotension if $\Delta\text{SBP} \ge 20$).
* **Functional Mobility:** Timed Up and Go (TUG) score, chair stand stability.
* **Laboratory Biomarkers:** eGFR (renal dose adjustments), Serum Sodium ($Na^+$), HbA1c.
* **Active Comorbidities:** Hypertension, Heart Failure, Diabetes, Dementia, BPH, CKD.

### 2.6 Report Generation & Longitudinal Database
* **Bedside FRID Risk Score (0–10):** Low ($0\text{--}2$), Moderate ($3\text{--}5$), High ($\ge 6$).
* **Two Formatted Outputs:**
  1. *Doctor's Clinical Encounter Note:* Formal medical justification, Beers/STOPP rationale, and EHR copy-paste text.
  2. *Patient's 4-Week Tapering Calendar:* Visual day-by-day pill guide with rebound management warnings.
* **Local & Cloud Database:** Stores patient name, age, MRN, date, baseline score, deprescribed drugs, and scheduled review date in local storage / IndexedDB with cloud synchronization.

---

## 3. Technology Stack & Deployment Model

| Layer | Recommended Technology | Why Selected |
| :--- | :--- | :--- |
| **Frontend Framework** | **HTML5 + Tailwind CSS + Vanilla JS / React / Flutter** | Lightning-fast rendering, responsive mobile/desktop, zero bloatware. |
| **OCR Engine** | **Google Cloud Vision API / Tesseract.js** | High-accuracy handwriting extraction on pharmaceutical scripts. |
| **Voice Processing** | **Web Speech API / Whisper AI** | Offline/in-browser medical voice recognition. |
| **AI LLM Correction** | **Gemini Flash / Gemini Pro API** | Rapid structured JSON entity extraction and drug normalization. |
| **Local Database** | **IndexedDB / SQLite (Offline-First)** | Guarantees 100% functionality inside hospital clinics without internet. |
| **Cloud Database** | **PostgreSQL / Supabase / Firebase** | HIPAA & GDPR/DISHA compliant encrypted patient records. |
| **Export Engine** | **jsPDF / HTML2Canvas** | Instant client-side PDF prescription generation without server latency. |

---

## 4. Database Schema (JSON Representation)

```json
{
  "encounter_id": "DEP-2026-8910",
  "patient_profile": {
    "name": "Ramesh Chandra",
    "age": 76,
    "gender": "Male",
    "clinical_frailty_scale": 5,
    "vitals": {
      "supine_bp": "138/82",
      "standing_bp": "112/68",
      "orthostatic_drop": true,
      "resting_hr": 58,
      "tug_seconds": 16.5
    },
    "labs": {
      "egfr": 32,
      "serum_sodium": 129,
      "hba1c": 6.4
    },
    "conditions": ["Hypertension", "Type 2 Diabetes", "CKD Stage 3b", "Recurrent Falls"]
  },
  "extracted_regimen": [
    { "input_name": "Pan-L", "resolved_generic": "Pantoprazole + Levosulpiride", "is_frid": true, "frid_class": "Indirect FRID (Dopamine Antagonist)" },
    { "input_name": "Glim-2", "resolved_generic": "Glimepiride 2mg", "is_frid": true, "frid_class": "Sulphonylurea (Severe Hypoglycaemia)" },
    { "input_name": "Chlorthalidone 12.5", "resolved_generic": "Chlorthalidone", "is_frid": true, "frid_class": "Thiazide-like Diuretic (Hyponatraemia)" },
    { "input_name": "Telmisartan 40", "resolved_generic": "Telmisartan", "is_frid": true, "frid_class": "Antihypertensive (Orthostasis)" }
  ],
  "fall_risk_score": 8,
  "fall_risk_category": "HIGH",
  "deprescribing_actions": [
    {
      "drug": "Levosulpiride (from Pan-L)",
      "action": "Immediate Cessation",
      "rationale": "High-risk dopamine blocker causing parkinsonian gait freezing and falls.",
      "substitution": "Switch to plain Pantoprazole 20mg."
    },
    {
      "drug": "Glimepiride 2mg",
      "action": "Deprescribe & Substitute",
      "rationale": "Severe hypoglycaemia risk with eGFR 32 and tight HbA1c 6.4%.",
      "substitution": "Switch to Linagliptin 5mg daily (no renal adjustment needed)."
    },
    {
      "drug": "Chlorthalidone 12.5mg",
      "action": "Discontinue",
      "rationale": "Directly driving hyponatraemia (Na+ 129) and postural BP drop (138 -> 112).",
      "substitution": "Rely on single-agent Telmisartan; target BP 135-145 mmHg."
    }
  ],
  "created_at": "2026-10-31T20:20:00Z"
}
```
