Start
  |
  v
Check if:
(loginAs === optometrist AND selectedPracticeId exists)
OR
(loginAs !== optometrist)
  |
  |-- No --> Show Alert (setIsAlertVisible(true)) --> End
  |
  |-- Yes -->
          |
          v
   Get Device Unique ID (DeviceInfo.getUniqueId)
          |
          v
   Check if biometric keys exist (biometricKeysExist)
          |
          |-- No --> End
          |
          |-- Yes -->
                |
                v
        Create biometric signature (createSignature)
                |
                |-- success === false --> End
                |
                |-- success === true -->
                      |
                      v
          If loginAs === optometrist:
            dispatch(optometristPractice)
          |
          v
        setLoading(true)
          |
          v
        Get FCM token (messaging().getToken)
          |
          v
        Build request data:
          if optometrist:
            device_id, signature, device_type, device_token, practice_id
          else:
            device_id, signature, device_type, device_token
          |
          v
        Send POST request to URLS.verifyBiometrics
          |
          v
        On success:
          - Save user data in secure storage
          - Save session tokens in secure storage
          - Dispatch updateSessionTokens
          - setLoading(false)
          |
          v
        If role === "optometrist"
            --> Navigate to OptometristNavigator
        Else
            --> Navigate to DoctorNavigator
          |
          v
        End



Additonal code-->

Start
  |
  v
User lands on Login Screen
  |
  v
+-------------------------------+
| Form Fields:                 |
| - loginAs (role picker)      |
| - email                      |
| - practice (if optometrist)  |
| - password (if next pressed) |
| - radio button (OTP via)     |
+-------------------------------+
  |
  v
User presses:
  |
  |-- If optometrist & not next pressed --> Validate + Show "Next" Button
  |
  |-- Else (or "Next" pressed) --> Show password + radio options + "Login"
  |
  v
User presses biometric login?
  |
  |-- Yes --> Call `onLoginWithBiometricsPress`
  |            |
  |            |--> If optometrist AND practice selected OR other role
  |            |      |
  |            |      |--> Get deviceId
  |            |      |--> Check biometric key exists
  |            |      |--> Create biometric signature
  |            |      |--> Build payload (incl. practice_id if opto)
  |            |      |--> Send API request
  |            |      |--> On success: Save tokens, Navigate
  |            |      |--> On error: Show error
  |            |
  |            |--> If optometrist but NO practice selected:
  |                   - Show Alert
  |
  |-- No --> Press normal Login button
               |
               |--> Call `onLoginPress` (not shared, but likely similar)
  |
  v
Other UI Options:
  - Forgot password --> Navigate to ForgotPassword
  - Register now --> Navigate to Signup
  - Picker modals (Practice, Profession)
  - Alert shown if validation fails
