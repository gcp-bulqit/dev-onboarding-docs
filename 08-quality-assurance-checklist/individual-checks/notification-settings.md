# Notification Settings by User Type

This document provides a simplified view of all notifications organized by user type for implementing the notification settings page/tab in each dashboard.

Each notification has:
- **Title**: Display name for the setting
- **Description**: Brief explanation of when this notification is triggered
- **Default**: Whether the notification is enabled by default (for toggleable notifications)

---

## Admin Notifications

| ID | Title | Description | Channels | Default |
|----|-------|-------------|----------|---------|
| A01 | Block Threshold Met | When a block reaches its subscription threshold | Message Center, Email, Dashboard, SMS | On |
| A02 | Bids Submitted | When vendors submit bids for an open service | Message Center, Email, Dashboard, SMS | On |
| A03 | Bidding Ended | When the bidding period for a service closes | Message Center, Email, Dashboard, SMS | On |
| A04 | Revised Proposal Submitted | When a finalist vendor submits a revised proposal | Message Center, Email, Dashboard, SMS | On |
| A05 | Low Rating Alert | When a homeowner submits a rating of 3 stars or fewer | Message Center, Email, Dashboard, SMS | On |
| A06 | Vendor Payment Failed | When a scheduled payment to a vendor fails | Message Center, Email, Dashboard, SMS | On |
| A07 | Homeowner Payment Failed | When a homeowner's credit card charge fails | Message Center, Email, Dashboard, SMS | On |
| A08 | Interest Threshold Met | When interested homeowners reach the predefined threshold | Message Center, Email, Dashboard, SMS | On |
| A09 | Vendor Low Rating Response Overdue | When a vendor hasn't responded to a low rating within 24h | Message Center, Email, Dashboard, SMS | On |
| A10 | Daily Scorecard | Daily platform health snapshot (scheduled at 8:00 AM) | Message Center, Email | On |
| A11 | Vendor Score Below Threshold | When a vendor's Bulqit Score drops below acceptable threshold | Message Center, Email, Dashboard, SMS | On |

### Planned Admin Notifications (Not Yet Implemented)

| ID | Title | Description |
|----|-------|-------------|
| NEW1 | Document Expiring | When a vendor's document is expiring |
| NEW2 | Unread Message Reminder | When a vendor doesn't acknowledge a message/notification |

---

## Vendor Notifications

| ID | Title | Description | Channels | Default |
|----|-------|-------------|----------|---------|
| V01 | Application Approved | When your vendor application is approved | Message Center, Email, Dashboard, SMS | On |
| V02 | Application Not Approved | When your vendor application is not approved | Message Center, Email, Dashboard, SMS | On |
| V03 | New Bidding Opportunity | When a block/service opens for bidding | Message Center, Email, Dashboard, SMS | On |
| V04 | Selected as Finalist | When you're selected as a finalist for a block | Message Center, Email, Dashboard, SMS | On |
| V05 | Bid Not Selected | When your bid was not selected to advance | Message Center, Email, Dashboard, SMS | On |
| V06 | Block Awarded | When you're awarded a block-service | Message Center, Email, Dashboard, SMS | On |
| V07 | Unscheduled Homeowners | When homeowners in your block need scheduling | Message Center, Email, Dashboard, SMS | On |
| V08 | Daily Schedule Reminder | Daily reminder of upcoming jobs | Message Center, Email, Dashboard, SMS | On |
| V09 | Update Job Status Reminder | When jobs are past their scheduled date without completion | Message Center, Email, Dashboard, SMS | On |
| V10 | Low Rating Received | When a homeowner submits a low rating for your service | Message Center, Email, Dashboard, SMS | On |
| V11 | Homeowner Payment Failed | When a homeowner's payment for a completed job fails | Message Center, Email, Dashboard, SMS | On |
| V12 | Homeowner Service Paused | When a homeowner's service is paused due to payment issues | Message Center, Email, Dashboard, SMS | On |
| V13 | Homeowner Service Resumed | When a paused homeowner's service is resumed | Message Center, Email, Dashboard, SMS | On |
| V14 | Homeowner Skipped Service | When a homeowner skips their next scheduled service | Message Center, Email, Dashboard, SMS | On |
| V15 | Homeowner Canceled Service | When a homeowner cancels their recurring subscription | Message Center, Email, Dashboard, SMS | On |

### Planned Vendor Notifications (Not Yet Implemented)

| ID | Title | Description |
|----|-------|-------------|
| NEW3 | Bidding Expiring Soon | When a service you can bid on is expiring soon |
| NEW4 | Bidding Extended | When bidding has been extended |
| NEW9 | Upcoming Jobs Digest | List of upcoming jobs |
| NEW10 | Resolution Completed | Confirmation when a negative issue is resolved |
| NEW13 | Payout Summary | When a payout is approved |
| NEW15 | Did Not Win Bid | When another vendor is selected |
| NEW16 | Scheduling Delay Reminder | When you haven't scheduled members in X days |
| NEW21 | Team Member Invited | When you invite a new team member |
| NEW23 | Declined Bid Confirmation | Confirmation when you decline a final bid |
| N24 | Tax Documents Uploaded | When tax documents are uploaded |
| N25 | Score Drop Alert | When your Bulqit score drops |
| N27 | Bulqit Payment Failed | When Bulqit payment to you fails |

---

## Member (Homeowner) Notifications

| ID | Title | Description | Channels | Default |
|----|-------|-------------|----------|---------|
| M01 | Block Ready | When your block reaches threshold and is open for vendor evaluation | Message Center, Email, Dashboard, SMS | On |
| M02 | Vendor Selected | When a vendor is selected for your block/service | Message Center, Email, Dashboard, SMS | On |
| M03 | Service Scheduled | When your vendor schedules your service | Message Center, Email, Dashboard, SMS | On |
| M04 | Enrollment Reminder | Reminder to complete service enrollment | Message Center, Email, Dashboard, SMS | On |
| M05 | Schedule Changed | When your vendor modifies your service schedule | Message Center, Email, Dashboard, SMS | On |
| M06 | Service Complete | When your service is complete and billed | Message Center, Email, Dashboard, SMS | On |
| M07 | Payment Failed | When your payment fails | Message Center, Email, Dashboard, SMS | On |
| M08 | Vendor Changed | When your service is reassigned to a new vendor | Message Center, Email, Dashboard, SMS | On |
| M09 | Add-On Proposal | When your vendor proposes an additional service | Message Center, Email, Dashboard, SMS | On |

### Planned Member Notifications (Not Yet Implemented)

| ID | Title | Description |
|----|-------|-------------|
| NEW5 | Upcoming Service Reminder | Reminder that a service is upcoming |
| NEW6 | Price Changed | When the SAM price has changed |
| NEW7 | Abandoned Signup | When you start signing up but don't complete |
| NEW8 | Vendor Skipped Job | When a vendor skips a job |
| NEW10 | Resolution Completed | Confirmation when a negative issue is resolved |
| NEW11 | Issue Acknowledged | Confirmation that your issue is being addressed |
| NEW12 | Referral Credit Added | When a referral signs up and credit is added |
| NEW14 | Refund/Credit Given | When a refund or credit is given |
| NEW17 | Email/Phone Verification | Verification confirmation |
| NEW18 | Cancel Confirmation | Confirmation of service/account cancellation |
| NEW19 | Schedule Shared | When schedule is shared with you |
| NEW20 | Service Rescheduled | When vendor reschedules a job |
| NEW22 | Property Info Changed | When vendor changes your property information |
| N26 | Credit Card Expiring | Reminder before credit card expires |

---

## Field Worker Notifications

| ID | Title | Description | Channels | Default |
|----|-------|-------------|----------|---------|
| F01 | Daily Schedule | Your scheduled jobs for the day | Message Center, Email, Dashboard, SMS | On |

---

## Implementation Notes

### Settings Page Structure

Each user type's notification settings page should display:

```
[Notification Title]
[Description text]                                    [Toggle On/Off]
```
