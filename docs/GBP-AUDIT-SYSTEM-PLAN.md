# GBP Audit System - Technical Planning Document

**Project:** Automated Google Business Profile Auditing System
**Target Users:** Local Service Companies (HVAC, Plumbing, Electrical)
**Version:** 1.0 Draft
**Created:** 2026-02-03

---

## Executive Summary

This document outlines the plan for building an automated GBP (Google Business Profile) auditing system that evaluates a business's profile against established best practices and generates actionable recommendations.

**Goal:** Take our Master GBP Optimization Guide (prescriptive guidelines) and turn it into a diagnostic tool that can audit any HVAC, Plumbing, or Electrical company's GBP and produce a scored report with prioritized improvements.

---

## Table of Contents

1. [Problem Statement](#1-problem-statement)
2. [System Overview](#2-system-overview)
3. [Data Acquisition Strategy](#3-data-acquisition-strategy)
4. [Audit Criteria & Scoring](#4-audit-criteria--scoring)
5. [Technical Architecture](#5-technical-architecture)
6. [AI/LLM Integration](#6-aillm-integration)
7. [Output & Reporting](#7-output--reporting)
8. [Implementation Phases](#8-implementation-phases)
9. [Technical Challenges & Mitigations](#9-technical-challenges--mitigations)
10. [Cost Considerations](#10-cost-considerations)
11. [Future Enhancements](#11-future-enhancements)

---

## 1. Problem Statement

### Current State
- We have comprehensive GBP optimization guidelines (22 sections, 100+ checkpoints)
- Manual auditing requires an expert to review each profile against guidelines
- Time-consuming: 2-4 hours per thorough audit
- Inconsistent: Different auditors may weight issues differently
- Not scalable: Can't efficiently audit multiple locations or track changes over time

### Desired State
- Automated system that takes a business identifier (name + location or GBP URL)
- Pulls relevant data from available sources
- Scores the profile against our established criteria
- Generates a professional audit report with prioritized recommendations
- Completes in minutes, not hours
- Consistent, repeatable results

---

## 2. System Overview

### High-Level Flow

```
┌─────────────────────────────────────────────────────────────────────┐
│                         INPUT                                        │
│  Business Name + City  OR  Google Maps URL  OR  Place ID            │
└─────────────────────────────────────────────────────────────────────┘
                                   │
                                   ▼
┌─────────────────────────────────────────────────────────────────────┐
│                    DATA ACQUISITION LAYER                            │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐               │
│  │ Google       │  │ Web          │  │ Third-Party  │               │
│  │ Places API   │  │ Scraping     │  │ APIs         │               │
│  └──────────────┘  └──────────────┘  └──────────────┘               │
└─────────────────────────────────────────────────────────────────────┘
                                   │
                                   ▼
┌─────────────────────────────────────────────────────────────────────┐
│                    DATA NORMALIZATION                                │
│  Structured JSON with all available profile data                     │
└─────────────────────────────────────────────────────────────────────┘
                                   │
                                   ▼
┌─────────────────────────────────────────────────────────────────────┐
│                    ANALYSIS ENGINE                                   │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐               │
│  │ Rule-Based   │  │ LLM          │  │ Vision AI    │               │
│  │ Scoring      │  │ Analysis     │  │ (Photos)     │               │
│  └──────────────┘  └──────────────┘  └──────────────┘               │
└─────────────────────────────────────────────────────────────────────┘
                                   │
                                   ▼
┌─────────────────────────────────────────────────────────────────────┐
│                    REPORT GENERATION                                 │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐               │
│  │ HTML         │  │ PDF          │  │ JSON         │               │
│  │ Report       │  │ Export       │  │ Data         │               │
│  └──────────────┘  └──────────────┘  └──────────────┘               │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 3. Data Acquisition Strategy

### 3.1 Available Data Sources

#### Option A: Google Places API (Recommended Primary)

**Endpoint:** `https://maps.googleapis.com/maps/api/place/details/json`

**Available Data:**
| Field | Available | Notes |
|-------|-----------|-------|
| Business Name | ✅ Yes | `name` |
| Address | ✅ Yes | `formatted_address`, `address_components` |
| Phone | ✅ Yes | `formatted_phone_number` |
| Website | ✅ Yes | `website` |
| Hours | ✅ Yes | `opening_hours` |
| Categories | ✅ Partial | `types` (Google's types, not exact GBP categories) |
| Rating | ✅ Yes | `rating` (1-5 scale) |
| Review Count | ✅ Yes | `user_ratings_total` |
| Reviews | ✅ Yes | `reviews` (up to 5 most relevant) |
| Photos | ✅ Yes | `photos` (references, need separate call) |
| Place ID | ✅ Yes | Unique identifier |
| Lat/Long | ✅ Yes | `geometry.location` |

**NOT Available via Places API:**
- ❌ Full review list (only 5)
- ❌ Business description
- ❌ Services menu
- ❌ Products
- ❌ Posts
- ❌ Q&A (discontinued anyway)
- ❌ Attributes (most)
- ❌ Owner responses to reviews
- ❌ Performance/Insights data

**Cost:** $17 per 1,000 requests (Place Details)

#### Option B: Web Scraping (Supplementary)

**Target:** Google Maps listing page / Google Search knowledge panel

**Potentially Scrapeable:**
| Field | Difficulty | Reliability |
|-------|------------|-------------|
| Business Description | Medium | Medium |
| Photo Count | Easy | High |
| Review Count | Easy | High |
| Recent Reviews | Medium | Medium |
| Posts (if visible) | Hard | Low |
| Services | Hard | Low |
| Attributes | Medium | Medium |

**Risks:**
- Against Google TOS
- Can break with UI changes
- May trigger CAPTCHAs
- IP blocking possible

**Recommendation:** Use sparingly, only for data not available via API

#### Option C: Client-Provided Data

**For Premium/Full Audits:**
- Client exports data from GBP dashboard
- Client grants API access (Business Profile API requires ownership)
- Client fills out supplementary form

**Additional Data Available:**
- ✅ Full business description
- ✅ All services with descriptions
- ✅ All posts history
- ✅ Performance metrics (views, clicks, calls)
- ✅ All reviews with responses
- ✅ Photo metadata

#### Option D: Third-Party Tool Integration

**Potential Partners:**
| Tool | Data Available | API | Cost |
|------|----------------|-----|------|
| BrightLocal | Citations, rankings, reviews | Yes | $29+/mo |
| Whitespark | Citations, rankings | Yes | $20+/mo |
| Moz Local | Citations, listings | Yes | $14+/mo |
| Semrush | Listings, analytics | Yes | $120+/mo |

### 3.2 Recommended Data Strategy

**Tier 1 - Basic Audit (Public Data Only)**
```
Primary: Google Places API
Supplementary: Light scraping for description/photos
Cost: ~$0.02 per audit
Data Completeness: 60-70%
```

**Tier 2 - Standard Audit (Enhanced)**
```
Primary: Google Places API
Secondary: Structured scraping
Tertiary: Client questionnaire for missing data
Cost: ~$0.05 per audit
Data Completeness: 80-85%
```

**Tier 3 - Premium Audit (Full Access)**
```
Primary: Client-provided GBP export/access
Secondary: Google Places API for public view
Tertiary: Competitor analysis (3-5 competitors)
Cost: ~$0.10 per audit + client time
Data Completeness: 95-100%
```

---

## 4. Audit Criteria & Scoring

### 4.1 Scoring Framework

**Total Score: 100 points**

```
GBP HEALTH SCORE BREAKDOWN
══════════════════════════════════════════════════════════

PROFILE COMPLETENESS (20 points)
├── Business Name (correct, no keyword stuffing)     2 pts
├── Address/SAB Configuration                        3 pts
├── Phone Number (local area code)                   2 pts
├── Website (present and working)                    3 pts
├── Business Hours (complete)                        3 pts
├── Primary Category (specific)                      3 pts
├── Additional Categories (4-5 optimal)              2 pts
└── Attributes (5+ set)                              2 pts

VISUAL CONTENT (15 points)
├── Photo Count
│   ├── 0-8 photos                                   0 pts
│   ├── 9-24 photos                                  3 pts
│   ├── 25-49 photos                                 5 pts
│   └── 50+ photos                                   7 pts
├── Photo Quality (AI assessed)                      4 pts
├── Photo Variety (team, work, location)             2 pts
└── Video Present                                    2 pts

REVIEWS (25 points) - Highest Weight
├── Review Count
│   ├── 0-9 reviews                                  0 pts
│   ├── 10-24 reviews                                3 pts
│   ├── 25-49 reviews                                5 pts
│   ├── 50-99 reviews                                7 pts
│   └── 100+ reviews                                 9 pts
├── Average Rating
│   ├── Below 4.0                                    0 pts
│   ├── 4.0-4.4                                      3 pts
│   ├── 4.5-4.7                                      5 pts
│   └── 4.8-5.0                                      6 pts
├── Review Recency (reviews in last 90 days)         4 pts
├── Response Rate (% of reviews with response)       3 pts
└── Response Quality (AI assessed)                   3 pts

CONTENT & ENGAGEMENT (15 points)
├── Business Description
│   ├── Missing                                      0 pts
│   ├── Present but short (<300 chars)               2 pts
│   ├── Good (300-600 chars)                         4 pts
│   └── Optimal (600-750 chars with keywords)        5 pts
├── Services Menu
│   ├── No services                                  0 pts
│   ├── 1-4 services                                 2 pts
│   ├── 5-10 services                                4 pts
│   └── 10+ services (diminishing returns)           3 pts
├── Google Posts
│   ├── No posts / posts older than 6 months         0 pts
│   ├── Posts within 6 months                        2 pts
│   ├── Posts within 30 days                         4 pts
│   └── Posts within 7 days                          6 pts
└── (Reserved for future features)                   0 pts

LOCAL SEO SIGNALS (15 points)
├── NAP Consistency (spot check top citations)       5 pts
├── Website Link Valid & Fast                        3 pts
├── SSL Certificate Present                          2 pts
├── Mobile-Friendly Website                          3 pts
└── Local Content on Website                         2 pts

ADVANCED FEATURES (10 points)
├── Google Verified Badge                            4 pts
├── Booking/Appointment Integration                  2 pts
├── Messaging Enabled (WhatsApp)                     2 pts
└── Products/Services with Pricing                   2 pts

══════════════════════════════════════════════════════════
TOTAL                                              100 pts
```

### 4.2 Score Interpretation

| Score Range | Grade | Interpretation |
|-------------|-------|----------------|
| 90-100 | A+ | Excellent - Minor optimizations only |
| 80-89 | A | Strong - Few improvements needed |
| 70-79 | B | Good - Some important gaps |
| 60-69 | C | Fair - Multiple areas need work |
| 50-59 | D | Poor - Significant improvements required |
| 0-49 | F | Critical - Major overhaul needed |

### 4.3 Issue Severity Levels

```
🔴 CRITICAL - Immediate action required
   - Profile not verified
   - Wrong business category
   - No phone number
   - No reviews (0)
   - Suspended or disabled listing

🟠 HIGH - Should fix within 1 week
   - Below 4.0 rating
   - No response to negative reviews
   - Fewer than 10 reviews
   - No photos
   - Outdated hours

🟡 MEDIUM - Should fix within 1 month
   - Missing business description
   - No services listed
   - No posts in 6+ months
   - Fewer than 25 photos
   - NAP inconsistencies

🟢 LOW - Nice to have improvements
   - Could add more categories
   - Could add more photos
   - Post frequency could increase
   - Could add booking integration
```

---

## 5. Technical Architecture

### 5.1 Technology Stack (Recommended)

```
BACKEND
├── Runtime: Node.js 20+ (or Python 3.11+)
├── Framework: Express.js / FastAPI
├── Database: PostgreSQL (audits, history)
├── Cache: Redis (API responses)
└── Queue: Bull/BullMQ (job processing)

AI/ML
├── LLM: Claude API (Haiku for cost, Sonnet for quality)
├── Vision: Claude Vision or Google Vision API
└── Embeddings: OpenAI or local model

APIS
├── Google Places API
├── Google PageSpeed API (website checks)
├── Screenshot API (visual capture)
└── (Optional) BrightLocal/Whitespark API

FRONTEND (if needed)
├── Framework: Next.js or simple HTML
├── Styling: Tailwind CSS
└── PDF Generation: Puppeteer or WeasyPrint

INFRASTRUCTURE
├── Hosting: Hetzner VPS / AWS / Vercel
├── CI/CD: GitHub Actions
└── Monitoring: Simple logging to start
```

### 5.2 Database Schema

```sql
-- Core Tables

CREATE TABLE audits (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    created_at TIMESTAMP DEFAULT NOW(),

    -- Input
    business_name VARCHAR(255),
    location VARCHAR(255),
    place_id VARCHAR(255),
    gbp_url TEXT,

    -- Scores
    total_score INTEGER,
    profile_score INTEGER,
    visual_score INTEGER,
    review_score INTEGER,
    content_score INTEGER,
    seo_score INTEGER,
    advanced_score INTEGER,
    grade CHAR(2),

    -- Raw Data
    raw_data JSONB,
    analysis_data JSONB,

    -- Metadata
    audit_type VARCHAR(50), -- basic, standard, premium
    status VARCHAR(50), -- pending, processing, completed, failed
    processing_time_ms INTEGER,
    cost_cents INTEGER
);

CREATE TABLE audit_issues (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    audit_id UUID REFERENCES audits(id),

    category VARCHAR(100),
    severity VARCHAR(20), -- critical, high, medium, low
    title VARCHAR(255),
    description TEXT,
    recommendation TEXT,
    current_value TEXT,
    target_value TEXT,

    points_lost INTEGER,
    sort_order INTEGER
);

CREATE TABLE audit_competitors (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    audit_id UUID REFERENCES audits(id),

    place_id VARCHAR(255),
    business_name VARCHAR(255),
    rating DECIMAL(2,1),
    review_count INTEGER,
    -- Additional comparison fields
    raw_data JSONB
);
```

### 5.3 API Endpoints

```
POST /api/v1/audit
  Input: { businessName, location } or { placeId } or { gbpUrl }
  Output: { auditId, status: "processing" }

GET /api/v1/audit/:auditId
  Output: { audit data, scores, issues, recommendations }

GET /api/v1/audit/:auditId/report
  Query: ?format=html|pdf|json
  Output: Formatted report

POST /api/v1/audit/:auditId/competitors
  Input: { competitors: [{ name, location }] }
  Output: { comparison data }
```

---

## 6. AI/LLM Integration

### 6.1 LLM Use Cases

| Use Case | Model | Input | Output |
|----------|-------|-------|--------|
| Description Analysis | Haiku | Business description text | Quality score, keyword analysis, suggestions |
| Review Sentiment | Haiku | Review texts | Sentiment breakdown, key themes, concerns |
| Response Quality | Haiku | Owner responses | Tone analysis, improvement suggestions |
| Recommendations | Sonnet | Full audit data | Prioritized action items, custom advice |
| Report Narrative | Sonnet | Scores + issues | Executive summary, section narratives |

### 6.2 Prompt Templates

**Business Description Analysis:**
```
Analyze this Google Business Profile description for a {industry} company:

"{description}"

Evaluate:
1. Length (optimal: 600-750 characters)
2. Keyword usage for local SEO
3. Unique value proposition clarity
4. Call-to-action presence
5. Professional tone

Return JSON:
{
  "score": 1-10,
  "length_assessment": "...",
  "keywords_found": [...],
  "keywords_missing": [...],
  "strengths": [...],
  "improvements": [...]
}
```

**Review Analysis:**
```
Analyze these customer reviews for a {industry} company:

{reviews_json}

Provide:
1. Overall sentiment (positive/neutral/negative breakdown)
2. Common praise themes
3. Common complaint themes
4. Urgent issues mentioned
5. Response quality assessment (if responses included)

Return JSON with structured analysis.
```

### 6.3 Vision AI Use Cases

**Photo Quality Assessment:**
```
Analyze these Google Business Profile photos for a {industry} company:

Evaluate each photo for:
1. Relevance to business (team, work, equipment, location)
2. Image quality (lighting, focus, resolution)
3. Professionalism
4. Customer appeal

Identify:
- Missing photo types (no team photos, no work examples, etc.)
- Low quality photos that should be replaced
- Good photos that showcase the business well
```

---

## 7. Output & Reporting

### 7.1 Report Sections

```
GBP AUDIT REPORT
════════════════════════════════════════

1. EXECUTIVE SUMMARY
   - Overall Score & Grade
   - Key Strengths (top 3)
   - Critical Issues (top 3)
   - Quick Wins (easy improvements)

2. SCORE BREAKDOWN
   - Visual score card
   - Category-by-category scores
   - Comparison to industry average (if available)

3. DETAILED FINDINGS

   3.1 Profile Completeness
       - Current state
       - Issues found
       - Recommendations

   3.2 Visual Content
       - Photo analysis
       - Video assessment
       - Recommendations

   3.3 Reviews & Reputation
       - Review metrics
       - Sentiment analysis
       - Response audit
       - Recommendations

   3.4 Content & Engagement
       - Description analysis
       - Services review
       - Posts audit
       - Recommendations

   3.5 Local SEO Signals
       - NAP check results
       - Website assessment
       - Recommendations

   3.6 Advanced Features
       - Current status
       - Opportunities

4. PRIORITIZED ACTION PLAN
   - Week 1: Critical fixes
   - Week 2-3: High priority
   - Month 1: Medium priority
   - Ongoing: Maintenance

5. COMPETITOR COMPARISON (if included)
   - Side-by-side metrics
   - Competitive gaps
   - Opportunities

6. APPENDIX
   - Raw data
   - Methodology
   - Glossary
```

### 7.2 Report Formats

**HTML Report:**
- Interactive, expandable sections
- Visual charts (Chart.js or similar)
- Click-to-expand details
- Mobile responsive
- Shareable link

**PDF Report:**
- Professional, print-ready
- Client logo placement
- Page numbers, headers
- 10-20 pages typical

**JSON Export:**
- Raw data for integration
- Webhook delivery option
- API response format

---

## 8. Implementation Phases

### Phase 1: MVP (2-3 weeks)

**Goal:** Working audit with core scoring, basic report

**Deliverables:**
- [ ] Google Places API integration
- [ ] Basic scoring engine (profile, reviews, photos count)
- [ ] Simple HTML report generation
- [ ] Command-line interface

**Scope:**
- Public data only
- No AI analysis yet (rule-based only)
- Single business audit
- HTML output only

### Phase 2: AI Enhancement (2-3 weeks)

**Goal:** Add LLM analysis for qualitative assessment

**Deliverables:**
- [ ] Claude API integration
- [ ] Description analysis
- [ ] Review sentiment analysis
- [ ] AI-generated recommendations
- [ ] Enhanced report with narratives

### Phase 3: Visual & Advanced (2 weeks)

**Goal:** Photo analysis, advanced features check

**Deliverables:**
- [ ] Vision AI for photo assessment
- [ ] Website speed/mobile check
- [ ] NAP consistency spot check
- [ ] PDF report generation

### Phase 4: Competitor & Scale (2 weeks)

**Goal:** Comparison features, multi-audit capability

**Deliverables:**
- [ ] Competitor pull and comparison
- [ ] Batch audit capability
- [ ] Database storage for history
- [ ] Basic web interface

### Phase 5: Productization (Ongoing)

**Goal:** Client-ready product

**Deliverables:**
- [ ] White-label report options
- [ ] Client portal
- [ ] Scheduling/recurring audits
- [ ] API for integrations
- [ ] Billing/usage tracking

---

## 9. Technical Challenges & Mitigations

| Challenge | Risk | Mitigation |
|-----------|------|------------|
| **Google API limits** | Medium | Implement caching, rate limiting, consider enterprise tier |
| **Data completeness** | High | Clear communication about what's included in each tier |
| **Scraping reliability** | High | Minimize scraping, use as fallback only, maintain selectors |
| **LLM costs** | Medium | Use Haiku for most tasks, cache responses, batch processing |
| **LLM consistency** | Medium | Structured prompts, JSON output, validation |
| **Photo analysis cost** | Medium | Sample photos (not all), cache results |
| **Report generation time** | Low | Async processing, job queue, progress indicators |
| **Accuracy of scoring** | Medium | Continuous calibration, human review of edge cases |

---

## 10. Cost Considerations

### Per-Audit Cost Estimate

**Basic Audit (MVP):**
```
Google Places API (1 call)      $0.017
LLM - Description (Haiku)       $0.001
LLM - Recommendations (Haiku)   $0.002
───────────────────────────────────────
Total                           ~$0.02
```

**Standard Audit:**
```
Google Places API (1 call)      $0.017
Google Photos API (10 photos)   $0.070
LLM - Multiple analyses         $0.010
Vision AI (5 photos)            $0.010
PageSpeed API                   Free
───────────────────────────────────────
Total                           ~$0.10
```

**Premium Audit (with competitors):**
```
Google Places API (6 calls)     $0.102
Google Photos API (30 photos)   $0.210
LLM - Full analysis             $0.050
Vision AI (15 photos)           $0.030
PageSpeed API                   Free
───────────────────────────────────────
Total                           ~$0.40
```

### Monthly Cost Projections

| Volume | Basic | Standard | Premium |
|--------|-------|----------|---------|
| 100 audits | $2 | $10 | $40 |
| 500 audits | $10 | $50 | $200 |
| 1,000 audits | $20 | $100 | $400 |

*Note: Does not include infrastructure, development, or support costs*

---

## 11. Future Enhancements

### Near-Term (3-6 months)
- [ ] Recurring audit scheduling (weekly/monthly)
- [ ] Change tracking and alerts
- [ ] Industry benchmarking database
- [ ] Multi-location support
- [ ] White-label partner program

### Medium-Term (6-12 months)
- [ ] Direct GBP integration (for clients who grant access)
- [ ] Automated fix suggestions with one-click implementation
- [ ] Review response AI assistant
- [ ] Post generation AI assistant
- [ ] Citation building integration

### Long-Term (12+ months)
- [ ] Full local SEO audit (beyond just GBP)
- [ ] Rank tracking integration
- [ ] ROI tracking (if client provides revenue data)
- [ ] Predictive scoring ("if you do X, score will increase by Y")
- [ ] Agency dashboard for managing multiple clients

---

## Appendix A: Reference Documents

- [Master GBP Optimization Guide](./master-gbp-optimization-guide-UPDATED.html)
- [Corrections & Updates Log](./MASTER-GUIDELINE-CORRECTIONS.md)
- [Google Places API Documentation](https://developers.google.com/maps/documentation/places/web-service/details)
- [Google Business Profile API](https://developers.google.com/my-business/reference/rest)

---

## Appendix B: Sample Audit Output Structure

```json
{
  "audit": {
    "id": "uuid",
    "created_at": "2026-02-03T12:00:00Z",
    "business": {
      "name": "ABC Plumbing",
      "place_id": "ChIJ...",
      "address": "123 Main St, Dallas, TX 75001",
      "phone": "(214) 555-1234",
      "website": "https://abcplumbing.com",
      "category": "Plumber"
    },
    "scores": {
      "total": 72,
      "grade": "B",
      "breakdown": {
        "profile": { "score": 16, "max": 20 },
        "visual": { "score": 10, "max": 15 },
        "reviews": { "score": 18, "max": 25 },
        "content": { "score": 9, "max": 15 },
        "seo": { "score": 12, "max": 15 },
        "advanced": { "score": 7, "max": 10 }
      }
    },
    "issues": [
      {
        "severity": "high",
        "category": "reviews",
        "title": "Low response rate to reviews",
        "description": "Only 23% of reviews have owner responses",
        "recommendation": "Respond to all reviews within 24-48 hours",
        "points_lost": 3
      }
    ],
    "recommendations": {
      "quick_wins": [...],
      "this_week": [...],
      "this_month": [...],
      "ongoing": [...]
    }
  }
}
```

---

*Document Version: 1.0 | Last Updated: 2026-02-03*
