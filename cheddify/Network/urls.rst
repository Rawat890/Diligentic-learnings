✅ High-Level Purpose

This code dynamically manages your API base URL (apiURL) in a React Native project.
It:

Loads an initial API URL from react-native-config

Overrides it with a value stored in AsyncStorage (but only in STAGE environment)

Exposes helper functions so other parts of the app always use the updated URL

This allows you to change backend URL at runtime without rebuilding the app.

🔍 Detailed Step-by-Step Explanation
1. Imports
import AsyncStorage from '@react-native-async-storage/async-storage';
import Config from 'react-native-config';


AsyncStorage → for storing key–value pairs persistently on the device

Config → environment variables from .env files (e.g., .env.stage, .env.dev)

2. Initial API URL
let apiURL = Config.API_BASE_URL;


At startup, apiURL is whatever your .env file defines.

3. Immediately Invoked Async Function (IIFE)
(async () => {
  if (Config.APP_ENV === "STAGE") {
    try {
      const allKeys = await AsyncStorage.getAllKeys();

      const storedUrl = await AsyncStorage.getItem("API_BASE_URL");
      if (storedUrl) {
        apiURL = storedUrl;
      }
      const hasApiKey = allKeys.includes("API_BASE_URL");
    } catch (error) {
      console.error('Failed to load stored API URL:', error);
    }
  }
})();

What this does:
✔ Runs automatically when the file loads

The IIFE executes immediately — before anything else imports this file.

✔ Only runs when APP_ENV = "STAGE"
Steps inside it:

It gets all stored keys from AsyncStorage.

It checks if "API_BASE_URL" is saved.

If yes → it overrides the default apiURL.

Otherwise → keeps the .env value.

This allows QA/Stage builds to change URL dynamically without rebuilding.

4. updateApiUrl function
export const updateApiUrl = async () => {
  if (Config.APP_ENV === "STAGE") {
    try {
      const storedUrl = await AsyncStorage.getItem("API_BASE_URL");
      if (storedUrl) {
        apiURL = storedUrl;
      }
    } catch (error) {
      console.error('Failed to update API URL:', error);
    }
  }
  return apiURL;
};

Purpose:

Lets you refresh the API URL on-demand (e.g., when reopening an environment-switching screen)

Same logic as the IIFE, but callable manually

5. URL Getter Functions
export const getSocketUrl = () => apiURL;
export const getBaseUrl = () => `${apiURL}/User/`;
export const getUploadUrl = () => `${apiURL}/Upload/`;


These always compute URLs using the current value of apiURL.

So even if apiURL changes later, your app uses the updated value.

6. Exported Versions as Functions
export const SOCKET_URL = () => getSocketUrl();
export const BASEURL = () => getBaseUrl();
export const API_BASE_IMAGE_UPLOAD = () => getUploadUrl();


These are function wrappers so the values are always fresh when called.

🧠 Why the code uses functions instead of constants

If you exported constants like:

export const BASEURL = `${apiURL}/User/`;


then it would be computed only once, and would not change if apiURL changes.
Using functions ensures the value always reflects the latest URL.

📌 Summary
Section	Purpose
Load default URL	From .env file
IIFE	Automatically override URL from AsyncStorage on startup
updateApiUrl()	Manually refresh URL later
getBaseUrl() etc.	Always return URL using the current apiURL
Exports	Ensure dynamic runtime URL switching works
✔ When would you use this code?

Changing API server URL in staging builds without rebuilding

Allowing QA to switch API endpoints

Dynamically pointing the app to new backend deployments

Feature flagging via AsyncStorage



Here’s an explanation of what these additional lines do — and why they work the way they do — in the same style as before.

🔍 What These Endpoint Functions Do
export const LOGIN_USER = () => BASEURL() + 'user_login';
export const SOCIAL_LOGIN = () => BASEURL() + 'social_login';
export const SIGNUP_STEPONE = () => BASEURL() + 'user_signup_step1';


Each exported constant is a function that returns a complete API endpoint using the latest apiURL.

This is important because apiURL is dynamic (can change at runtime if stored in AsyncStorage).
If these were simple constants, they would not update.

🧠 Why These Are Functions (Not Strings)

apiURL can change (when Stage mode reloads from AsyncStorage).

Example:

App loads with .env value → apiURL = https://stage.example.com

Later, AsyncStorage returns a new value → apiURL = https://new-server.test

If endpoints were constants, they would NOT update.

Using functions ensures:

LOGIN_USER()


always reflects the current value of apiURL.

📐 Example Resolution

If:

apiURL = "https://staging.example.com";


and BaseURL function is:

BASEURL() → "https://staging.example.com/User/"


then:

LOGIN_USER() → "https://staging.example.com/User/user_login"

🔧 How They Fit Into the System

The flow looks like:

AsyncStorage
→ updates apiURL
→ BASEURL() uses apiURL
→ endpoints like LOGIN_USER() build final URLs

This makes all endpoints dynamic and environment-aware.

🧼 Cleaner & More Scalable Version (Optional)

If you want cleaner code, you could structure it like:

const endpoint = (path) => () => `${BASEURL()}${path}`;

export const LOGIN_USER     = endpoint('user_login');
export const SOCIAL_LOGIN   = endpoint('social_login');
export const SIGNUP_STEPONE = endpoint('user_signup_step1');


Same result — shorter & maintainable.

📌 Summary
Function	Returns
LOGIN_USER()	<apiURL>/User/user_login
SOCIAL_LOGIN()	<apiURL>/User/social_login
SIGNUP_STEPONE()	<apiURL>/User/user_signup_step1

They are functions so the URLs always match the current API base URL, even if it changes during runtime.

If you want, I can also help you:

✅ Auto-generate endpoints
✅ Validate or sanitize URLs
✅ Create a central API client that updates itself automatically