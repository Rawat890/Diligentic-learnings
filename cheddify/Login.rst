User opens login screen
          ↓
Load user from AsyncStorage?
          ↓
Yes? → navigateToOnboardingStep
No? → show login form
          ↓
User enters email/password OR social login
          ↓
Validate input
          ↓
API request (login or socialLogin)
          ↓
If signup incomplete → onboarding screen
Else → MainNavigator
          ↓
Set analytics + crashlytics
