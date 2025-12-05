(AddBankAccount)

This is a class-based screen that:

Lets the user enter bank details

Validates input
Adds the bank account to Stripe
Triggers KYC if needed
Uses Redux to get user info
Uses NavigationService to redirect

🔹 Key Functionalities Explained
1. State Variables
Variable	Purpose
accountNumber	Bank account number
routingNumber	Bank routing number
accountHolderName	Owner name
accountType	Individual / Company
isAbsoluteLoading	Loader overlay
2. Adding Bank Account

Steps:

Validate required fields
Create hardcoded test bank object
Call actions.addCardApi
Update card list in Redux

Navigate back

If KYC not complete → show alert → allow user to start KYC

3. Deep Link Listener

Linking.addEventListener('url', this.handleDeepLink)
Used for Stripe redirect after KYC verification.

📘 Flowchart 2 — Add Bank Account
                  ┌─────────────────────────────┐
                  │ User Opens AddBankAccount   │
                  └──────────────┬──────────────┘
                                 │
                                 ▼
                 ┌──────────────────────────────┐
                 │ User Enters Bank Details     │
                 └──────────────┬──────────────┘
                                 │
                      Press "Add bank account"
                                 │
                                 ▼
                 ┌──────────────────────────────┐
                 │ Validate User Input          │
                 └───────┬──────────────────────┘
                         │
         ┌───────────────┼───────────────────────────────┐
         ▼               ▼                               ▼
 Invalid Field → showSnackBar()          Valid → continue
                         │
                         ▼
         Build test Stripe bank details object
                         │
                         ▼
              Call actions.addCardApi()
                         │
                         ▼
           Update cards in Redux (getAllCards)
                         │
                         ▼
                   Navigation.goBack()
                         │
                         ▼
         ┌────────────────────────────────────────┐
         │ If stripe_KYC not done → Show Alert    │
         │   “Do KYC Now / Do Later”              │
         └────────────────────────────────────────┘
                         │
                     If YES
                         │
                         ▼
              Call doStripeSellerKYC()
