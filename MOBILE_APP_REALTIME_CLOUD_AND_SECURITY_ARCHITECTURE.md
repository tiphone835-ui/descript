# Deprescript AI: Production Real-Time Mobile & Cloud Security Architecture (Android & iOS)

**Document Version:** 2.0 (Production Roadmap 2026)  
**Target Platforms:** iOS (App Store) & Android (Google Play Store)  
**Regulatory Standards:** HIPAA (USA), GDPR (EU), Digital Personal Data Protection (DPDP) Act 2023 (India), ISO 27001, and NABH Digital Health Guidelines.

---

## 1. System Topology Overview

```
                      ┌─────────────────────────────────────────┐
                      │          NATIVE MOBILE CLIENTS          │
                      │       Flutter (iOS & Android)           │
                      └────────────────────┬────────────────────┘
                                           │
                        Mutual TLS 1.3     │   SSL Certificate Pinning
                        Biometric Auth     │   Encrypted Local SQLite Cache
                                           │
                                           ▼
                      ┌─────────────────────────────────────────┐
                      │       SECURE CLOUD API GATEWAY          │
                      │    FastAPI / Node.js on Cloud Run       │
                      │    (HIPAA BAA Compliant Environment)    │
                      └───────┬─────────────────────────┬───────┘
                              │                         │
            WebSockets / RLS  │                         │  Zero-Retention PHI
                              ▼                         ▼
   ┌─────────────────────────────────────┐   ┌─────────────────────────────────────┐
   │     MANAGED CLOUD DATABASE          │   │      REAL-TIME AI CLUSTER           │
   │  Supabase / PostgreSQL (AWS / GCP)   │   │  • Google Cloud Vision (Doc OCR)    │
   │  • Row-Level Security (Tenant Sep)   │   │  • Google Medical Speech-to-Text    │
   │  • AES-256 Storage Encryption       │   │  • Gemini 1.5 Flash Private API     │
   │  • Point-in-Time Backup Recovery    │   │    (De-identified brand resolver)   │
   └─────────────────────────────────────┘   └─────────────────────────────────────┘
```

---

## 2. Recommended Frontend Architecture (Flutter)

### Why Flutter (Dart)?
1. **Single Unified Codebase:** 100% shared business logic and UI between iOS and Android.
2. **Native Performance:** AOT (Ahead-of-Time) compilation directly to machine ARM64 code ensures sub-second response times and 60-120 FPS animations.
3. **Medical Grade Sensor Access:** Direct hardware access to camera (for prescription OCR), microphone (for ambient dictation), and biometric sensors (Face ID, Touch ID, Fingerprint).
4. **Offline Resilience:** Local encrypted database (**SQLCipher**) caches all 25 FRID classes and 181 drugs locally so clinicians can run fall-risk audits even inside hospital basements without internet connectivity.

### Recommended Key Flutter Packages
* `camera` & `google_mlkit_text_recognition`: Instant on-device OCR preprocessing.
* `speech_to_text`: Real-time microphone audio capture.
* `supabase_flutter`: Real-time WebSocket connection to the cloud database.
* `local_auth`: iOS Face ID and Android Fingerprint biometric authentication.
* `flutter_secure_storage`: Hardware-backed keychain/keystore token storage.
* `sqflite_sqlcipher`: AES-256 local database encryption.

---

## 3. Real-Time Cloud Infrastructure & Synchronization

### 3.1 Database: PostgreSQL with Supabase (HIPAA BAA)
* **Real-time WebSockets:** Live streaming of prescription updates across hospital workstations, tablets, and mobile devices.
* **Row-Level Security (RLS) Policy Example:**
  ```sql
  -- Ensure clinicians can only view and edit their own patients or authorized hospital ward patients
  CREATE POLICY "Doctor Patient Access Policy"
  ON public.patient_encounters
  FOR ALL
  TO authenticated
  USING (
    doctor_id = auth.uid() OR 
    ward_id IN (SELECT ward_id FROM hospital_staff WHERE staff_id = auth.uid())
  );
  ```

### 3.2 Real-Time Microservices Pipeline
1. **Prescription Ingestion (Camera / Voice / Text):** Captured on mobile $\rightarrow$ streamed over TLS 1.3 to API gateway.
2. **De-Identification Layer (PHI Stripping):** The API gateway strips patient names, addresses, and phone numbers before sending raw drug lists to the AI normalizer.
3. **AI Resolution (Gemini API):** Converts unstructured brand strings (*"Pan-L OD, Amaryl 2mg"*) into structured generic tuples (*Pantoprazole 40mg + Levosulpiride 75mg; Glimepiride 2mg*).
4. **Clinical Decision Engine:** Checks 25 FRID classes, flags drug-drug interactions, calculates fall risk score (0–10), and computes a 4-week tapering calendar.
5. **Real-Time Broadcast:** The evaluated report is pushed back to the mobile UI in $< 1.2\text{ seconds}$ and persisted in the encrypted database.

---

## 4. Comprehensive Healthcare Security Blueprint ("The Fortress Model")

### 4.1 Device-Level Hardening (Mobile)
* **Hardware Biometrics:** Require biometric unlock (Face ID / Fingerprint) every time the app opens or returns from background.
* **Root & Jailbreak Detection:** Use `flutter_jailbreak_detection` to prevent app launch on modified devices vulnerable to memory inspection.
* **Screen Leak Prevention:**
  - Android: Enable `FLAG_SECURE` in `MainActivity.kt` to prevent screen capture, screenshotting, or task-switcher previews of patient medical records.
  - iOS: Add blur overlay on `applicationWillResignActive`.
* **Zero Plaintext Storage:** API keys, JWT access tokens, and encryption keys are stored exclusively in **iOS Keychain** and **Android Keymaster/Keystore**.

### 4.2 Network & Communication Hardening
* **Transport Layer Security (TLS 1.3 Only):** Disallow legacy TLS 1.0/1.1/1.2.
* **SSL Certificate Pinning:** Pin public key hashes (`HPKP`) in the Flutter app to eliminate Man-in-the-Middle (MitM) traffic sniffing on public hospital Wi-Fi.
* **DDoS & WAF Protection:** Cloudflare Enterprise Web Application Firewall in front of all cloud APIs.

### 4.3 Database & Storage Cryptography
* **Encryption at Rest:** All tables, disk blocks, and database backups encrypted using **AES-256-GCM** with customer-managed keys (AWS KMS / GCP Cloud KMS).
* **Field-Level Encryption:** Sensitive identifiers (Patient National ID / Aadhaar / ABHA ID / MRN) are encrypted with a separate key before writing to the database.
* **Immutable Audit Trail:** Maintain an append-only `clinical_audit_log` recording every read, edit, or export:
  ```sql
  CREATE TABLE audit_logs (
    log_id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    actor_id UUID NOT NULL,
    action_type VARCHAR(50) NOT NULL, -- 'VIEW_RECORD', 'DEPRESCRIBE_MED', 'EXPORT_PDF'
    resource_id UUID NOT NULL,
    ip_address INET NOT NULL,
    timestamp TIMESTAMPTZ DEFAULT NOW()
  );
  ```

---

## 5. Regulatory Compliance Checklist

| Standard | Jurisdiction | Specific Requirement Implemented |
| :--- | :--- | :--- |
| **HIPAA** | USA | Business Associate Agreement (BAA) executed with cloud vendor; automatic logoff after 5 mins of inactivity; encrypted backups. |
| **DPDP Act 2023** | India | Explicit patient consent capture prior to prescription upload; right to erasure; local data residency within Indian data centers. |
| **DISHA Guidelines** | India | Standardized Electronic Health Record (EHR) data formatting; readiness for Ayushman Bharat Digital Mission (ABDM / ABHA ID). |
| **Apple App Store (Guideline 5.1)** | Global | Clear Privacy Policy URL; justification for Camera and Microphone usage strings in `Info.plist`; Medical Disclaimers. |
| **Google Play (Health Policy)** | Global | Health App Declaration; secure credentials handling; HIPAA compliance statement. |

---

## 6. Prerequisites & Accounts Setup Guide

1. **Apple Developer Account (Organization Tier):**
   - Cost: \$99/year.
   - Requirements: D-U-N-S Number for your company/organization.
   - Purpose: Publish on the iOS App Store and distribute internal beta builds via TestFlight.
2. **Google Play Console Account:**
   - Cost: \$25 one-time registration fee.
   - Purpose: Publish on Android Google Play Store.
3. **Cloud Infrastructure (Supabase / AWS / GCP):**
   - Set up project in an Indian data center (e.g., `ap-south-1` Mumbai) for low latency and Indian data sovereignty.
4. **Google Cloud Platform (GCP) Project:**
   - Enable **Cloud Vision API** (for OCR) and **Vertex AI / Gemini API** (for clinical normalization).
5. **Medical Legal Disclaimer & Terms:**
   - Incorporate explicit Clinical Decision Support System (CDSS) disclaimer: *"This application provides decision support for qualified healthcare practitioners and does not replace clinical judgment."*

---

## 7. Phase-by-Phase Production Timeline

```
Week 1 - 2: Flutter Mobile Core & UI
  ├── Flutter project initialization (Android + iOS)
  ├── Animated branding splash screen & biometric login
  └── Local 25 FRID classes & 181 drugs SQLite database integration

Week 3 - 4: Cloud Backend & Auth
  ├── Supabase PostgreSQL deployment with Row-Level Security
  ├── Clinician authentication (Email OTP / State Medical Registration ID)
  └── Real-time WebSocket synchronization across devices

Week 5 - 6: Real-Time AI Engines
  ├── Native camera prescription scanning with Cloud Vision OCR
  ├── Voice-to-text dictation integration
  └── Gemini 1.5 Flash Indian brand-to-generic normalization engine

Week 7 - 8: Security Hardening & Store Deployment
  ├── SSL pinning, FLAG_SECURE screen lock, SQLCipher encryption
  ├── Audit logging & HIPAA/DPDP compliance validation
  └── TestFlight beta testing & submission to Apple App Store / Google Play
```
