# Supabase Database Model + Calendar for Engineering Work

## Engineering Timeline (6 weeks, 1 product-focused engineer)

| Week | Focus | What gets built |
|---|---|---|
| **1** | Foundation + data pipeline | Set up app shell, design system primitives, Supabase project, auth scaffolding, and baseline schema. Ingest and normalize Ontario retirement home data (name, address, care type, pricing, geo, compliance) into `homes`. Deploy staging. |
| **2** | Browse + trust-first profiles | Build Search tab: list view, filters, fit sorting, Hidden Gems lane. Build Home Profile: amenities, fit context, Track Record preview, OpenDoor session previews. |
| **3** | Onboarding + matching | Build family sign-up/login and separate User vs Loved One records. Implement 3-step intake (care level, budget/funding, amenities, cultural/religious fit + celebrated events, response SLA, tour preference). Ship match scoring + ranked results. |
| **4** | Active Intros + scoped chat | Build intro creation from home profile, Active Intros pipeline states, intro detail timeline, and per-intro chat thread (no global inbox). Add unread markers and notification event hooks. |
| **5** | Tours + profile management | Build schedule tour flow (date/time, in-person/virtual, attendance), confirmation screen, calendar export path, and profile tab with split edit flows (`Edit User` vs `Edit Loved One`). Re-run match scoring when Loved One is edited. |
| **6** | Live trust layer + launch QA | Add OpenDoor Live session model + playback metadata support, complete Track Record UX, harden telemetry, run accessibility/performance QA, bug bash, and launch checklist for production. |

---

## Data Model (Supabase / Postgres)

```mermaid
erDiagram
    USERS ||--o{ LOVED_ONES : cares_for
    USERS ||--|| USER_NOTIFICATION_PREFS : has
    USERS ||--o{ INTROS : initiates

    LOVED_ONES ||--o{ INTROS : context_for
    LOVED_ONES ||--o{ MATCH_SNAPSHOTS : scored_for

    HOMES ||--o{ INTROS : receives
    HOMES ||--o{ HOME_TRACK_RECORD_ENTRIES : has
    HOMES ||--o{ HOME_LIVE_SESSIONS : hosts
    HOMES ||--o{ MATCH_SNAPSHOTS : scored_in

    INTROS ||--o{ INTRO_MESSAGES : contains
    INTROS ||--o| TOUR_BOOKINGS : may_schedule

    HOME_LIVE_SESSIONS ||--o{ HOME_LIVE_QA_ITEMS : includes

    USERS {
        uuid id PK
        string full_name
        string email
        string phone
        timestamp created_at
        timestamp updated_at
    }

    USER_NOTIFICATION_PREFS {
        uuid user_id PK, FK
        bool email_digest
        bool push_notifications
        bool sms_tour_confirmations
        timestamp updated_at
    }

    LOVED_ONES {
        uuid id PK
        uuid user_id FK
        string full_name
        int age
        string relationship_to_user
        string location_label
        float location_lat
        float location_lng
        int search_radius_km
        string care_level
        int budget_min
        int budget_max
        array funding_sources
        array must_have_amenities
        array cultural_religious_fit_tags
        array celebrated_event_preferences
        int acceptable_response_hours
        string tour_preference
        timestamp created_at
        timestamp updated_at
    }

    HOMES {
        uuid id PK
        string name
        string care_type
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
        bool hidden_gem
        bool opendoor_verified
        array amenities
        array languages
        array religious_practices
        array lifestyle_tags
        array heavily_celebrated_events
        timestamp last_scraped_at
        timestamp claimed_at
        uuid claimed_by_user_id
    }

    MATCH_SNAPSHOTS {
        uuid id PK
        uuid loved_one_id FK
        uuid home_id FK
        int fit_percent
        jsonb fit_breakdown
        timestamp generated_at
    }

    INTROS {
        uuid id PK
        uuid user_id FK
        uuid loved_one_id FK
        uuid home_id FK
        string status
        timestamp request_sent_at
        timestamp intro_accepted_at
        timestamp tour_scheduled_at
        timestamp tour_completed_at
        timestamp created_at
        timestamp updated_at
    }

    INTRO_MESSAGES {
        uuid id PK
        uuid intro_id FK
        string sender_role
        string sender_name
        string body
        bool is_system_message
        timestamp sent_at
        timestamp read_at
    }

    TOUR_BOOKINGS {
        uuid id PK
        uuid intro_id FK
        uuid home_id FK
        uuid loved_one_id FK
        timestamp scheduled_for
        string mode
        bool resident_attending
        string reference_code
        string status
        timestamp calendar_exported_at
        timestamp created_at
        timestamp updated_at
    }

    HOME_TRACK_RECORD_ENTRIES {
        uuid id PK
        uuid home_id FK
        string event_type
        string title
        string details
        string source
        string evidence_url
        timestamp occurred_at
        bool verified
        timestamp created_at
    }

    HOME_LIVE_SESSIONS {
        uuid id PK
        uuid home_id FK
        string title
        timestamp started_at
        timestamp ended_at
        string status
        string mux_playback_id
        int viewer_peak
        jsonb blocks
        timestamp created_at
    }

    HOME_LIVE_QA_ITEMS {
        uuid id PK
        uuid live_session_id FK
        string question
        string asked_by
        int answer_duration_sec
        timestamp answered_at
    }
```

---
