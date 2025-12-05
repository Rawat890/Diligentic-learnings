✅ PART 1 — Explanation of the First Code (ListCheddaWalletTransactions)

This React Native component displays Chedda Wallet transactions and supports:

Pagination

Filtering

Merging multiple transaction sources

Sorting

Viewing transaction details

Transferring wallet money to Stripe

Handling KYC and bank account requirements

🔹 Key Functionalities Explained
1. State Variables
Variable	Purpose
transactions	Stores loaded transactions
loadingTransactions	Loader for initial fetch or actions
loadingMore	Loader for pagination
skip	Pagination offset
hasMore	Determines if more items exist
orderId	If passed, list only transactions for a specific order

2. Fetching Wallet Transactions

getWalletTransactions is the heart of this file.

Steps:

Set loading flags

Create API payload (skip, limit, order_id)

Call actions.getCheddaWalletTransaction(payload)

API returns three arrays:

wallet_transactions
deductions
group_payments_received

These are merged into one list

Remove duplicates by _id
Sort newest → oldest
Apply pagination logic
Update the state

3. Rendering Each Transaction

For each transaction:

Show profile picture OR initials

Show name + formatted timestamp

Show amount with + or – sign

If order page: hide +/-

Navigate to details screen depending on:

Tip transaction
Group payment
Normal transaction

4. Loading More Transactions

Triggered when list reaches bottom:

if (!loadingMore && hasMore) {
    getWalletTransactions(true);
}

5. Withdrawing Money to Stripe

User presses “Transfer Money Now”:

Flow:

Check if user has positive balance
Check if Stripe account exists
Check if KYC is completed

If all good → call payToStripe() → Stripe payout

6. UI Rendering

Header
Withdrawal button
FlatList of transactions
Loader overlay

📘 Flowchart 1 — Wallet Transaction Listing
                      ┌──────────────────────────┐
                      │ Component Mounts         │
                      └───────────┬──────────────┘
                                  │
                                  ▼
                     ┌──────────────────────────┐
                     │ getWalletTransactions()  │
                     └───────────┬──────────────┘
                                 │
                 ┌───────────────┼────────────────┐
                 │Initial Load?   │Load More?      │
                 ▼               ▼                ▼
     Set loadingTransactions   Set loadingMore   Set skip
                 │
                 ▼
     Build API payload (skip, limit, order_id?)
                 │
                 ▼
        Call actions.getCheddaWalletTransaction
                 │
                 ▼
     Merge all 3 arrays from response
                 │
                 ▼
     Remove duplicates by _id
                 │
                 ▼
     Sort by created_at (desc)
                 │
                 ▼
     Update transactions state
   (replace or merge depending on loadMore)
                 │
                 ▼
         Update skip & hasMore
                 │
                 ▼
     Stop all loading indicators


User Interactions:

Press Transaction → Navigate to:
  - TipTransactionDetails
  - GroupTransactionDetails
  - TransactionDetails

Scroll to bottom → handleLoadMore()

Press “Transfer Money Now” →
   ┌──────────────────────────────────┐
   │ Check Stripe Account Exists?     │
   └───────────────────┬──────────────┘
                       ▼
        Check KYC Completed? 
                       │
                       └──No → Alert → Start KYC
                       │
                       ▼
                 Yes → payToStripe()


