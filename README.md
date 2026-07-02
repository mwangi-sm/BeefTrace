# BeefTrace

A digital livestock traceability system that enables users to track the complete lifecycle of an animal from birth on to the final beef product purchased by consumers

## Table of Contents

- [Problem & Approach](#problem--approach)
- [User Roles](#user-roles)
- [Core Design Principles](#core-design-principles)
- [Tech Stack](#tech-stack)
- [Architecture](#architecture)
- [Project Structure](#project-structure)
- [Getting Started](#getting-started)
- [Current Status](#current-status)
- [Roadmap](#roadmap)
- [Reference Documents](#reference-documents)

---

## Problem & Approach

Kenya's beef supply chain has fragmented visibility — an animal's official identity often does not follow it cleanly through sale, transport, slaughter, and retail, leaving consumers with little confidence in provenance claims. BeefTrace KE addresses this by maintaining an **append-only event ledger** that records every verified touchpoint an animal passes through, anchored to its ANITRAC identity, and exposing a simplified, consumer-facing provenance view via QR code at the point of sale.

Provenance claims shown to consumers are expressed using a **T0–T5 confidence model**, reflecting how much of the chain has been independently verified rather than presenting an all-or-nothing trust claim.

## User Roles

The system supports eight distinct stakeholder roles, each with a tailored app experience and permission set:

| Role | Responsibility |
|---|---|
| Administrator | System configuration, user & role management |
| Veterinary Officer | Health records, vaccination tracking, certifications |
| Farmer | Animal registration, ownership records |
| Transport Officer | Movement permits, in-transit event logging |
| Market Officer | Sale/auction events, ownership transfer |
| Slaughterhouse Officer | Slaughter records, carcass linkage |
| Retailer | Product listing, QR code generation for cuts |
| Consumer | QR scanning, provenance lookup |

## Core Design Principles

These constraints come directly from the project blueprint and are treated as non-negotiable:

- **Offline-first** — the app must be fully usable in low/no-connectivity rural and market environments, syncing when connectivity returns.
- **Append-only event ledger** — no destructive edits to historical records; corrections are new events, not overwrites. No blockchain — a conventional, auditable ledger pattern is used instead.
- **Privacy-by-default** — data exposure is scoped to what each role and the consumer-facing view actually needs.
- **Complements ANITRAC** — BeefTrace links to and validates against Kenya's official animal registry; it does not attempt to replace it.
- **RBAC everywhere** — role-based access control is enforced both client-side and at the backend (Firestore Security Rules); neither layer alone is trusted.

## Tech Stack

| Layer | Choice |
|---|---|
| App framework | Flutter (Dart) |
| Architecture | Clean Architecture, feature-first folder structure |
| State management / DI | Riverpod |
| Local storage | Hive (offline-first cache) |
| Remote database | Firebase Firestore |
| Sync | Custom `SyncEngine` with exponential backoff retry |
| Auth | Firebase Authentication — Email/Password + Phone OTP |
| File storage | Firebase Storage |
| Backend logic | Firebase Cloud Functions (TypeScript, Blaze plan) |
| Routing | GoRouter with auth guards |
| Charts | fl_chart |
| Design system | Material 3, agricultural green palette, Poppins typeface |

## Architecture

```
┌─────────────────────────────────────────────┐
│                   UI Layer                   │
│   Screens · Widgets · Riverpod Consumers     │
├─────────────────────────────────────────────┤
│                Domain Layer                  │
│   Entities · Use Cases · Repository Contracts│
├─────────────────────────────────────────────┤
│                  Data Layer                  │
│  Repositories · Hive (local) · Firestore     │
│         (remote) · SyncEngine (bridge)       │
├─────────────────────────────────────────────┤
│              Firebase Backend                │
│  Auth · Firestore · Storage · Cloud Functions│
└─────────────────────────────────────────────┘
```

Local writes go to Hive first (so the app is fully usable offline), then the `SyncEngine` reconciles them with Firestore in the background with retry/backoff. RBAC is checked client-side for UX and re-enforced by Firestore Security Rules as the source of truth.

## Project Structure

```
lib/
├── core/
│   ├── constants/         # App-wide constants, Kenya reference data
│   ├── theme/              # Material 3 theme, agricultural palette
│   ├── errors/              # Exception types, Firebase error mapping
│   ├── services/            # Connectivity, audit logging
│   └── sync/                 # SyncEngine
├── features/
│   ├── auth/                 # Login, register, phone OTP, role assignment
│   ├── animals/               # Registration, list, detail, timeline
│   ├── qr/                     # QR generation & scanning
│   ├── transport/               # Permits, movement events
│   ├── slaughter/                 # Slaughterhouse records
│   ├── dashboard/                   # Role-aware home dashboard
│   ├── notifications/
│   ├── profile/
│   └── reports/
└── main.dart

functions/                      # Firebase Cloud Functions (TypeScript)
├── roleAssignment/
├── qrValidation/
├── transportNotifications/
├── vaccinationReminders/
└── consumerLookup/             # Public HTTP endpoint for QR scans
```

## Getting Started

### Prerequisites

- Flutter SDK (stable channel)
- Firebase CLI + FlutterFire CLI
- A Firebase project on the **Blaze** plan (required for Cloud Functions)

### Setup

```bash
# Install dependencies
flutter pub get

# Configure Firebase for this app
flutterfire configure

# Run on a connected device/emulator
flutter run
```

### Cloud Functions

```bash
cd functions
npm install
firebase deploy --only functions
```

### Firestore Rules & Indexes

```bash
firebase deploy --only firestore:rules,firestore:indexes,storage
```

## Current Status

The Flutter implementation is roughly **50% complete**. Delivered so far:

- Full folder structure, `pubspec.yaml`, core constants & theme
- Error handling and Firebase exception mapping
- Connectivity and audit log services
- All domain entities and Firestore-serialized data models
- Auth and animal repositories with offline queue support
- `SyncEngine` and Riverpod providers
- Auth screens: login, register, forgot password, phone OTP
- Animal registration (3-step flow with GPS capture)
- QR generation and scanning screens
- Dashboard with role-aware quick actions and fl_chart donut chart
- Animal list, detail, and timeline views
- Transport permit screens
- Slaughter record screens
- Notifications, profile/settings, reports screen
- Cloud Functions: role assignment, QR validation, transport notifications, vaccination reminders, public consumer lookup endpoint
- Firestore Security Rules (immutable event ledger pattern), Storage rules, composite indexes, Android/iOS platform config

## Roadmap

- [ ] Complete remaining ~50% of Flutter screens/features
- [ ] Seed Kenya-specific reference data (breeds, counties, ANITRAC codes)
- [ ] Finish authentication flow with role assignment end-to-end
- [ ] Fully connect Animal CRUD operations to the SyncEngine
- [ ] Validate and test Cloud Functions in a live Firebase environment

## Reference Documents

- JHUB BeefTrace Project Blueprint v1.0 — authoritative requirements & architecture source
- ANITRAC — Kenya's official animal traceability registry (external integration target)

---

*Developed under JHUB Africa, Jomo Kenyatta University of Agriculture and Technology (JKUAT).*
