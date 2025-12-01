+-----------------------------------+
| User taps "Use Biometrics for     |
| Login" button                     |
+-----------------------------------+
                  |
                  v
+--------------------------------+
| Get Unique Device ID          |
| DeviceInfo.getUniqueId()      |
+--------------------------------+
                  |
                  v
+--------------------------------+
| Check if biometric keys exist |
| rnBiometrics.biometricKeysExist |
+--------------------------------+
          |
     +----+----+
     |         |
 [No Keys]  [Keys Exist]
     |         |
     v         v
 (Return)   Proceed →
               |
               v
+----------------------------------------------+
| Prompt biometric auth with createSignature   |
| - promptMessage: "login"                     |
| - payload: uniqueId                          |
+----------------------------------------------+
               |
         +-----+-----+
         |           |
     [Failure]    [Success]
         |           |
         v           v
     (Return)     Continue →
                     |
                     v
         +--------------------------+
         | setLoading(true)         |
         +--------------------------+
                     |
                     v
+--------------------------------------------+
| Get FCM token via messaging().getToken()   |
+--------------------------------------------+
                     |
                     v
+----------------------------------------------------------+
| Prepare and send request to URLS.verifyBiometrics        |
| - device_id: uniqueId                                    |
| - signature: biometric signature                         |
| - device_type: Platform.OS                               |
| - device_token: FCM token                                |
+----------------------------------------------------------+
                     |
                     v
+----------------------------------------------------------+
| On Success:                                              |
| - Store USER_DATA                                        |
| - Store SESSION_TOKENS (access, refresh, public/private)|
| - Dispatch updateSessionTokens                           |
| - setLoading(false)                                      |
| - Navigate to PatientNavigator                           |
+----------------------------------------------------------+


catch block-->
+-------------------------+
| Show error message      |
+-------------------------+
            |
+-------------------------+
| setLoading(false)       |
+-------------------------+
