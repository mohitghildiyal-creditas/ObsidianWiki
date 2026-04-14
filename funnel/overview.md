# Funnel Overview — IBL Credit Card Upgrade
*Compiled: 2026-04-08 02:30* `funnel` `ibl-ccu`

## Full Funnel Breakdown

| Stage | Campaign | Conv% | Organic | Conv% |
|-------|----------|-------|---------|-------|
| Sessions | 0 | — | 0 | — |
| Clicked CTA | 0 | N/A | 0 | N/A |
| Mobile Validated | 0 | N/A | 0 | N/A |
| Login OTP Validated | 0 | N/A | 0 | N/A |
| Consent OTP Validated | 0 | N/A | 0 | N/A |
| Booked | 0 | N/A | 0 | N/A |
| Approved | 0 | N/A | 0 | N/A |

## Top Failure Reasons

| Failure Type | Occurrences |
|-------------|-------------|

## Funnel Stage Definitions
- **1_Sent**: COUNT(DISTINCT AccountIdIndex) from response table — total customers targeted
- **2_Delivered**: SMS: StatusIndex = 'Delivered'. Email: Status = 'Delivered'. WhatsApp: StatusIndex IN ('DELIVERED','READ').

- **3_Open_Read**: SMS: always 0 (SMS has no open tracking). Email: Status IN ('OPENED','CLICKED'). WhatsApp: StatusIndex = 'READ'.

- **4_Sessions**: COUNT(DISTINCT ridIndex) in CED_CCU_UserJourneyState WHERE source_id IN (UUIDs from response) AND source = 'campaign'. This = customers who actually clicked the link.

- **5_Benefit_Carousel**: eth_ccu_event_tracking: action = 'card_upgrade_carousal_on_1_card'
- **6_Clicked_UpgradeNow_CTA**: eth_crd_upgd_attempt_log: stage='SELECTION' AND status='success'
- **7_Login_Screen**: eth_ccu_event_tracking: action = 'card_upgrade_cc_validate_modal_opened'
- **8_Mobile_Validated**: eth_crd_upgd_attempt_log: stage='MOBILE' AND status='success'
- **9_LoginOTP_Triggered**: eth_crd_upgd_otp: workflow='prelogin' AND status IN ('GENERATED','VALIDATED','EXPIRED')
- **10_LoginOTP_Validated**: eth_crd_upgd_otp: workflow='prelogin' AND validate_time IS NOT NULL
- **11_Upgrade_Summary_Page**: eth_ccu_event_tracking: action = 'welcome_gift_preview_page_landed'
- **12_Upgrade_Summary_CTA**: eth_ccu_event_tracking: action = 'welcome_screen_update_card_button_clicked'
- **13_ConsentOTP_Screen**: eth_ccu_event_tracking: action = 'card_upgrade_otp_consent_modal_opened'
- **14_ConsentOTP_Triggered**: eth_crd_upgd_otp: workflow='postlogin' AND status IN ('GENERATED','VALIDATED','EXPIRED')
- **15_ConsentOTP_Validated**: eth_crd_upgd_otp: workflow='postlogin' AND validate_time IS NOT NULL
- **16_Booked**: eth_crd_upgd_booking: upg_appl_reference IS NOT NULL
- **17_Approved**: eth_crd_upgd_booking: status = 'APPROVED'

## Improvement Rules
- If session_rate < 2%, SMS link or delivery is the bottleneck — check vendor delivery rates
- If SELECTION stage success < 50% of sessions, upgrade CTA copy/placement needs A/B testing
- If OTP drop-off > 30%, check OTP delivery latency — expired OTPs are the top drop reason
- If Fee_charge_fail is high, pre-validate customer balance before initiating upgrade flow
- If Card_blocked > 10% of failures, CRM data used for targeting is stale — refresh eligibility list
- WhatsApp consistently outperforms SMS on session_rate and booking_rate — shift budget accordingly
- Same-day booking rate indicates urgency; schedule campaigns on salary dates (1st, 7th) for higher same-day conversion
- Compare SELECTION success vs API_failure — high API failures = backend infra issue, not targeting

## Backlinks
- [[channels/sms]] · [[channels/email]] · [[channels/whatsapp]]
- [[index]] — back to master index
