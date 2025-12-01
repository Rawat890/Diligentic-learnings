[START: Component Mount or Focus]
            |
            v
 ┌──────────────────────────────┐
 │ useFocusEffect or useEffect │
 └────────────┬────────────────┘
              v
     [Call fetchPayments()]
              |
              v
 ┌─────────────────────────────────────────────────────────────┐
 │ fetchPayments()                                             │
 │  - Set loading = true (unless refreshing)                   │
 │  - API: POST /fetchPayments                                 │
 │  - Response:                                                │
 │    → payment_history                                        │
 │    → pending_payment_files                                  │
 │    → total_payment_amount                                   │
 │  - Update state variables                                   │
 │  - Set loading = false                                      │
 └─────────────────────────────────────────────────────────────┘
              |
              v
 ┌──────────────────────────────────────────────────────────────┐
 │ Setup socket listeners:                                      │
 │  - "/patient-file-payment-succeeded" → fetchPayments()       │
 │  - "/refresh-files-awaiting-submission-list" → fetchPayments│
 └──────────────────────────────────────────────────────────────┘
              |
              v
 ┌────────────────────────────────────────────┐
 │ RENDER UI using FlatList                   │
 │ - Header:                                  │
 │    If pendingPayment exists:               │
 │      → Show payment card(s)                │
 │      → Show "Pay Now" button               │
 │    If paymentHistory exists:               │
 │      → Show summary with export button     │
 └────────────────────────────────────────────┘
              |
              v
 ┌────────────────────────────────────────────┐
 │ [User taps "Pay Now" button]               │
 └────────────┬───────────────────────────────┘
              v
 ┌─────────────────────────────────────────────────────────────┐
 │ createCheckoutSession()                                     │
 │  - Get first pendingPayment item                            │
 │  - API: POST /createPaymentIntent with patient_file_id      │
 │  - Response: client_secret                                  │
 │  - Stripe: initPaymentSheet with client_secret              │
 │  - Stripe: presentPaymentSheet()                            │
 │      - If cancelled → do nothing                            │
 │      - If success:                                          │
 │          - Show success message                             │
 │          - Call fetchPayments() to refresh data             │
 └─────────────────────────────────────────────────────────────┘
              |
              v
 ┌────────────────────────────────────────────┐
 │ [Payment History List Display]             │
 │ - FlatList items:                          │
 │    → id_number                             │
 │    → amount                                │
 │    → date (formatted)                      │
 │    → status (confirmed/cancelled)          │
 │    → relevant icon                         │
 └────────────────────────────────────────────┘
              |
              v
 ┌────────────────────────────────────────────┐
 │ [User taps Export Icon]                    │
 └────────────┬───────────────────────────────┘
              v
 ┌─────────────────────────────────────────────────────────────┐
 │ exportPayments()                                            │
 │  - API: POST /exportPayments                                │
 │  - Response: csv_file                                       │
 │  - Construct S3 URL                                         │
 │  - Determine download path (iOS vs Android)                 │
 │  - Use react-native-blob-util to download CSV               │
 │  - Show success/failure message                             │
 └─────────────────────────────────────────────────────────────┘
              |
              v
     ┌────────────────────────┐
     │ [Pull to Refresh]      │
     │ → setRefreshing(true)  │
     │ → call fetchPayments() │
     └────────────────────────┘
              |
              v
          [END STATE]



🔄 COMPONENT LIFECYCLE SUMMARY
Trigger	Effect
useFocusEffect	Calls fetchPayments()
useEffect (on mount)	Sets up socket listeners
Socket event (payment succeeded)	Calls fetchPayments()
Socket event (refresh list)	Calls fetchPayments()
Pull to refresh gesture	Calls fetchPayments()
Successful Stripe payment	Calls fetchPayments()

⚡️ Why Is Socket Used?

Here's the breakdown:

✅ 1. Real-Time Feedback After Payment

Event: "/patient-file-payment-succeeded"

Purpose: When a payment is completed successfully (via Stripe or admin update), the server emits this event to the client.

Result: The client calls fetchPayments() to refresh the payment history and pending payments list immediately.

💡 This avoids the user having to manually refresh or navigate away and back again to see updated info.

✅ 2. Refresh Files Awaiting Submission

Event: "/refresh-files-awaiting-submission-list"

Purpose: Used when the list of pending payment files has changed — e.g., a backend job updates file states.

Result: Again, the client re-fetches payment data.

🔁 Without Sockets?

If sockets were not used:

You would have to use polling (e.g., setInterval) or require the user to manually refresh.

It would lead to a laggy UX where users might see stale payment info.

You risk showing wrong payment states (e.g., pending when it’s actually paid).