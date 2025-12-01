Component flow overview -->
 ┌────────────┐
 │ App Loads  │
 └────┬───────┘
      ▼
 ┌────────────────────────────────────────────────────────┐
 │ useEffect (onMount):                                   │
 │  - dispatch(getAllChats)                               │
 │  - dispatch(fetchClinics)                              │
 │  - request notification permissions                    │
 │  - setup Firebase listeners (foreground, background)   │
 │  - handle initial notification press                   │
 │  - listen to AppState changes                          │
 └────┬────────────────────────────────────────────────────┘
      ▼
┌────────────────────────────┐
│ useFocusEffect (on focus): │
│ - refreshUserData()        │
│ - getUnavailableDates()    │
│ - dispatch(getInProgressFile())│
└────────────┬───────────────┘
             ▼
      ┌──────────────────────┐
      │ refreshUserData():   │
      │ - getTodaysClinics() │
      │ - dispatch(getUserData()) │
      └──────────────────────┘



Socket.io flow -->

useEffect: [userDetails.chat_decryption_key, tokens.refreshToken]

 ┌────────────┐
 │ socket init│
 └────┬───────┘
      ▼
 ┌───────────────────────────────────────┐
 │ socket.emit("/chat/online-users")     │
 │   → dispatch(setOnlineUsers(...))     │
 └────┬──────────────────────────────────┘
      ▼
 ┌─────────────────────────────────────────────┐
 │ socket.on(...) event listeners:             │
 │                                             │
 │ - /notifications-sent-to-user → refresh     │
 │ - /file-submitted-* → getTodaysClinics      │
 │ - /chat/get-online-users → setOnlineUsers   │
 │ - /chat/on-new-message-to-refresh-chats     │
 │ - /hide-or-show-patient-file-category       │
 │ - /update-patient-file-sort-by              │
 │ - /doctor-reader-unavailable-dates          │
 └─────────────────────────────────────────────┘


Push Notifications flow -->

useEffect (on mount):

 ┌────────────────────────────┐
 │ messaging().onMessage()    │◄───── Foreground notification
 └────┬───────────────────────┘
      ▼
 - if notification.data.type === update_notification_badge
     → refreshUserData()

 ┌──────────────────────────────┐
 │ messaging().setBackground... │
 │ messaging().onNotification.. │◄───── Background notification
 │ messaging().getInitial...    │
 └────┬─────────────────────────┘
      ▼
 - handleOnNotificationPress(data)
     → navigate to correct screen


Availabilty toggle -->
User toggles Switch → onValueChange()

if (currently unavailable):
  - deleteDoctorReaderUnavailabilityDates()
else:
  - addDoctorReaderUnavailableDates()

getUnavailableDates() is called after both to refresh


UI rendering -->
Home.tsx return JSX:

 ┌────────────────────────────────────────────┐
 │ <Wrapper>                                  │
 │   ├── Welcome Header                       │
 │   ├── Calendar + Availability Switch       │
 │   ├── Clinics List (from getTodaysClinics) │
 │   ├── Completed Files (if flag enabled)    │
 │   ├── Readers & Consultants                │
 │   └── BottomSheet (if urgent alert exists) │
 └────────────────────────────────────────────┘


Quick Timeline -->
1. App launches → useEffect fires
2. Notification permissions requested
3. Socket connects (if keys/token exist)
4. useFocusEffect runs on screen focus
5. Clinics and availability are fetched
6. AppState listener refreshes on return
7. Notification received → triggers refresh or navigation
8. Socket events push real-time updates
9. UI re-renders with new state
