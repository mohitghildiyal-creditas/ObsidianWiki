# Database Schema Reference
*Compiled: 2026-04-08 02:30* `schema` `reference`

> Auto-compiled from context.yaml. Do not edit manually.

## Tables

### CED_CampaignCreationDetails
**Role:** Campaign master — one row per campaign scheduled

| Column | Meaning |
|--------|----------|
| `ChannelIndex` | SMS / EMAIL / WHATSAPP |
| `CampaignIdIndex` | Numeric campaign ID (joins to response tables) |
| `TemplateIdIndex` | Template used |
| `CampaignTitle` | Human-readable campaign name |
| `ScheduleDateIndex` | Date campaign was sent |
| `ActiveIndex` | 1 = live campaign |
| `TestCampaignIndex` | 1 = test (exclude from analysis) |
| `ProjectIdIndex` | Project identifier for CC Upgrade |
| `SegmentType` | Target segment (ETB, NTB, etc.) |

### CED_SMSResponse_Intermediate
**Role:** SMS delivery status — one row per customer per SMS send

| Column | Meaning |
|--------|----------|
| `CampaignIdIndex` | Joins to CED_CampaignCreationDetails.CampaignIdIndex |
| `AccountIdIndex` | Encrypted customer account ID |
| `UuidIndex` | Session UUID — JOINS to CED_CCU_UserJourneyState.source_id |
| `StatusIndex` | Delivered = message reached handset. HYP_MTD_LastFiveFail = last 5 sends all failed. HYP_ThirtyDays_LastThreeFail = 3 fails in 30 days. |
| `TestCampaignIndex` | Exclude where = 1 |
| `UpdateDate` | Last status update timestamp |

### CED_EMAILResponse_Intermediate
**Role:** Email delivery and engagement status

| Column | Meaning |
|--------|----------|
| `CampaignIdIndex` | Campaign reference |
| `AccountIdIndex` | Customer account |
| `UuidIndex` | Session UUID (same join as SMS) |
| `Status` | Delivered = inbox delivery. OPENED = email opened (counts as Open/Read). CLICKED = link clicked in email (counts as Open/Read). |
| `EventType` | Granular event (OPEN, CLICK, BOUNCE, UNSUBSCRIBE) |
| `TestCampaign` | Exclude where = 1 |

### CED_WhatsAppResponse_Intermediate
**Role:** WhatsApp delivery and read status

| Column | Meaning |
|--------|----------|
| `CampaignIdIndex` | Campaign reference |
| `AccountIdIndex` | Customer account |
| `UuidIndex` | Session UUID |
| `StatusIndex` | DELIVERED = message delivered to device. READ = customer opened the WhatsApp message. Both DELIVERED and READ count as IsDelivered = 1. |
| `TestCampaign` | Exclude where = 1 |

### CED_CCU_UserJourneyState
**Role:** Customer session/attempt — THE CENTRAL HUB TABLE. One row per unique upgrade attempt session. source_id = UUID from response tables (the link between campaign and journey). ridIndex = session ID used by all downstream tables.


| Column | Meaning |
|--------|----------|
| `ridIndex` | Unique session ID — PRIMARY JOIN KEY for all funnel tables |
| `source_id` | UUID from response tables — link from response to journey |
| `source` | campaign = triggered from campaign link | organic = direct app entry |
| `project_type` | Always 'crd_upgd' for this product |
| `udf1` | Customer account_id (encrypted) — joins to eth_crd_upgd_booking.account_id |
| `udf2` | Encrypted mobile number |
| `final_state` | Terminal state of the journey (success/failure/abandoned) |
| `active` | 1 = session still active |
| `creation_time` | When customer clicked the upgrade link (IST = UTC+5:30) |

> **Note:** Use creation_time + INTERVAL 330 MINUTE to convert UTC to IST

### eth_crd_upgd_attempt_log
**Role:** Stage-by-stage log of the upgrade flow — one row per stage attempt

| Column | Meaning |
|--------|----------|
| `ridIndex` | Joins to CED_CCU_UserJourneyState.ridIndex |
| `stage` | INITIALIZED → SELECTION → CARD (or MOBILE) → OTP → CARD_DETAIL → OTP_CONSENT → (BOOKING happens in booking table) |
| `status` | success / failure |
| `creation_time` | When this stage was attempted |

> **Note:** A single ridIndex can have multiple rows (retries per stage). Use MAX(IF(stage='X' AND status='success',1,0)) to get stage completion flag.


### eth_ccu_event_tracking
**Role:** Granular behavioral events — button clicks, screen views

| Column | Meaning |
|--------|----------|
| `rid` | Joins to CED_CCU_UserJourneyState.ridIndex |
| `action` | Event action (e.g. card_upgrade_carousal_on_1_card) |
| `category` | Same event in category field (redundant with action — check both) |
| `ct` | Event timestamp (IST) |

### eth_crd_upgd_otp
**Role:** OTP generation and validation events

| Column | Meaning |
|--------|----------|
| `ref_idIndex` | Joins to CED_CCU_UserJourneyState.ridIndex |
| `workflow` | prelogin = login OTP | postlogin = consent OTP (for card upgrade) |
| `status` | GENERATED / VALIDATED / EXPIRED |
| `validate_time` | Non-null means OTP was successfully validated |
| `ct` | OTP creation time |

### eth_crd_upgd_booking
**Role:** Final booking/application record — one row per upgrade application

| Column | Meaning |
|--------|----------|
| `rid` | Joins to CED_CCU_UserJourneyState.ridIndex |
| `account_id` | Joins to CED_CCU_UserJourneyState.udf1 |
| `upg_appl_reference` | Application reference number (non-null = booked) |
| `status` | APPROVED / REJECT / PENDING |
| `ct` | Booking creation timestamp (IST) |
| `card_type` | Target card the customer upgraded to |
| `source` | campaign / organic |
| `fee_amt` | Annual fee charged |

### eth_crd_upgd_event_log
**Role:** Error and failure events during the upgrade journey

| Column | Meaning |
|--------|----------|
| `ridIndex` | Joins to CED_CCU_UserJourneyState.ridIndex |
| `event_type` | Failure category code |
| `reason` | Full text description of the failure |
| `ct` | When failure occurred |

### cc_upgrade_mom_trend
**Role:** Month-on-month summary of campaign allocation and bookings (pre-aggregated)

| Column | Meaning |
|--------|----------|
| `month_year` | e.g. 'Apr-2025' |
| `allocation_received` | Total customers allocated to campaigns |
| `wa_delivered` | WhatsApp messages delivered |
| `sms_delivered` | SMS messages delivered |
| `total_booked` | Total CC upgrade bookings that month |

## Backlinks
- [[funnel/overview]] · [[index]]
