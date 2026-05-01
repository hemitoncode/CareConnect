# Supabase Database Model + Calendar for Engineering Work

## Engineering Timeline (4–6 weeks, 1 engineer)

| Week | Focus | What gets built |
|---|---|---|
| **1** | Foundation | Set up the codebase and design system. Build a working MVP shell deployed to staging. Pull all 1,500 Ontario retirement homes from public data and create a profile for each one. |
| **2** | Browse | Search, filters, map view, and provider profile pages. Families can find homes. |
| **3** | Match | Family sign-up form. Matching algorithm. Personalized home recommendations. |
| **4** | Leads | Providers can claim their profile. Families send inquiries. Email and text alerts. Billing for confirmed referrals. |
| **5** | Trust | Track Record (auto-updated activity log). Verified family reviews. Quality score updates. |
| **6** | Polish | Provider analytics dashboard. One creative feature (OpenDoor Live or deeper matching). Final QA and launch. |

---

## Data Model (Supabase / Postgres)

```mermaid
erDiagram
    PROVIDERS ||--o{ LEADS : receives
    PROVIDERS ||--o{ ACTIVITY_LOG_ENTRIES : has
    PROVIDERS ||--o{ REVIEWS : about
    PROVIDERS ||--o{ LIVE_STREAMS : hosts
    PROVIDERS ||--o{ REFERRALS : earns
    FAMILIES  ||--o{ LEADS : submits
    FAMILIES  ||--o{ REVIEWS : writes
    FAMILIES  ||--o{ REFERRALS : completes
    LEADS     ||--o{ REFERRALS : produces
    REFERRALS ||--o| REVIEWS : enables

    PROVIDERS {
        uuid id PK
        string name
        string type
        string license_number
        string address
        float lat
        float lng
        int capacity
        int price_min
        int price_max
        float rhra_compliance_score
        float native_score
        float review_score
        float evidence_score
        timestamp last_scraped_at
        array amenities
        array languages
        array religious_practices
        array lifestyle_tags
        timestamp claimed_at
        uuid claimed_by_user_id FK
        bool hidden_gem
    }

    FAMILIES {
        uuid id PK
        string contact_name
        string email
        string phone
        string relation_to_resident
        int resident_age
        string mobility_level
        string budget_range
        array preferred_languages
        array cultural_tags
        array hobbies
        string move_timeline
        float location_pref_lat
        int location_pref_radius_km
    }

    LEADS {
        uuid id PK
        uuid family_id FK
        uuid provider_id FK
        string status
        timestamp created_at
        timestamp first_response_at
        bool family_consent_to_contact
    }

    REFERRALS {
        uuid id PK
        uuid lead_id FK
        uuid family_id FK
        uuid provider_id FK
        string status
        timestamp contact_exchanged_at
        timestamp move_in_confirmed_at
        int fee_amount
        string stripe_invoice_id
    }

    ACTIVITY_LOG_ENTRIES {
        uuid id PK
        uuid provider_id FK
        string event_type
        string source
        string evidence_url
        timestamp occurred_at
        bool verified
    }

    REVIEWS {
        uuid id PK
        uuid family_id FK
        uuid provider_id FK
        uuid referral_id FK
        int rating
        string body
        bool verified_resident
        timestamp created_at
    }

    LIVE_STREAMS {
        uuid id PK
        uuid provider_id FK
        timestamp started_at
        timestamp ended_at
        string mux_playback_id
        int viewer_peak
        jsonb blocks
    }
```
