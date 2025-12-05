# Taoist Data Studio - Data Modeling Reference

**Overview:** This document provides a comprehensive reference for all SQL views and data models in our data warehouse architecture. These models power our business intelligence dashboards in Google Looker Studio, enabling data-driven decision making across advertising, customer funnel, and revenue analytics.

---

## Table of Contents

1. [Ads Layer](#ads-layer)
2. [Funnel Layer](#funnel-layer)
3. [Revenue Layer](#revenue-layer)

---

## Ads Layer

**Business Purpose:** Track advertising performance across platforms and attribute customer journeys back to ad campaigns. Enables ROI analysis, campaign optimization, and customer acquisition cost (CAC) calculations.

**Data Flow:** Platform-specific data → Unified view → Customer attribution & campaign analysis

### Layer 01: Platform-Specific Ad Performance

**Purpose:** Raw daily performance metrics from individual advertising platforms.

| View | Description |
|------|-------------|
| `ads_google_ad_daily.sql` | Daily Google Ads campaign performance metrics (spend, impressions, clicks, conversions) |
| `ads_meta_ad_daily.sql` | Daily Meta/Facebook Ads performance metrics (spend, impressions, clicks, conversions) |

### Layer 02: Unified Advertising View

**Purpose:** Combine multiple ad platforms into a single standardized view for cross-platform analysis.

| View | Description |
|------|-------------|
| `ads_perf_daily.sql` | Unified daily ad performance combining Google Ads and Meta Ads with standardized metrics |

### Layer 03: Customer Attribution & Analysis

**Purpose:** Link customer transactions and revenue back to advertising campaigns via UTM parameters. Enables full-funnel analysis from ad spend to revenue.

| View | Description |
|------|-------------|
| `ads_customer_details.sql` | Customer-level details with advertising attribution, linking orders back to ad campaigns via UTM parameters |
| `ads_product_sales.sql` | Product sales summary with advertising attribution |
| `ads_campaign_dashboard_V4_subscriber_refined.sql` | Comprehensive campaign performance dashboard tracking the full funnel: **Ad Spend → Trials → Students → Revenue** |

**Key Business Questions Answered:**
- Which campaigns drive the most valuable customers?
- What's the return on ad spend (ROAS) by campaign?
- How do different ad platforms compare in customer acquisition?
- What's the customer acquisition cost (CAC) by campaign?

---

## Funnel Layer

**Business Purpose:** Track the complete customer journey through the subscription lifecycle from trial to recurring customer. Enables conversion rate analysis, churn tracking, and cohort-based retention metrics.

**Data Source:** Samcart subscription and transaction data

### Core Funnel Views

| View | Description |
|------|-------------|
| `samcart_fct_funnel_events_V3_New_Student_Validated.sql` | Comprehensive funnel events tracking all customer lifecycle stages |
| `samcart_fct_funnel_charges.sql` | Charge-level records for New Student and Recurring Student charges |

### Customer Lifecycle Stages Tracked

1. **Trial Students** - Customers who started a 7-day free trial
2. **Paid Starters** - Customers who paid upfront without a trial
3. **New Students** - First charge after trial or paid starter (charge_count = 1)
4. **Recurring Students** - Multiple charges (charge_count > 1)
5. **Churned Students** - Customers who canceled their subscription
6. **Failed Charges** - Payment attempts that failed
7. **Refunds** - Refunded transactions
8. **One-off Purchases** - Non-subscription purchases

### Key Metrics & Analysis

**Charge Tracking:**
- Charge sequence tracking (1st, 2nd, 3rd charge, etc.)
- Subscription type classification (Trial-Based vs Paid Starter)
- Charge dates and amounts
- Cohort month tracking for retention analysis

**Key Business Questions Answered:**
- How many customers are in each stage of the funnel?
- What's the conversion rate from trial to paid student?
- How many customers churn vs. become recurring?
- What's the customer lifetime value by cohort?
- What's the retention rate over time?
- What percentage of customers start with trials vs. paid upfront?

---

## Revenue Layer

**Business Purpose:** Unified view of all transactional and revenue data across payment platforms. Enables revenue reporting, MRR/ARR calculations, and customer segmentation by payment status.

**Data Sources:** Stripe and PayPal payment platforms

### Core Revenue Views

#### revenue_fct_transactions

**Purpose:** Unified fact table combining transactional and revenue data from Stripe and PayPal payment sources.

**Data Sources:**
- **Stripe:** invoices, subscriptions, customers
- **PayPal:** transactions, subscriptions, plans

**Key Features:**
- Combines paid transactions from both Stripe and PayPal
- Includes trial creation events (both Stripe and PayPal)
- Standardizes transaction data across payment platforms
- Calculates charge counts for customer segmentation
- Categorizes customers as: Trial Student, New Student, or Recurring Student

**Output Columns:**

| Column | Description |
|--------|-------------|
| `source` | Payment platform (Stripe or PayPal) |
| `transaction_id` | Unique transaction identifier |
| `source_customer_id` | Customer ID from source system |
| `customer_email` | Customer email address |
| `created_at` | Transaction timestamp |
| `amount` | Transaction amount |
| `currency` | Currency code |
| `status` | Transaction status |
| `source_subscription_id` | Subscription ID from source |
| `product_name` | Product/plan name |
| `transaction_type` | Type of transaction (charge, trial_created) |
| `charge_count` | Sequential charge number for paying customers |
| `plan_tenure_clean` | Human-readable plan tenure (Monthly, Annual, Other) |
| `customer_status` | Customer classification (Trial Student, New Student, Recurring Student) |

**Business Logic:**

| Customer Status | Definition |
|----------------|------------|
| **Trial Student** | Transactions with amount = 0 (trial creation events) |
| **New Student** | First charge (charge_count = 1) with amount > 0 |
| **Recurring Student** | Subsequent charges (charge_count > 1) with amount > 0 |

**Use Cases:**
- Revenue reporting and analysis
- Customer lifecycle tracking
- Subscription revenue calculations (MRR, ARR)
- Customer segmentation by payment status
- Cross-platform revenue aggregation
- Revenue attribution to customer segments

---

## Data Architecture Overview

```
┌─────────────────────────────────────────────────────────────┐
│                    Data Sources                              │
├─────────────────────────────────────────────────────────────┤
│  Stripe  │  PayPal  │  Samcart  │  Google Ads  │  Meta Ads  │
└────┬──────────┬──────────┬──────────┬──────────────┬─────────┘
     │          │          │          │              │
     └──────────┴──────────┴──────────┴──────────────┘
                    │
        ┌───────────┴───────────┐
        │                       │
   ┌────▼────┐            ┌────▼────┐
   │ Revenue │            │  Funnel │
   │  Layer  │            │  Layer  │
   └────┬────┘            └────┬────┘
        │                     │
        └──────────┬───────────┘
                   │
            ┌──────▼──────┐
            │  Ads Layer  │
            │ (Attribution)│
            └──────┬──────┘
                   │
            ┌──────▼──────┐
            │   Looker    │
            │   Studio    │
            │  Dashboards │
            └─────────────┘
```

---

## How to Use This Document

1. **For Business Stakeholders:** Use this document to understand what data is available and what business questions each view can answer.

2. **For Data Analysts:** Reference the column descriptions and business logic sections to understand how to query each view correctly.

3. **For Developers:** Use the data architecture overview and data flow descriptions to understand dependencies and relationships between views.

4. **For New Team Members:** Start with the "Business Purpose" section of each layer to understand the high-level goals, then dive into specific views as needed.
