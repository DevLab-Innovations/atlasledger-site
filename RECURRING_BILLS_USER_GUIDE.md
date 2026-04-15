# Recurring Bills User Guide

Welcome to the Recurring Bills feature in Atlas Ledger. This guide will help you understand how to create, manage, and track your recurring bills with accuracy and ease.

## Table of Contents

- [What Are Recurring Bills?](#what-are-recurring-bills)
- [Getting Started](#getting-started)
- [Creating a Bill](#creating-a-bill)
- [Managing Bills](#managing-bills)
- [Understanding Payment Periods](#understanding-payment-periods)
- [Payment History](#payment-history)
- [Frequently Asked Questions](#frequently-asked-questions)

---

## What Are Recurring Bills?

Recurring bills are expenses that repeat on a regular schedule. Examples include:

- **Monthly subscriptions** (Netflix, Spotify, Adobe Creative Cloud)
- **Utilities** (electricity, water, internet)
- **Loan payments** (mortgage, car loan)
- **Insurance premiums**
- **Quarterly taxes**
- **Annual memberships**

Atlas Ledger tracks each payment period separately, giving you a complete payment history and helping you stay on top of when bills are due.

### How It Works

Instead of just marking a bill as "paid" or "unpaid," the app tracks:

- **Which period** the payment is for (e.g., "January 2026")
- **When it was paid** (actual payment date)
- **How much was paid** (supports variable amounts)
- **Payment status** (pending, paid, partial, or skipped)

This means that after you mark January's Netflix bill as paid, the app will automatically show February as pending when the new month arrives.

---

## Getting Started

### Prerequisites

The Recurring Bills feature is available with a **Premium subscription** or higher.

To access bills:

1. Open Atlas Ledger
2. Tap the **Money** tab in the bottom navigation
3. Tap **Bills** from the menu

---

## Creating a Bill

### Step 1: Start Creating a Bill

1. On the Bills List screen, tap the **+** button (bottom-right corner)
2. The Bill Form will open

### Step 2: Fill in Bill Details

| Field | Description | Example |
|-------|-------------|---------|
| **Name** | Name of the bill | "Netflix Subscription" |
| **Amount** | Regular payment amount | $15.99 |
| **Category** | Type of expense | "Subscription" |
| **Is Recurring** | Toggle on for bills that repeat | On |
| **Frequency** | How often the bill occurs | Monthly |
| **Due Day** | Day of the month/week it's due | 15 (15th of each month) |
| **Start Date** | When tracking begins | January 1, 2026 |
| **Notes** | Optional notes | "Family plan" |

### Step 3: Understanding Frequency Options

- **Weekly**: Bill repeats every week (e.g., daycare on Fridays)
- **Monthly**: Bill repeats each month on the same day (e.g., rent on the 1st)
- **Quarterly**: Bill repeats every 3 months (e.g., HOA fees)
- **Annual**: Bill repeats once per year (e.g., Amazon Prime membership)

### Step 4: Setting the Start Date

The **start date** determines when the app begins tracking this bill. It doesn't have to be the date you created the bill.

**Example**: If you create a bill on January 15th but set the start date to January 1st, the app will consider January your first payment period.

### Step 5: Save the Bill

Tap **Save** to create the bill. It will now appear in your Bills List.

---

## Managing Bills

### Viewing Your Bills

The Bills List shows all your bills grouped by status:

1. **Overdue** (red) - Bills past their due date that haven't been paid
2. **Upcoming** (yellow) - Bills due soon
3. **Paid** (green) - Bills already paid for the current period

Each bill card displays:

- Bill name and amount
- Category icon
- Full due date (e.g., "Due Jan 15, 2026")
- Period status (e.g., "Paid - January" or "Due in 5 days" or "3 days overdue")

### Marking a Bill as Paid

There are two ways to mark a bill as paid:

#### Method 1: Swipe to Pay (Quick)

1. Find the bill in the list
2. **Swipe right** on the bill card
3. Tap **Paid**
4. Confirm the payment period
5. Done! The app creates an expense entry and marks the period as paid

#### Method 2: Long Press (Advanced)

1. **Long press** on the bill card
2. Select **Mark as Paid**
3. Choose options:
   - **Payment Date**: When you actually paid (defaults to today)
   - **Amount Paid**: Actual amount (defaults to bill amount)
   - **Notes**: Optional note for this payment
4. Tap **Confirm**

### Variable Amount Bills

Some bills change each period (like utility bills). When marking them as paid:

1. Swipe right and tap **Paid**
2. In the confirmation dialog, tap **Adjust Amount**
3. Enter the actual amount you paid
4. The app tracks both the expected amount and actual amount
5. Your payment history will show the variance

**Example**: Your electric bill is usually $80, but in winter it was $127.50. The history will show:

```
December 2025    $127.50    Paid Dec 15    (expected $80.00, variance +$47.50)
```

### Skipping a Payment

If you intentionally skip a payment period:

1. Long press on the bill
2. Select **Skip This Payment**
3. Optionally add a reason (e.g., "On vacation")
4. The period is marked as skipped and won't show as overdue

### Undoing a Payment

If you marked a bill as paid by mistake:

1. The app shows a snackbar with an **Undo** button for 5 seconds
2. Tap **Undo** to reverse the action
3. The expense entry is deleted and the bill returns to pending status

Alternatively, you can delete the payment from the **Payment History** view.

### Editing a Bill

1. Long press on the bill card
2. Select **Edit**
3. Make your changes
4. Tap **Save**

**Note**: Changing the amount or frequency affects future periods only. Past payment history is preserved.

### Deleting a Bill

1. Long press on the bill card
2. Select **Delete**
3. Confirm the deletion

**Warning**: This deletes all payment history for this bill. Consider archiving instead if you want to preserve history.

---

## Understanding Payment Periods

### What Is a Period?

A **period** represents a specific billing cycle for your recurring bill. The format depends on the frequency:

| Frequency | Period Format | Example |
|-----------|---------------|---------|
| Weekly | "Week X, YYYY" | "Week 5, 2026" |
| Monthly | "Month YYYY" | "January 2026" |
| Quarterly | "QX YYYY" | "Q1 2026" (Jan-Mar) |
| Annual | "YYYY" | "2026" |

### Period Labels on Bills

When you view a bill in the list, you'll see its period status:

- **"Paid - January"** - You paid this bill for January already
- **"Due in 5 days"** - Bill is upcoming, due in 5 days
- **"Due Jan 15, 2026"** - Full due date for the current period
- **"3 days overdue"** - Bill was due 3 days ago and hasn't been paid

### How Overdue Detection Works

The app uses **full dates**, not just day numbers, to determine if a bill is overdue.

**Example**:
- Bill: Monthly bill due on the 1st
- Today: January 29th
- Status: **NOT overdue** if you already paid the January 1st bill
- On February 2nd: **Overdue** because the February 1st payment is past due

This prevents false overdue warnings when you've already paid the current period.

### Day 31 in Short Months

If your bill is due on the 31st:

- **January**: Due on Jan 31
- **February**: Due on Feb 28 (or Feb 29 in leap years)
- **March**: Due on Mar 31
- **April**: Due on Apr 30

The app automatically adjusts the due date to the last day of short months.

---

## Payment History

### Viewing Payment History

To see all past payments for a bill:

1. Long press on the bill card
2. Select **Payment History**
3. The history screen opens

Alternatively, tap the bill card to open details, then tap **View History**.

### What's in the Payment History

The payment history shows:

- **Period** (e.g., "January 2026")
- **Status** badge (Paid, Skipped, Partial, Pending)
- **Due Date** (when the bill was due)
- **Paid Date** (when you actually paid) - shows "(late)" if after due date
- **Amount** paid vs. expected
- **Variance** (if the amount paid differs from expected)
- **Reason** (for skipped payments)
- **Notes** (if you added any)

### History Grouped by Year

Payment history is organized by year for easy navigation:

```
2026
  January 2026     $15.99    Paid Jan 15
  February 2026    $15.99    Paid Feb 14

2025
  December 2025    $15.99    Paid Dec 15
  November 2025    $15.99    Paid Nov 15
  October 2025     $14.99    Paid Oct 15    (price changed)
```

### Deleting a Payment Record

If you need to remove a payment from history:

1. Open the Payment History screen
2. **Swipe left** on the payment record
3. Tap the **Delete** icon
4. Confirm deletion

**Warning**: This deletes the payment record and any linked expense entry.

---

## Frequently Asked Questions

### Q: What's the difference between recurring and one-time bills?

**Recurring bills** repeat on a schedule. After you pay one period, the next period automatically becomes pending.

**One-time bills** don't repeat. Once paid, they stay paid and don't cycle to a new period.

### Q: Can I retroactively record past payments?

Yes! When marking a bill as paid:

1. Tap **Adjust Date**
2. Select the date you actually paid in the past
3. The payment is recorded with the historical date

### Q: What if I miss a payment and want to pay it late?

The bill will show as overdue in the list. When you mark it as paid, the expense is created with the date you paid (today or a custom date), but the payment history will show it was late.

### Q: How do I handle bills with changing amounts?

When marking the bill as paid, adjust the amount to match what you actually paid. The history will track the variance between expected and actual amounts.

### Q: What happens if I change the bill amount?

Changing the bill amount affects future periods only. Past payment records are preserved with the amounts you actually paid.

### Q: Can I see all bills due in a specific month?

Yes, the Bills List screen has a filter option. Tap the filter icon and select a date range to see bills due in that period.

### Q: What if my payment is auto-paid from my bank?

You still need to manually mark the bill as paid in the app when you see the bank transaction. This creates the expense entry and tracks the payment.

**Future Enhancement**: Auto-pay tracking is planned for a future release.

### Q: How do I handle bills that are paid from a different account?

Atlas supports multiple accounts with a Premium subscription. When creating a bill, it's associated with your default account. To use a different account, change your default account in Settings before creating the bill.

### Q: What if I want to stop tracking a bill temporarily?

You can:

1. **Skip upcoming payments** individually (mark each period as "Skipped")
2. **Delete the bill** (but this removes all history)
3. **Edit the bill** and set `Is Recurring` to `Off` (makes it a one-time bill)

**Best Practice**: Use the Skip feature for temporary pauses (e.g., "On vacation for 2 months").

### Q: Can I export my bill payment history?

Yes! Bill payments are included in expense reports when you export to Excel or PDF. Go to **Reports** → **Export** and select your date range.

### Q: Do bill payments affect my cash flow reports?

Yes! When you mark a bill as paid, the app creates an expense entry in the corresponding category. This expense is included in all cash flow calculations and reports.

### Q: What's the difference between start_date and due_day?

- **Start Date**: The date you want to begin tracking this bill (determines first payment period)
- **Due Day**: The day of the month/week the bill is due (e.g., 15th of each month)

**Example**: Start date is Jan 1, 2026, due day is 15. Your first payment is due on Jan 15, 2026.

---

## Tips for Success

1. **Set up all your recurring bills** at once to get a complete picture of your obligations
2. **Use accurate start dates** to ensure periods align with your actual billing cycles
3. **Check the Bills List regularly** (or enable notifications) to avoid missed payments
4. **Review payment history** quarterly to spot trends (e.g., increasing utility costs)
5. **Use the Notes field** to track important details (e.g., "Called to negotiate lower rate")
6. **Adjust amounts for variable bills** to maintain accurate expense tracking
7. **Mark bills as Skipped** when appropriate to keep your records clean

---

## Need Help?

If you encounter issues or have questions:

- Check the **FAQ** above
- Visit the **Help Center** in the app (Settings → Help)
- Contact support at support@atlasledger.com

---

**Last Updated**: 2026-01-30
**Feature Version**: Recurring Bills v1.0
**Minimum App Version**: Atlas Ledger 2.0+
