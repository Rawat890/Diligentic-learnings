Below is a clean, organized, human-friendly list of every function in your Home component, including:

✅ What it does
✅ Why it’s needed
✅ How it fits into the Home screen’s workflow

✅ Full Function Reference for Home extends PureComponent
🔹 1. Constructor
What it does

Initializes state variables

Sets up socket listeners (offlineUser)

Initializes Crashlytics + Analytics

Creates videoRef

Why it’s used

This sets up the Home screen environment before the UI renders, ensuring:

Socket communication starts

User tracking begins

Initial app state is defined

🔹 2. componentDidMount
What it does

Runs after the component mounts:

Clears FastImage cache on blur

Loads user data + event reservation

Sets AppState listeners

Sets deep link listener

Fetches notifications & messages

Sets push notification listeners

Checks permissions

Subscribes on navigation focus

Syncs contacts

Shows review prompt

Starts location listener

Registers socket event likeCommentPost

Why it’s used

This function initializes everything the Home screen needs to operate once visible.
It’s the “main setup”.

🔹 3. componentWillUnmount
What it does

Removes:

Navigation listeners

Notification listeners

AppState listener

Deep link listener

Socket listeners

Why it’s used

Prevents memory leaks and listeners firing after the screen is closed.

🔹 4. shouldSyncContacts

Checks if the device can read contacts.

If iOS → sync directly

If Android → request permission

If denied → show popup

Why

To safely sync contacts only when allowed.

🔹 5. syncContacts

Runs actual syncing by calling:
this.props.syncMyContacts()

Why

To upload user's contacts to backend for features like:

Finding friends

Contact-based suggestions

🔹 6. onLikeCommentPost

Updates home feed UI when a comment is liked via socket.

Why

To keep feed in sync realtime.

🔹 7. handleLocationChange

Triggered when location updates.
Fetches:

New home feeds

Access token refresh with location

Why

Feeds depend on user location.

🔹 8. handleStripeDeepLink

Handles Stripe onboarding link.

Why

Stripe onboarding sometimes returns the user to the app via URL.

🔹 9. handleUserWentOffline

Called when socket emits offlineUser.

Why

Updates list of online users.

🔹 10. getInitialAppNotification

Checks:

Initial Firebase notification

Initial Notifee notification

If found → navigate accordingly.

Why

If user opens app by tapping a notification from "killed state", it must navigate correctly.

🔹 11. checkNotificationsPermission

Requests push permissions (iOS + Android).
If allowed → updates device token.

Why

Ensures device receives push notifications.

🔹 12. listNotification

Fetches app notifications and counts unread ones.

Why

To show unread badge.

🔹 13. listMessage

Gets chat messages.

Why

Message list is required for message preview or counters.

🔹 14. setNotificationListener

Sets:

Firebase foreground notification listener

Notifee foreground event listener

Handles:

New messages

Badge counters

Notification taps

Why

Manages all in-app notification behavior.

🔹 15. listOtherUser

Fetches user info when notification contains only basic data.

Why

Needed to open "messageListing" when notification refers to a user ID but not full details.

🔹 16. navigateOnNotification

Huge switch-case navigation handler.

Why

Whenever a notification comes in, this decides what screen to open:

Posts

Profile

Feeds

Messages

Orders

Events

Reviews

Shares

🔹 17. moveToNewScreen

Wrapper for navigation with scrollToTop function passed.

🔹 18. checkIfDynamicLinkOpenedByGuestUser

Checks cached dynamic link data.

Why

If guest clicked a dynamic link before login → app must navigate after login.

🔹 19. _handleAppStateChange

When app comes to foreground:

Checks OTA updates

Checks cached dynamic links

Re-checks permissions

Why

App needs fresh data & permissions when resumed.

🔹 20. moveToCachedDynamicLinkScreen

Routes user based on stored dynamic link type:

Profile

Offer

Request

Event QR

Guest list

Reviews

Order invitation

Why

Dynamic links must work even after cold start.

🔹 21. handleGuestReviewDeepLink

Loads review modal for guest review.

Why

Allows host ↔ guest review system.

🔹 22. onGuestReviewAdded

Submits guest review + hides modal.

Why

Completes review workflow.

🔹 23. checkPermission

Checks location permission and updates redux.

Why

Many features use location (feed, offers, events).

🔹 24. scrollToTopFunction

Scrolls feed to top.

Why

Linked to header tap or navigation.

🔹 25. getUserLocation

Tries to get device location safely.

🔹 26. getUserData

Refreshes access token with location.

Why

Some APIs expect latest location on token.

🔹 27. closeLoading

Sets loading state to false.

🔹 28. getEventReservation

Loads stored event check-in data.

Why

Used for event QR guest check-in flow.

🔹 29. onCancelReview

Closes review modal.

🔹 30. checkAndShowReviewPrompt

Shows in-app review prompt for specific test users.

🔹 31. showCustomReviewPrompt

Displays custom alert box.

🔹 32. requestReview

Triggers OS native in-app review flow.

🔹 33. showContactsPermissionDeclarationModal / hideContactsPermissionDeclarationModal

Toggles modal.

🎯 Final Notes

This Home component handles a massive amount of app logic:

Core responsibilities

Real-time socket updates

Push notifications

Dynamic links

Location-based feed

Contact syncing

Navigation control

Review prompt

Stripe deep links

Event check-in