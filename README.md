# Taoist-Data-Studio-Data-Modelling
All relevant SQL form the data architecture of our data warehouse and how that visualized our intelligence dashboard in Google Looker Studio

# Data Studio SQL Folders Overview

## Ads Layer

**Purpose:** Tracks advertising performance and attributes customer journeys to ad campaigns coming through Samcart.

### Layer 01: Platform-Specific Ad Performance
- **ads_google_ad_daily.sql** - Daily Google Ads campaign performance metrics
- **ads_meta_ad_daily.sql** - Daily Meta/Facebook Ads performance metrics

### Layer 02: Unified Advertising View
- **ads_perf_daily.sql** - Combines Google Ads and Meta Ads into a single unified view

### Layer 03: Customer Attribution & Analysis
- **ads_customer_details.sql** - Customer-level details with advertising attribution, linking orders back to ad campaigns via UTM parameters
- **ads_product_sales.sql** - Product sales summary
- **ads_campaign_dashboard_V4_subscriber_refined** - Campaign performance dashboard tracking the full funnel: ad spend → trials → students → revenue




## Funnel Lyer

**Purpose:** Pure Samcart funnel analysis tracking the complete customer journey through the subscription lifecycle.

### Core Funnel Views
- **samcart_fct_funnel_events_V3_New_Student_Validated** - Comprehensive funnel events tracking:
  - Trial Students (7-day trial starts)
  - Paid Starters (paid upfront, no trial)
  - New Students (first charge after trial or paid starter)
  - Recurring Students (multiple charges)
  - Churned Students
  - Failed charges and refunds
  - One-off purchases

- **samcart_fct_funnel_charges.sql** - Charge-level records for New Student and Recurring Student charges, including:
  - Charge sequence tracking
  - Subscription type (Trial-Based vs Paid Starter)
  - Charge dates and amounts
  - Cohort month tracking

**Key Concept:** This folder answers questions like:
- How many customers are in each stage of the funnel?
- What's the conversion rate from trial to paid student?
- How many customers churn vs. become recurring?
- What's the customer lifetime value by cohort?
- What's the retention rate over time?
