[Start]
   |
   v
[Render OtpVerification Component]
   |
   v
[Initialize States]
  - loading = false
  - isDoctorSignupSuccessAlertVisible = false
   |
   v
[Read Props: userEmail, navigatedFrom, userDetails, sendOtpOn]
   |
   v
[Setup useForm with default OTP if in dev/test mode]
   |
   v
[useEffect: Autofill OTP if available]
   |
   v
[Render Form UI]
   - Email (readonly)
   - OTP Input
   - Verify Button
   - Back Button
   |
   v
[User Presses "Verify"]
   |
   v
[onVerifyPress Executed]
   |
   v
[Set loading = true, Keyboard.dismiss()]
   |
   v
[Build updatedUserDetails]
   - Add email and OTP
   - Upload profile_picture (if SIGNUP)
   - Upload insurance_certificate (if SIGNUP & doctor)
   |
   v
[Send POST Request]
   - URL = login or register based on navigatedFrom
   |
   v
[If LOGIN]
   -> Save PREVIOUSLY_LOGGED_IN_ROLE
   |
   v
[If SIGNUP or LOGIN and not optometrist OR not approved]
   -> Show "Signup Successful" Alert
   |
   v
[Else]
   -> Save USER_DATA
   -> Save SESSION_TOKENS
   -> Update Redux Session Tokens
   -> Update Redux User Details
   |
   v
[Check for Biometric Availability]
   |
   v
[IF biometric NOT available OR already setup]
   -> Navigate to DoctorNavigator OR OptometristNavigator
   |
   v
[ELSE]
   -> Navigate to SetBiometricAuthentication screen
   |
   v
[End]
