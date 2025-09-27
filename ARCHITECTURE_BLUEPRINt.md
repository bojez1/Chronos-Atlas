# 🚀 ARCHITECTURE_BLUEPRINT.md

## "Chronos Atlas" Technical Architecture

This document outlines the definitive stack, components, and optimized data model for the project.

### 1.1 The Optimized Technology Stack

| Layer | Component | Technology | Rationale |
| :--- | :--- | :--- | :--- |
| **Backend Language** | **Python** 🐍 | Versatile, great for ETL and API logic. |
| **Web Framework** | **Django** | Provides a complete, secure structure (ORM, Admin, Auth). |
| **Primary Database** | **PostgreSQL** 🐘 | **Crucial for performance.** Superior time-series indexing and relational integrity. |
| **API Layer** | **GraphQL (Graphene-Django)** | Efficient, flexible data fetching for the complex Timeline View. |
| **Caching/Broker** | **Redis** | In-memory cache for API results; message broker for Celery tasks. |
| **Background Jobs** | **Celery** | Runs long-lived ETL/Data-update jobs asynchronously. |
| **Data Source** | **Wikidata (via SPARQLWrapper)** | Provides structured, open historical data. |
| **Deployment Standard** | **Docker/Docker Compose** | Ensures consistent, portable environments. |

---

### 1.2 The Refined Data Model (PostgreSQL Schema)

The schema is built for speed, relationship complexity, and future filtering needs.

#### A. Core Entities

| Table | Field | Data Type | Purpose & Optimization |
| :--- | :--- | :--- | :--- |
| **`Figure`** | `id`, `slug`, `name`, `summary` | PK, Char, Char, Text | Core biographical data. |
| | `wikidata_id` | CharField (Unique) | Essential for mapping and ETL updates. |
| | `birth_date`, `death_date` | DateField, DateField (Nullable) | Precise dates. |
| | **`normalized_birth_year`** | **IntegerField (Indexed)** | **CRITICAL:** Indexed for extremely fast timeline range queries. |
| | **`normalized_death_year`****| **IntegerField (Indexed, Nullable)** | **CRITICAL:** Indexed for extremely fast timeline range queries. |
| | **`instance_of_QIDs`** | ArrayField(CharField) | Stores raw Wikidata identifiers for flexible technical filtering. |
| **`WorkOrEvent`** | `id`, `figure` (FK), `title` | PK, FK, Char | Timeline milestones. |
| | `event_date` | DateField | Date the event occurred. |
| | **`event_type`** | CharField | Categorizes the milestone: `WORK`, `LIFE_EVENT`, `INVENTION`, etc. |

#### B. Relational and Taxonomy Tables

| Table | Field | Relationship Rationale |
| :--- | :--- | :--- |
| **`Influence`** | `id` | Primary key for the relationship itself. |
| | **`subject`** (FK to `Figure`) | The person the relationship **is about** (e.g., The one who was influenced). |
| | **`object`** (FK to `Figure`) | The person **related to** the subject (e.g., The influencer). |
| | **`relationship_type`** | Explicit type: `INFLUENCED_BY`, `STUDENT_OF`, `CONTEMPORARY_OF`. Allows efficient querying in both directions. |
| **`Field`** | `id`, `name` (Unique) | High-level categories (e.g., 'Physics', 'Literature'). |
| **`Occupation`** | `id`, `name` (Unique) | Detailed, searchable occupations (e.g., 'Theoretical Physicist'). |
| **M2M Join Tables**| (Between `Figure` and `Field`/`Occupation`) | Allows one figure to be associated with multiple categories/roles for flexible filtering. |
