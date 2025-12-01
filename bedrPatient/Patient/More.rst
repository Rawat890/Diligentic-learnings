Start
 │
 ├─ Render <Wrapper> with header titled "More"
 │
 ├─ Inside Wrapper:
 │    ├─ Show Card with options:
 │    │    ├─ "Your Data" → navigate to SCREEN_NAMES.YourData
 │    │    ├─ "Settings" → navigate to SCREEN_NAMES.Settings
 │    │    └─ "Logout" → Set logoutAlertVisible = true
 │    │
 │    └─ If logoutAlertVisible is true → Show <Alert>
 │         ├─ Button1: Cancel → logoutAlertVisible = false
 │         └─ Button2: Logout → Call handleLogout()
 │
 ├─ handleLogout():
 │    ├─ Set loading = true
 │    ├─ Hide alert
 │    ├─ Send GET request to logout endpoint (with refresh token)
 │    ├─ If successful:
 │    │    ├─ Close socket connection
 │    │    ├─ Set socket.connected = false
 │    │    ├─ Dispatch RESET_STORE
 │    │    └─ Remove local user data
 │    ├─ If error 403:
 │    │    ├─ Clear local user data
 │    │    └─ Dispatch RESET_STORE
 │    └─ Set loading = false
 │
 ├─ If loading = true → Show <Loader />
 │
 └─ End
