# Changelog

This document records all version update history of the Danduola Agent Intelligent Audit Platform.

## v1.2.0 (2025-12-01)

::: tip Milestone Release
This update introduces the all-new Hybrid Policy Engine, supporting automatic retrieval of basis from PDF policy documents.
:::

### ✨ New Features
- **Hybrid Policy Engine**: 
  - Support uploading enterprise financial policy documents in PDF/Word format.
  - Added RAG (Retrieval-Augmented Generation) module for semantic retrieval of unstructured policies.
- **Multimodal Invoice Comparison**: Added seal recognition for invoice stamping areas, supporting detection of risks such as blurred or obstructed seals.
- **Webhooks Integration**: Support callbacks to external systems via Webhook after the audit process ends.

### 🚀 Improvements
- **Performance**: Rule engine cold start time reduced by 40%.
- **UX Optimization**: Optimized the popup position of the browser plugin in SAP systems to avoid blocking key input fields.
- **Log Audit**: Enhanced the granularity of operation logs, now tracking Token consumption for every AI decision.

### 🐛 Bug Fixes
- Fixed an issue where the sidebar could not be fully expanded on low-resolution screens.
- Fixed an issue where parsing failed for certain special format PDF invoices.

---

## v1.1.0 (2025-11-15)

### ✨ New Features
- **Prompt Debugger**: Added an online Prompt debugging tool in the Strategy Studio, supporting real-time preview of AI output.
- **Custom Risk Levels**: Support user-defined colors and labels for risk levels (e.g., High Risk, Medium Risk, Low Risk).

### 🚀 Improvements
- **Model Upgrade**: Underlying model upgraded to GPT-4o, significantly improving logical reasoning capabilities.
- **Internationalization**: Added full support for English interface.

---

## v1.0.0 (2025-10-01)

🎉 **Danduola Agent Initial Release!**

### Core Features
- **Companion Audit Plugin**: Supports Chrome/Edge browsers, seamlessly integrating with mainstream ERP systems.
- **Smart Rule Orchestration**: Natural language-based rule configuration system.
- **Full Voucher Scanning**: Supports OCR recognition for invoices, contracts, bank receipts, etc.
