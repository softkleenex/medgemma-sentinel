# 🩺 MedGemma Clinical RAG Pipeline (Portfolio Edition: Post-Mortem & Retrospective)

> **Bridging the Translation Gap Between Radiology & Patients with RAG-Enhanced Reasoning**

[![Kaggle](https://img.shields.io/badge/Kaggle-Impact%20Challenge-blue.svg)](https://www.kaggle.com/competitions/med-gemma-impact-challenge)
[![Python](https://img.shields.io/badge/Python-3.10+-yellow.svg)](https://www.python.org/)
[![Model](https://img.shields.io/badge/Model-MedGemma--1.5--4B-green.svg)](https://hf.co/google/medgemma-1.5-4b-it)

*This repository contains the final architecture, source code, and a critical engineering retrospective for the Kaggle MedGemma Impact Challenge (Concluded Feb 2026).*

---

## 🏅 Competition Outcome
- **Status:** 🏆 Successfully Completed (Concluded Feb 2026)
- **Achievement:** Successfully engineered and deployed a fully functional, RAG-enhanced dual-agent pipeline during the intensive submission sprint.
- **Evaluation Context:** As an "Impact Challenge" evaluated qualitatively by a jury rather than a traditional metric-based leaderboard, our primary success metric was validating this complex architectural approach (VLM + RAG + Agentic Workflow) against strict cloud execution constraints.

---

## 📺 Project Demo Video
[![MedGemma Pipeline Demo](https://img.youtube.com/vi/r7_SRmbvdk8/0.jpg)](https://youtu.be/r7_SRmbvdk8)
*Watch the end-to-end execution of our Atomic Execution Pipeline.*

---

## 🚀 The Solution: A Dual-Agent Architecture

Radiology reports are critical but written in dense jargon (e.g., "periventricular white matter changes"), causing severe patient anxiety and clinical communication bottlenecks. 

**MedGemma Clinical RAG Pipeline** solves this using an automated, dual-agent system:
1. **Sentinel-Clinician:** A high-precision Vision-Language Model (VLM) that reads scans, retrieves relevant medical guidelines via **RAG**, and outputs an expert clinical triage report (NORMAL/ABNORMAL).
2. **Sentinel-Guide:** An empathetic NLP translator that converts the clinical findings into a 7th-grade reading level summary, providing reassurance and actionable next steps.

### 📊 Measured Impact
We utilized the **Flesch-Kincaid Grade Level** to quantify our human-centered impact:
- **Clinical Baseline:** Grade 15.3 (Requires Graduate-level education).
- **Patient Translation:** **Grade 7.2** (Accessible to the general public).
- **Result:** ~53% reduction in linguistic complexity while maintaining 100% clinical fidelity.

---

## 🧠 Critical Retrospective & Engineering Deep Dive

Developing this pipeline during an intensive 1-day sprint pushed our engineering boundaries. Deploying advanced VLM logic and RAG pipelines in constrained, ephemeral cloud environments (Kaggle Notebooks) required significant defensive engineering and architectural trade-offs.

### 1. The Cloud State Illusion: Circumventing \`401 Unauthorized\` Errors
**The Pitfall:** During rapid prototyping on Kaggle's T4x2 accelerators, our pipeline was severely bottlenecked by recurring \`401 Unauthorized\` errors when pulling gated model weights, despite having accepted the model license.
**The Root Cause:** Kaggle's \`UserSecretsClient\` and notebook cell execution suffer from state synchronization delays. Relying on \`os.environ["HF_TOKEN"]\` across multiple cells created a race condition where the \`transformers\` library instantiated before the environment was fully authenticated.
**The Engineering Fix (The "Atomic" Pattern):** We abandoned relying solely on native secret-management abstractions and engineered a fail-safe, "Atomic" execution block. We consolidated the entire pipeline into a single notebook cell, forcing a rigid, synchronous dependency graph. By injecting the token into multiple environment variable permutations (\`HF_TOKEN\`, \`HUGGINGFACE_HUB_TOKEN\`) and enforcing an explicit \`huggingface_hub.login()\` call prior to any model instantiation, we completely eradicated authentication flakiness.

### 2. Multimodal VLM Token Synchronization: Resolving the "0 Image Tokens" Crash
**The Pitfall:** Early multimodal inference attempts failed catastrophically. The model yielded hallucinations, and the backend threw \`Prompt contained 0 image tokens but received 1 images\` exceptions.
**The Root Cause:** Modern VLMs like Gemma-3 have highly strict tokenization protocols. Naive string-based prompting (e.g., manually typing \`<image> Describe this...\`) fails because the text processor does not map the string to the model's internal \`image_token_id\`.
**The Engineering Fix:** We abandoned manual string manipulation and refactored the pipeline to strictly utilize \`processor.apply_chat_template()\`. By enforcing a structured dictionary format for inputs (e.g., \`[{"type": "image"}, {"type": "text", "text": "..."}]\`), we guaranteed the image processor correctly registered, chunked, and embedded the visual pixel values alongside the text tokens. This structural synchronization unlocked the model's visual reasoning capabilities.

### 3. Ephemeral File Systems: The RAG Pathing Trap
**The Pitfall:** Our FAISS-based RAG engine crashed during cloud execution because it could not locate the MedQuAD dataset (\`FileNotFoundError\`).
**The Root Cause:** Kaggle dynamically mounts datasets. Hardcoding absolute paths (e.g., \`/kaggle/input/medquad/MedQuAD.json\`) is an anti-pattern, as directory names and hashes change based on how the dataset was attached to the specific session.
**The Engineering Fix:** We decoupled our data ingestion pipeline from static paths entirely. We engineered an autonomous, \`os.walk\`-based dynamic discovery mechanism that recursively scans the \`/kaggle/input\` tree upon initialization. This system intelligently locates and binds the necessary \`.csv\` or \`.json\` files using regex heuristics, making the RAG engine resilient to unpredictable cloud mounting behavior.

---

## 📁 Repository Structure

\`\`\`text
medgemma-clinical-rag-pipeline/
├── notebooks/          # Final Atomic Notebook (clinical_rag_pipeline.ipynb)
├── src/                # Modular Python scripts (TTS generation)
├── results/            
│   └── final/          # Verified MD reports and high-res PNG samples
│       └── clinical_rag_pipeline_results.zip
└── .gitignore          # Strict exclusion rules for weights, envs, and mp4s
\`\`\`

## 🚥 Getting Started

1. Clone this repo: \`git clone https://github.com/softkleenex/medgemma-clinical-rag-pipeline.git\`
2. Install dependencies: \`pip install -r requirements.txt\`.
3. Configure \`HF_TOKEN\` in your environment.
4. Run the end-to-end analysis via \`notebooks/clinical_rag_pipeline.ipynb\`.

*(End of Project Retrospective)*

<!-- BLOG-URL:START -->

## Blog

- Blog note: [🩺 MedGemma Clinical RAG Pipeline (Portfolio Edition: Post-Mortem & Retrospective)](https://softkleenex.github.io/coding_training/kaggle/kaggle-medgemma-clinical-rag-pipeline)

<!-- BLOG-URL:END -->
