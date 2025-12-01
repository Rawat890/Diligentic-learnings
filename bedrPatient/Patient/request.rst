START
  |
  v
Load Session Tokens from Redux Store (accessToken, refreshToken, publicKey, privateKey)
  |
  v
Add Required Headers (origin, host)
  |
  v
Is URL in `urlsToBeIgnoredForEncryption`?
  |                \
  | NO              YES
  |                 |
  v                 v
Encrypt Request Body?
  |
  v
Is URL in `urlsToBeEncryptedByStaticKey`?
  |               \
  | YES            NO
  |                |
  v                v
Encrypt with Static Public Key    Encrypt with Dynamic Public Key
  |
  v
Add accessToken to Headers (if available)
  |
  v
SEND REQUEST via Axios
  |
  v
Is Response Status 401 (Unauthorized)?
  |               \
  | YES            NO
  |                |
  v                v
Is `isRefreshing` already TRUE?
  |                    \ 
  | YES                NO
  |                     |
  v                     v
Queue Request       Set `isRefreshing` = TRUE
  |                 |
  v                 v
Wait for Refresh     Call `getRefreshToken(refreshToken)`
                    |
                    v
        Was refresh successful?
            |                    \
           YES                     NO
            |                      |
            v                      v
      Save new access token        Clear Session, Show "Session Expired"
      Update Redux + Secure Storage
      |
      v
      Retry all queued requests
      |
      v
Decrypt Response?
  |
  v
Is URL in `urlsToBeDecryptedByStaticKey`?
  |                                   \
  | YES                                NO
  |                                     |
  v                                     v
Decrypt with Static Private Key   Decrypt with Dynamic Private Key
  |
  v
RETURN DECRYPTED RESPONSE
  |
  v
END



ChatGPT said:

The provided code is a custom secure HTTP request handler built on top of Axios for a React Native application. It supports:

✅ Dynamic encryption/decryption of request/response payloads

✅ Session token refresh flow when access token expires (HTTP 401)

✅ Key management and secure storage

✅ Request queuing and retrying during token refresh

✅ Environment-specific behavior

🧠 Overall Purpose

This is a secure, robust API layer that:

Encrypts outgoing requests

Decrypts incoming responses

Handles expired access tokens via refresh flow

Safely stores updated session credentials

Manages which URLs to skip for encryption/decryption

🧩 Let's Break It Down – Line by Line
✅ Imports
import Axios, { AxiosRequestConfig } from "axios";


Imports Axios and its types for request configuration.

import { NativeModules } from "react-native";
import Config from "react-native-config";


Gets access to native modules and environment variables from .env.

import i18next from "../localization";
import { updateSessionTokens } from "../screens/Common/PersonalInformation/sessionSlice";
import store from "../store";


Handles localization and Redux store session state updates.

import { ENVS, RESET_STORE, SESSION_TOKENS, URLS } from "./constants";
import { decrypted, encrypted } from "./helperFunctions/encryption";
import { setSecuredData } from "./helperFunctions/localStorage";
import {
  removeLocalUserDetails,
  showErrorMessage,
} from "./helperFunctions/miscellaneous";


App-specific constants and helpers for encryption, local storage, error handling.

🚀 Axios Instance
export const axios = Axios.create({
  baseURL: URLS.baseUrl,
  validateStatus: (status) => {
    if (status === 200 || status === 401) {
      return true;
    }
    return false;
  },
});


Creates a custom Axios instance with a base URL.

Accepts 200 OK and 401 Unauthorized as valid statuses (for token refresh handling).

🧾 URL Lists for Handling Encryption/Decryption
const urlsToBeIgnoredForEncryption = [...];
const urlsToBeEncryptedByStaticKey = [...];
const urlsToBeIgnoredForDecryption = [...];
const urlsToBeDecryptedByStaticKey = [...];


These arrays define:

URLs that should not be encrypted

URLs that should use a static public key

URLs that should not be decrypted

URLs that should be decrypted with a static private key

This prevents encrypting binary uploads or open endpoints like privacy policy.

🔐 Key Variables
let ztQHtgPBBW5ZJrrYjJtM = "";
// ... more key parts (obfuscated names)

let isRefreshing = false;


These hold public/private key values, likely retrieved securely from native code.

isRefreshing flag is used to prevent multiple simultaneous token refresh calls.

🔑 Get Keys from Native Module
const ggNUxCzxsTEpDLOzUGpn = async () => { ... }
const ThL9hKcApbzisyn8uVXl = async () => { await ggNUxCzxsTEpDLOzUGpn(); }
ThL9hKcApbzisyn8uVXl();


Calls a native module Package1 to get partial key components.

Combines them with .env parts to create full keys used for encryption.

🛑 This approach improves security by not storing the full key in JS.

🔐 Get Public/Private Key Based on Env
const a1dckIrCMkUk5Q1oE0o4 = (): string => { ... }
const z6Zt1Og6EmTJiCayZQLN = (): string => { ... }


Returns environment-specific public/private keys.

🔁 Token Refresh Request
const getRefreshToken = async (refreshToken: string) => {
  ...
};


Sends a refresh_token to the backend to get a new access token.

🔁 Retry Queue
interface RetryQueueItem {
  resolve: ...
  reject: ...
  newConfig: ...
}
const refreshAndRetryQueue: RetryQueueItem[] = [];


Queue that holds failed requests during a token refresh.

Once new token is obtained, these are retried automatically.

💼 Core Request Function
export const request = async (config: AxiosRequestConfig) => {
  ...
};


This is the main custom request handler. Here's how it works:

🔸1. Attach Headers
config.headers = {
  ...config.headers,
  origin: Config.ORIGIN_URL,
  host: Config.HOST_URL,
};


Adds custom headers for backend to verify origin.

🔸2. Get Static Keys (Fallback)
const JHSBDV3H43BDBjdsjjhc = sjhbdch3jhwchahc();
const jsbdcshb3shvbiasv23 = ehbdvevh32hebvhn();


Fallback static public/private keys from elsewhere if not in state.

🔸3. Encrypt Request Body
let requestBody = config.data;
if (!urlsToBeIgnoredForEncryption.includes(...) && ...) {
  requestBody = encrypted({ data: config.data, publicKey });
}


Encrypts data with dynamic or static key before sending.

🔸4. Add Access Token Header
if (accessToken) {
  config.headers = {
    ...config.headers,
    access_token: accessToken,
  };
}


Sends JWT or similar auth token.

🔸5. Make Request
const response = await axios(config);

🔸6. Handle 401 Unauthorized
if (response.status === 401) {
  ...
}

▶ If Already Refreshing:
refreshAndRetryQueue.push({ newConfig, resolve, reject });


Queues the request and exits. It’ll retry once the refresh completes.

▶ If Not Refreshing:
isRefreshing = true;
try {
  const data = await getRefreshToken(refreshToken);
  ...
  refreshAndRetryQueue.forEach(...);
}


Tries to get a new token

Updates Redux store and local storage

Retries all queued requests

▶ On Refresh Failure:
removeLocalUserDetails();
store.dispatch({ type: RESET_STORE });
showErrorMessage(i18next.t("sessionExpired"));


Clears user session

Logs out the user with an error message

🔸7. Decrypt Response
if (urlsToBeDecryptedByStaticKey.includes(config.url)) {
  ...
} else if (!urlsToBeIgnoredForDecryption.includes(config.url)) {
  ...
}


Decrypts response payload using:

Static private key for special URLs

User’s private key for most others

📦 Returns Final Response (Decrypted or Raw)
return decryptedResponse;

📌 Summary of Functionality
Feature	Purpose
Axios.create()	Central request manager with status control
Native key loading	Securely gets keys for encryption
Dynamic/static encryption logic	Flexible encryption based on URL
Token refresh handler	Automatic session renewal on 401
Retry queue	Ensures failed requests retry after refresh
Local & Redux state update	Keeps tokens in sync
Custom error handling	Localized, session-expiry alerts