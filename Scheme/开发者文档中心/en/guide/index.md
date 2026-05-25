# Platform Overview

**AI-Audit Platform** is a next-generation intelligent financial compliance audit system for large enterprises. It breaks the limitations of traditional OCR + rule engines by utilizing **LLM (Large Language Model)** and **Agent** technologies, achieving a leap from "formal audit" to "substantive audit".

## Table of Contents

- [Getting Started](./getting-started.md)
- [Configuration](./configuration.md)

## Core Pain Points Solved

In the traditional financial shared service center model, we solve the following core problems:

::: info 🚫 Fragmented Audit Experience
**Old Model**: Auditors need to switch screens between image systems, ERP, and OA.
**New Model**: **Browser Plugin** lives directly in the business system, providing audit suggestions "on demand" without switching windows.
:::

::: info 📉 Rigid Rule Maintenance
**Old Model**: Modifying an "accommodation limit" rule requires IT development intervention, with a long cycle.
**New Model**: **Natural Language Orchestration** of rules allows financial experts to adjust logic directly via Prompts, taking effect in seconds.
:::

::: info 🧩 Difficulty Processing Unstructured Data
**Old Model**: Can only compare numbers, unable to understand compliance risks in "business entertainment" notes.
**New Model**: **Multimodal AI** can read attachment images, understand approval flow notes, and combine with policy documents for semantic judgment.
:::

## System Architecture

The platform adopts a **"Client-Cloud-Brain"** layered architecture design:

- **Client**: Browser Plugin (Sidekick), responsible for environment perception and interaction.
- **Cloud (Core)**: Java Microservices, responsible for business gateway and data persistence.
- **Brain**: .NET Agent Engine, responsible for multimodal perception, reasoning, and decision-making.
