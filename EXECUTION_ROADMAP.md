# 🗺️ EXECUTION_ROADMAP.md

## "Chronos Atlas" Phased Implementation Roadmap

This plan divides the project into manageable phases, prioritizing the MVP first, then scaling performance and features.

#### Phase 0: Foundation (Backend Core)
| Milestone | Deliverables | Key Technical Focus |
| :--- | :--- | :--- |
| **0.1 Setup & Schema** | Docker environment for 3 services (Django, PG, Redis). Define all Django Models. | **Docker/Containerization.** |
| **0.2 Performance Layer** | Apply all **PostgreSQL Time-Series Indexes** (GiST/BRIN) on `normalized_year` fields. | **Database Optimization.** |
| **0.3 ETL Proof-of-Concept** | Working Django Management Command using `SPARQLWrapper`. Implements **date transformation logic** to populate `normalized_year`. Load **1,000+** test figures. | **Data Integrity & ETL Scripting.** |
| **0.4 API Boilerplate** | Setup `graphene-django` and define all base types (`FigureType`, etc.). | **GraphQL Schema Definition.** |

---

#### Phase 1: MVP (Minimum Viable Product)
| Milestone | Deliverables | Key Technical Focus |
| :--- | :--- | :--- |
| **1.1 Master Timeline API** | Fully functional `timelineFigures` GraphQL Resolver accepting `startYear/endYear`. | **Optimized PG Query Logic.** |
| **1.2 Frontend & Visualization** | Basic frontend setup. Interactive, horizontal **Timeline View** rendering figure life-spans. | **Visualization Library Implementation.** |
| **1.3 Figure Detail View** | Profile page with the figure's bio, works, and the **Vertical Works/Events Timeline**. | **GraphQL Relational Resolving.** |
| **1.4 Launch** | Basic search, clean styling, and routing. **MVP is ready for initial external review.** | **Core UX/UI.** |

---

#### Phase 2: Polish & Graph Features
| Milestone | Deliverables | Key Technical Focus |
| :--- | :--- | :--- |
| **2.1 Core Caching** | Implement **Redis Caching** for the high-traffic `timelineFigures` Resolver. | **Performance & Redis Integration.** |
| **2.2 Asynchronous ETL** | Convert ETL script to a scheduled **Celery background task**. | **Celery Setup & Task Offloading.** |
| **2.3 Relational Features** | Load **Influence** and **Occupation** data. Update GraphQL to expose these complex relationships. | **Advanced ETL & Data Modeling.** |
| **2.4 Filtering & UX** | Implement dynamic filtering based on `Field` and `Occupation`. Add final styling. | **Frontend State Management.** |

---

#### Phase 3: Production Ready (V1.0)
| Milestone | Deliverables | Key Technical Focus |
| :--- | :--- | :--- |
| **3.1 Full-Scale Load** | Complete ingestion of **all target figures (10,000+)**. | **Database Monitoring & Tuning.** |
| **3.2 Hardening** | Implement API Rate Limiting, security, and backup procedures. | **Security & Stability.** |
| **3.3 Production Deployment** | Deploy to a cloud environment (e.g., AWS/GCP) using Docker. | **CI/CD Pipeline.** |
| **3.4 Publish** | **"Chronos Atlas" V1.0 is launched.** | **Final QA & Maintenance Plan.** |
