Urls using - 
addBiometrics
verifyBiometrics
deleteBiometrics


available -- is the device has support to Biometrics'

SetBioMetrics screen -->

     +----------------------------+
     | User taps a button:       |
     | - "Enable Biometric"      |
     | - OR "Maybe Later"        |
     +----------------------------+
            /           \
           /             \
    [Enable Biometric] [Maybe Later]
           |                |
           v                v


ENABLE BIOMETRICS FOR USER --->

try block ->
+-------------------------------+
| Create ReactNativeBiometrics |
+-------------------------------+
            |
            v
+------------------------------+
| Prompt biometric auth        |
| message: "confirmBiometrics"|
+------------------------------+
            |
   +--------+--------+
   |                 |
[Failure]         [Success]
   |                 |
   v                 v
(Return)     +----------------------+
             | Generate Biometric   |
             | Public/Private Keys  |
             +----------------------+
                     |
                     v
             +--------------------+
             | Get Device Unique  |
             | ID using DeviceInfo|
             +--------------------+
                     |
                     v
             +--------------------------+
             | Send POST request to     |
             | URLS.addBiometrics       |
             | with user_id, public_key,|
             | device_id                |
             +--------------------------+
                     |
                     v
     +-------------------------------+
     | Show success message:         |
     | "biometricsAddedSuccessfully" |
     +-------------------------------+
                     |
         +-----------+------------+
         |                        |
 [isFromSettings = true]   [false]
         |                        |
         v                        v
+-------------------+     +-----------------------------+
| Call goBack()     |     | Dispatch updateSessionTokens|
+-------------------+     | Navigate to PatientNavigator|
                          +-----------------------------+


  catch block -->

        +---------------------------+
        | Show error message        |
        +---------------------------+
                    |
        +---------------------------+
        | Set loading to false      |
        +---------------------------+
                    |
        +------------------------------+
        | Check if biometric keys exist|
        +------------------------------+
                    |
            +-------+--------+
            |                |
        [Yes]              [No]
            |                |
            v                v
+------------------+     (No action)
| Delete keys      |
+------------------+


MAY BE LATER FLOW -->
+-----------------------------+
| Create ReactNativeBiometrics|
+-----------------------------+
            |
            v
+------------------------------+
| Check if biometric keys exist|
+------------------------------+
            |
     +------+---------+
     |                |
   [Yes]             [No]
     |                |
     v                v
+---------------+  (Continue)
| Delete keys   |
+---------------+
            |
            v
+-------------------------------+
| Check isFromSettings          |
+-------------------------------+
        |
  +-----+-----+
  |           |
[true]      [false]
  |           |
  v           v
goBack()   Dispatch updateSessionTokens
           Navigate to PatientNavigator
