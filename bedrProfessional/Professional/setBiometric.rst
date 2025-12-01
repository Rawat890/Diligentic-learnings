[Start]
   |
   v
[Render SetBiometricAuthentication Screen]
   |
   v
[Extract Params]
  - id
  - role
  - access_token
  - refresh_token
  - public_key
  - private_key
  - isFromSettings
   |
   v
[User Sees "Enable Biometric" and "Maybe Later" Options]
   |
   v
+------------------------------+
|      User Makes Choice       |
+------------------------------+
     |                         |
     |                         |
     v                         v
[Enable Biometric]       [Maybe Later Pressed]
     |                         |
     v                         v
[Check Sensor Availability]   [Check if Keys Exist]
     |                         |
     |                         v
[If available → Prompt User]  [Delete Keys (if exist)]
     |                         |
     v                         v
[User Confirms?]              [Continue]
     |                         |
   Yes/No                      |
     |                         |
     v                         |
[If YES → Create Keys]         |
     |                         |
[Get Device ID]                |
     |                         |
[Send Request to /addBiometrics]
     |
     v
[Success?]
     |
     v
[If isFromSettings]
     → Show success message
     → Navigate back
     |
     v
[Else → Save Session Tokens]
     |
     v
[Redirect based on role]
   - Optometrist → OptometristNavigator
   - Doctor → DoctorNavigator
   |
   v
[End]
