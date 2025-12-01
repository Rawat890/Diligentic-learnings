WrapperWithGradient
 └── Header (Back button, title: "Signup")
 └── KeyboardAwareScrollView
      └── Card
           ├── LargeButtonText ("Welcome")
           ├── NamingSmallest ("Enter details")
           ├── ProfileImagePlaceholder
           ├── SignupAs Dropdown
           ├── Conditionally Rendered:
           │    ├── GOC Number (Optometrist)
           │    └── GMC Number, Salutation, Insurance fields (Doctor)
           ├── TextInputs for:
           │    ├── First Name
           │    ├── Last Name
           │    ├── Email Address
           │    ├── Mobile Number (with country picker)
           │    └── Date of Birth (with MaskInput + DatePicker trigger)
           .............

On sign up press-->
User taps submit → Validate form → Gather device token →
Build `userDetails` object → Send OTP API Request → 
If success:
   ↳ Show success message (SMS or Email)
   ↳ Navigate to OtpVerification screen
Else:
   ↳ Show error / dispatch network error


Media upload ->
User taps "Upload Photo" →
 → setIsPickerVisible(true)
 → Choose: Camera or Gallery
 → On success: set form value `profilePicture`
 → On permission error: Show Alert, Allow Settings


If "signupAs" === doctor:
   → Show GMC Number
   → Show Salutation Picker
   → Show Insurance Certificate
Else if "signupAs" === optometrist:
   → Show GOC Number
