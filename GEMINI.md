# 🧠 GEMINI Context: Spec Builder Framework

## 🎯 Meta-Identity
**Spec Builder** is an **Agentic Engineering Framework** designed to automate the lifecycle of industrial-grade technical specifications. It transforms ambiguous requirements (PRDs) into deterministic, implementable "Specs" through a self-healing iterative pipeline.

---

## 🏗️ Architecture & Component Roles

### 1. Control Plane: `.agent/workflows/`
Stateful, executable logic that orchestrates the AI Agent's behavior:
- **`spec-factory` (Master)**: Implements the recursive self-healing loop.
- **`audit-prd` (Gate)**: Quantitative quality filter for input requirements (**Gate: 85/100**).
- **`gen-specs` (Constructor)**: Handles Asset Crystallization and Refinement.
- **`audit-specs` (Judge)**: Strictly evaluates coverage, feasibility, and NFR compliance (**Pass: 90/100**).
- **`git` (Lifecycle)**: Release management and conventional commit enforcement.

### 2. Knowledge Plane: `prompt/`
Standardized technical DNA enforced across all workflows:
- **`veteran-persona`**: 15+ years experience context; rigorous, anti-handwaving, and direct.
- **`nfr-standards`**: Abstract requirements for Isolation, PII Scouring, Observability, and Idempotency.
- **`anti-nih`**: Mandatory reuse of industrial-grade components; rejection of custom wheels.
- **`execution-standards`**: Quantitative phrasing, TRACEability, and bilingual output specs.

### 3. Data Plane: `docs/` (Workspace)
Transient and persistent engineering assets. The `docs/` folder is a **User Workspace** where the framework produces its output. Individual files like `PRD_Spec_Builder.md` are **Example Inputs**, not part of the framework core.

---

## 🔄 The Self-Healing Loop (Logic Flow)
1. **Source Audit**: PRDs scoring < 85/100 are rejected immediately.
2. **Crystallization**: Generates atomic specs with traceability mappings.
3. **Verification**: Independent audit evaluates the generated spec set.
4. **Correction**: If Score < 90/100, the Agent reads the audit report and auto-patches the specs.
5. **Sync**: Mandatory cross-checking of `_Spec_Index_Matrix.md` and `Development_Plan.md`.

---

## 🛠️ Agent Operations (How to interact with this repo)

### 🚨 Critical Mandates
- **SSOT Enforcement**: `_Spec_Index_Matrix.md` is the single source of truth for requirement mapping.
- **Framework Separation**: DO NOT treat `docs/*.md` as core framework logic. Do not contaminate `prompt/` with technology-specific implementations from example data.
- **English Reasoning, Chinese Output**: Think in high-precision English logic, but all generated specifications and reports must be in **Simplified Chinese**.
- **Quantification**: Every technical statement must be measurable (P95 limits, QPS targets, Storage bytes).

### 📂 Directory Map
- `/prompt`: **READ-ONLY** (unless refining framework standards).
- `/.agent/workflows`: **READ-ONLY** (unless optimizing automation logic).
- `/docs`: **READ-WRITE** (Main workspace for production).
- `/templates`: **READ-ONLY** (Blueprint library).

---

## 📈 Current Version: v0.1.0
*Focus: Automated pipeline stability, self-healing recursion, and architectural decoupling.*
