# User Journey Optimization: Reducing Drop-Offs & Boosting Conversions for Fintech Lending Product

## 1. Business Problem & Impact
**Context:** The company’s flagship product, "InstaLoan," provides instant credit up to $5,000 via a mobile app. While marketing campaigns successfully drive high Top-of-Funnel (ToF) traffic (50k+ monthly installs), the Application-to-Disbursal conversion rate is stagnant at **12%**, significantly below the target of **18%**.

**The Business Cost:**

- **CAC Inefficiency:** With a Customer Acquisition Cost (CAC) of $40 per install, every non-converting user represents wasted marketing spend.
- **Operational Drag:** Users who drop off during manual verification stages waste expensive vendor API calls (KYC/Credit Bureau pulls) without generating revenue.
- **Revenue Leakage:** An estimated **$2.4M in annualized revenue opportunity** is lost due to friction in the funnel.

## 2. Data & Tools Used
I utilized a **90-day rolling window of clickstream data** joined with backend user tables.

**Schema Design:**

- **`events_stream`:** `user_id`, `event_name` (e.g., `screen_view_kyc`), `timestamp`, `platform` (iOS/Android), `meta_data` (error_codes, load_time_ms).
- **`users_master`:** `user_id`, `risk_segment`, `income_bracket`, `credit_score_bucket`, `acquisition_channel`.
- **`loan_applications`:** `application_id`, `requested_amount`, `approved_amount`, `apr`, `status`.

### Loan Funnel Steps
| Step | Description | Key Event |
|------|-------------|-----------|
| 1. Landing Screen | User clicks “Apply for Loan” | `loan_cta_clicked` |
| 2. Eligibility Check | User provides basic info (income, employment, city) | `eligibility_passed` / `eligibility_failed` |
| 3. KYC Completion | User uploads PAN, Aadhaar, selfie | `kyc_completed` |
| 4. Credit Assessment | Backend credit and risk checks | `credit_approved` / `credit_rejected` |
| 5. Offer Acceptance | User reviews interest rate & tenure | `offer_accepted` |
| 6. Loan Disbursal | Amount credited to bank account | `loan_disbursed` |

Each step had clear success and failure events, allowing accurate funnel tracking.

## 3. Key Metrics & KPIs
- **Global Conversion Rate (GCR):** Total Disbursals / Total Installs
- **Step-wise Drop-off Rate:** (Users entering Step N - Users entering Step N+1) / Users entering Step N
- **Time-to-Decision:** Average time from `signup_complete` to `offer_generated`.
- **KYC Success Rate:** % of users passing document verification on the first attempt.
- **CAC per Disbursed Loan:** Total Marketing Spend / Total Loans Disbursed

## 4. Data Preparation (Python/Pandas)
- **Sessionization:** Used Pandas to sort events by `user_id` and `timestamp` to reconstruct the sequence of actions.
- **Flagging "Looping" Behavior:** Created a custom feature to count how many times a user attempted the same action (e.g., `upload_attempts > 3`).
- **Time-Delta Calculation:** Calculated the duration between steps (e.g., `Time_Entering_KYC` vs `Time_Completing_KYC`) to measure friction, not just binary conversion.

## 5. Analysis & Insights
### A. Funnel Leakage Analysis
Using Python to visualize the funnel, I identified two critical **"Red Zones"** where **65%** of the total drop-off occurred.

1. **The KYC Cliff (Step 3):** 42% drop-off.
   - **Observation:** Users were abandoning the app specifically during the "Selfie" and "ID Upload" stage.
2. **The Offer Shock (Step 5):** 28% drop-off.
   - **Observation:** Users viewed the offer but closed the app without accepting.

### B. Segmentation Deep Dive
I segmented the KYC drop-off data by Device and OS:

- **iOS Users:** 15% drop-off at KYC.
- **Android Users:** 55% drop-off at KYC.

**Insight:** The disparity suggested a technical friction point specific to Android devices, likely related to camera permissions or image compression on low-end devices.

### C. Behavioral Analysis (Offer Stage)
For users dropping off at the "Offer Screen," I correlated the drop-off with the Interest Rate (APR) offered.

- **APR < 15%:** 70% Acceptance Rate.
- **APR > 25%:** 12% Acceptance Rate.

**Insight:** High-risk users were being approved but served high interest rates that caused immediate churn. The product was technically "working" (approving users), but commercially failing (users finding the price uncompetitive).

## 6. Root Cause Analysis
**Root Cause 1: Android Camera Optimization (Technical)**
- **Data Evidence:** `error_logs` showed a spike in `camera_permission_denied` and `image_blur_error` events specifically for Android versions < 10.
- **Business Reality:** The strict liveness detection SDK was too heavy for older Android phones, causing the app to crash or lag, leading to user frustration.

**Root Cause 2: Income Verification Friction (UX)**
- **Data Evidence:** Median time spent on "Bank Statement Upload" was **12 minutes**.
- **Business Reality:** Users were asked to upload PDFs manually. Most users are mobile-first and do not have PDF bank statements saved on their phones.

## 7. Recommendations & Experiments
Based on the data, I proposed three initiatives prioritized by **ICE (Impact, Confidence, Ease).**

| Experiment | Hypothesis | Action | Success Metric |
|------------|------------|--------|----------------|
| **1. Account Aggregator (AA) Integration (High Impact)** | Replacing manual PDF upload with an Account Aggregator (API-based bank login) will reduce friction. | Integrate a vendor for instant bank linking. | Increase Step 3 conversion from **58% to 70%**. |
| **2. Dynamic "Smart" KYC (Technical Fix)** | For Android users on low-end devices, switching from "Video Liveness" to "Passive Still Image" liveness detection will reduce bandwidth requirements and improve success. | Implement device/OS-specific KYC flow logic. | Reduce Android KYC error rate by **50%**. |
| **3. Pre-Qualification Slider (Product)** | Giving users control over loan amount and tenure will increase perceived value and acceptance. | Introduce a slider allowing users to trade off Loan Amount for Tenure on the offer screen. | Increase Offer Acceptance Rate by **10%**. |
