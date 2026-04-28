# Learncast — Project Snapshot & Analysis

## Project Snapshot

Learncast addresses a common learning bottleneck: students miss class, re-reading dense notes is slow and passive, and traditional study formats don't fit modern schedules.

The tool targets anyone who learns on the go: commuters, gym-goers, busy professionals, and benefits auditory learners who retain information better through listening than reading.

Built as an MVP at the Ironhack AI Bootcamp, Learncast takes any PDF, `.txt` file, or pasted transcript and runs it through a three-stage Python pipeline:

1. **Data cleaning**
2. **GPT-4o mini summarisation** using the Feynman method and story arc structure
3. **OpenAI TTS audio generation**

The output is a personalised MP3 podcast recap, selectable voice, adjustable tone, delivered in minutes instead of hours of re-reading.

> **Known MVP limitations:** processing time for dense documents (~10 min), voice naturalness degrading at longer lengths, and URL scraping that needs further iteration.

---

## Stakeholder Impact

| # | Role / Relationship | Need | Risk if Ignored | Influence | Interest |
|---|---|---|---|---|---|
| 1 | **End Users / Students & Active Learners** | Accurate, engaging audio recaps that fit into a busy schedule and support retention without dedicated study time | Tool gets built for ideal conditions and never adopted. Core value proposition fails because real usage patterns were never considered | Low | High |
| 2 | **Founding Team** | Viable product-market fit, scalable architecture, and a clear monetisation path | Scope creep, technical debt, or a demo that never becomes production-ready. Capital spent with no deployable outcome | High | High |
| 3 | **IT / DevOps** | Stable, documented, deployable app with clear dependency management, no hardcoded secrets, and a reproducible environment | App runs on one developer's laptop and nowhere else | High | Low |
| 4 | **Legal / Compliance** | Clarity on what data is sent to third-party APIs, how uploads are stored, and whether ToS covers commercial use of generated content | User transcripts containing confidential content sent to external APIs without proper disclosure | High | Low |
| 5 | **Finance** | Predictable, modelable cost per user interaction to enable viable pricing (subscription, pay-per-use, freemium) | No rate limiting means a single large PDF triggers disproportionate API costs, making the product lose money on every heavy user | High | Low |
| 6 | **Customer Support** | Clear error messages, internal documentation, and enough pipeline understanding to diagnose common failures | Users get cryptic errors with no resolution path. Churn increases and brand reputation suffers | Low | Medium |
| 7 | **B2B Customers (L&D Programs)** | Data privacy guarantees, data leak prevention, quality commitment | Enterprise deals fall through at the security review stage. LearnCast locked out of the highest-revenue market segment | High | High |

---

## From Demo to Real Project

### Operations: Monitoring & Incident Response

Currently the app has basic Python logging to the terminal and no alerting. If the OpenAI API goes down or a TTS call fails, the user sees an error in the UI and nothing is logged persistently.

**For production:**
- Structured logging via Datadog or Sentry
- Uptime monitoring on the Hugging Face Space URL
- Defined uptime and reliability SLAs
- On-call rotation for peak study hours

---

### Security & Secrets Handling

During development, a `.env` file and Hugging Face repository secrets were used, reasonable for a prototype.

**For production:**
- Secrets manager (AWS Secrets Manager or HashiCorp Vault)
- Automatic secret rotation
- Pre-commit hook to scan for accidentally committed credentials

---

### Data Lifecycle: PII, Retention & Training Data

Learncast currently processes transcript content entirely in memory and passes it directly to the OpenAI API. No data is stored server-side and there is no policy around post-processing content handling.

**For production:**
- Clear data retention policy (duration, training data opt-out)
- GDPR / FERPA compliance if used in educational institutions
- Explicit consent flows and the ability to delete data on request

---

### Error Handling & Edge Cases

The current pipeline handles the happy path well. Edge cases such as scanned PDFs, corrupted files, non-English transcripts, and very short inputs receive basic error messages with no recovery path.

**For production:**
- OCR fallback for scanned documents
- Clear user-facing error messages and input validation before API calls
- Retry logic for transient API failures
- **If Stage 2 fails:** display the raw cleaned transcript
- **If Stage 3 fails:** keep the generated script readable for manual review

---

### API Budget & Cost Management

Learncast currently makes sequential OpenAI API calls per generation with token limits up to 8,192 per call. No rate limiting, cost tracking, or user quotas exist.

**For production:**
- Per-user usage quotas
- Cost monitoring via the OpenAI usage dashboard
- Alerts for unexpected spend
- Evaluation of whether `gpt-4o-mini` remains the best cost-quality tradeoff at scale
- Caching for repeated or similar inputs to reduce redundant API calls

---

### Handoff & Client Documentation

The project has a README covering local setup, but it assumes comfort with conda environments, API keys, and command-line tools. No user-facing documentation exists for non-technical users.

**For production:**
- Deployment guide with screenshots
- Troubleshooting FAQ
- One-click install script
- Training sessions for client staff
- Defined support process: contact, response time commitment, bug tracking
- Agreed definition of done with client upfront

---

### Scope Beyond the Demo: Multilingual Support & Accessibility

The interface is functional but not optimised for end-user experience and is implicitly English-only.

**For production:**
- Full UX design process with user testing and accessible design patterns
- Clear error states and progress indicators for longer processing times
- Language detection and multilingual TTS for global learner bases
- Audio transcripts of generated content for users with hearing impairments

---

## Revision Brief

### Before

At the start of the project, success meant getting the pipeline to work: upload a transcript, get audio out. There was no formal scope document, no defined user, and no risk assessment. We assumed the happy path: clean PDFs, English content, one user at a time, and an unlimited API budget. If it ran locally and produced a podcast, we considered it done.

### After

Having thought through stakeholders and production reality, we would reframe success around three things: **who the tool is actually for**, **what it needs to handle reliably**, and **what done really means**.

Key shifts:
- **Narrow the MVP** to a specific user (e.g., Ironhack students reviewing bootcamp material in English) rather than building for everyone at once
- **Expand to instructors** as a second user type, since teachers uploading class PDFs is a natural extension with different needs around content control and accuracy
- **Define non-functional requirements upfront** — the API call vs. wait time tradeoff (up to 10 min for better output) is a decision a real client needs to approve before development starts
- **Add a security review gate** before any external deployment
- **Set a cost ceiling** per user session
- **Redefine done** to include edge case handling and performance benchmarks, not just the happy path
