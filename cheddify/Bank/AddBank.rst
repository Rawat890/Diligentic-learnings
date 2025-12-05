[ Name is same as component name ]

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



✅ Explanation of getAllCards

This function is a Redux Thunk action that fetches either:

All payment cards (type = CARD)

All bank accounts (type = BANK)

It then dispatches the appropriate Redux actions depending on:

Request start

Request success

Request failure

It also optionally returns the default card via a callback.

🔹 How It Works (Step-by-Step)
1. The function receives an object
{ type, cb }


type → either CARD or BANK

cb → optional callback for default card

2. It returns a function for Redux Thunk
return (dispatch) => {}


This allows asynchronous API calls.

3. Dispatching "REQUESTED" actions

Depending on the type:

If requesting CARD list:
dispatch({ type: GET_ALL_CARDS_REQUESTED })

If requesting BANK list:
dispatch({ type: GET_ALL_BANKS_REQUESTED })


These typically turn on loaders.

4. API call
listCardApi({ type })


This fetches the list of cards or bank accounts from backend.

5. On Success

If it's CARD:

dispatch({
  type: GET_ALL_CARDS_SUCCEEDED,
  payload: { allCards: res.data }
});


Also find default card:

const defaultCard = res.data.find(card => card.default);


If callback exists AND default card found:

cb(null, defaultCard);


If it's BANK:

dispatch({
  type: GET_ALL_BANKS_SUCCEEDED,
  payload: { allBanks: res.data },
});

6. On Error

If error and type = CARD:

dispatch({ type: GET_ALL_CARDS_FAILED });


If error and type = BANK:

dispatch({ type: GET_ALL_BANKS_FAILED });

📘 Flowchart — getAllCards Action
                    ┌─────────────────────────┐
                    │ getAllCards({type, cb}) │
                    └───────────┬─────────────┘
                                │
                                ▼
                     ┌──────────────────────┐
                     │ return dispatch(...) │
                     └───────────┬─────────┘
                                 │
            ┌────────────────────┼────────────────────┐
            │                    │                    │
            ▼                    ▼                    ▼
If type=CARD            dispatch(GET_ALL_CARDS_REQUESTED)
If type=BANK            dispatch(GET_ALL_BANKS_REQUESTED)

                                 │
                                 ▼
                    Call listCardApi({type})
                                 │
               ┌─────────────────┼─────────────────────┐
               │                 │                     │
               ▼                 ▼                     ▼
        ┌────────────────────────────────┐
        │ API Success                    │
        └────────────────┬───────────────┘
                         │
           ┌─────────────┼──────────────────────────────┐
           │             │                                │
           ▼             ▼                                ▼
 If type=CARD      dispatch(GET_ALL_CARDS_SUCCEEDED)      │
                    payload = allCards                    │
                    Find defaultCard                      │
                    If cb and defaultCard → cb(default)   │

 If type=BANK      dispatch(GET_ALL_BANKS_SUCCEEDED)       
                    payload = allBanks

                                 │
                                 ▼
        ┌────────────────────────────────┐
        │ API Error                      │
        └────────────────┬───────────────┘
                         │
           ┌─────────────┼──────────────────────────────┐
           │             │                                │
           ▼             ▼                                ▼
If type=CARD      dispatch(GET_ALL_CARDS_FAILED)

If type=BANK      dispatch(GET_ALL_BANKS_FAILED