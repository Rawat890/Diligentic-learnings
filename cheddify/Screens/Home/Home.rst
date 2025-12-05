HOME SCREEN - HOME.
constructor(props) {
  super(props);
✔ What it does:
Calls super(props) to make this.props available (required in React class components).



const {
  userData,
  userData: {_id},
} = props;
✔ What it does:
Destructures props.

Pulls userData out of props.

Also extracts _id from userData.

Example result:
this.props.userData → { name: "John", _id: "123", ... }
userData → same object
_id → "123"


this.state = {
  eventReservation: null,
  isTicketSeen: false,
  guestOfferReviewData: null,
  isGuestReviewVisible: false,
  isDataLoading: false,
  onlineUser: [],
  unreadNotifs: 0,
  shouldShowContactsPermissionDeclarationModal: false,
  showVideo: false,
};
✔ What it does:
Initializes local component state:

State	Meaning
eventReservation	for showing reservation details
isTicketSeen	if user opened their event ticket
guestOfferReviewData	guest user’s review data (after dynamic link)
isGuestReviewVisible	controls review popup
isDataLoading	general loading flag
onlineUser	list of online users (from socket)
unreadNotifs	unread notification count
shouldShowContactsPermissionDeclarationModal	show permissions modal
showVideo	flag to show the video

this.videoRef = createRef();
✔ What it does:
Creates a reference to a video component so code can play/pause.

SOCKET INITIALIZATION
socketServices.initializeSocket({userId: _id, access_token: this.props.access_token});
✔ Establishes a WebSocket connection (e.g., using Socket.IO).
✔ Sends the user's ID + token for authentication.

socketServices.on('offlineUser', this.handleUserWentOffline);
✔ Listens for "offlineUser" socket event
✔ When triggered → calls handleUserWentOffline().

Crash & Analytics setup


crashlytics().setUserId(_id);
analytics.identify(_id);
analytics.registerDefaultSuperProperties(userData);
✔ Crashlytics: associate crash logs with this user
✔ Analytics: identify user for tracking
✔ Registers default analytics properties (age, name, gender, etc.)

2. componentDidMount() – Runs once after component loads
This is a big method — here is step-by-step:

A. Clear image cache when screen loses focus


this._blurUnsubscribe = this.props.navigation.addListener('blur', () => {
  FastImage.clearMemoryCache();
});
✔ When user leaves the Home screen → clears image cache.

B. Fetch initial data


this.getUserData();
this.getEventReservation();
✔ Loads user data
✔ Loads event/reservation details

C. Track app state (foreground/background)


this.appState = AppState.currentState;
this.checkIfDynamicLinkOpenedByGuestUser();

this.appStateListener = AppState.addEventListener(
  'change',
  this._handleAppStateChange,
);
✔ Stores current app state
✔ Checks if app opened through a guest dynamic link
✔ Listens for app state changes (active → background)

D. Listen for Stripe deep links


this.stripeDeepLink = Linking.addEventListener(
  'url',
  this.handleStripeDeepLink,
);
✔ Listens for URLs like:
myapp://stripe/onboarding?some-token

When triggered → calls handleStripeDeepLink.

E. Notifications & Messages


this.listNotification();
this.listMessage();
this.getInitialAppNotification();
this.setNotificationListener();
✔ Fetch old notifications
✔ Fetch messages
✔ Handle notification that opened the app
✔ Start listeners for new push notifications

F. Contact permissions


this.checkPermission();
✔ Checks OS-level permissions (location, contacts, etc.)

G. Refresh when screen becomes focused


this.unsubscribeNavigationListener = this.props.navigation.addListener(
  'focus',
  () => {
    this.getUserData(true);
    this.getEventReservation();
    this.checkNotificationsPermission();
  },
);
✔ When screen is focused again:

Refresh user data

Refresh reservation

Check notification permissions

H. Sync local contacts with server


this.shouldSyncContacts();
✔ Checks if the user has granted contact permissions
✔ If yes → syncs contacts
✔ If not → shows permission modal

I. Show an app review prompt if needed


this.checkAndShowReviewPrompt();
✔ Conditions (e.g., user used app more than 10 times)
✔ Shows review prompt modal

J. Start listening to location changes


await startLocationListener(this.handleLocationChange);
✔ Listens for GPS updates
✔ On change → fetch new home feed & push location to server

K. Socket listener for comment likes


socketServices.on('likeCommentPost', this.onLikeCommentPost);
✔ If someone likes a comment → update home feed

3. componentWillUnmount()
Runs when Home screen is removed.



this._blurUnsubscribe();
this.stripeDeepLink.remove();
this.appStateListener.remove();
this.unsubscribeFromNotificationListener();
this.unsubscribeNotifeeForegroundListener();
this.unsubscribeNavigationListener();

socketServices.removeAllListener();
✔ Cleans up all listeners
✔ Closes socket listeners
✔ Prevents memory leaks

4. shouldSyncContacts()


shouldSyncContacts = async () => {
  if (getPlateForm()) {
    this.syncContacts();
    return;
  }
✔ If platform is iOS → skip permission check → sync contacts immediately
(Android requires manual permission checking)

Check Android contacts permission


const isGranted = await PermissionsAndroid.check(
  PermissionsAndroid.PERMISSIONS.READ_CONTACTS,
);
✔ Checks if READ_CONTACTS permission is already granted

If granted → sync contacts


if (isGranted) {
  this.syncContacts();
  return;
}
If not → show modal


this.showContactsPermissionDeclarationModal();
✔ Modal explains why the app needs contact permission
✔ User will then manually grant it

5. syncContacts()


syncContacts = () => {
  this.hideContactsPermissionDeclarationModal();
  this.props.syncMyContacts();
};
✔ What it does:
Hides the permission modal

Calls Redux action syncMyContacts() that:

fetches the phone’s contact list

uploads contacts to server

updates app state

6. onLikeCommentPost (socket event)


onLikeCommentPost = async data => {
  this.props.updateHomeFeedStatus({data});
};
✔ Whenever someone likes a post/comment → update home feed UI
✔ Uses Redux action updateHomeFeedStatus

7. handleLocationChange()


handleLocationChange = async location => {
  try {
    const currentLocation = await getCurrentLocation();
✔ Gets current GPS coordinates
✔ location parameter is ignored — it re-fetches

If location is available:


this.props.getHomeFeeds({
  loadMore: false,
  refreshing: false,
  currentLocation,
  resetSkipCount: true,
});
✔ Fetch new home feed posts based on location
✔ Resets pagination counter

Send location to server


const query = {
  lat: currentLocation?.latitude,
  lng: currentLocation?.longitude,
};
await actions.accessToken(query);
✔ Probably updates user’s current location on server
✔ Or refreshes some access token using location

Catch error


} catch (error) {
  console.error('Error fetching current location:', error);
}


DETAILED EXPLANATION (PART 2)

We continue from handleStripeDeepLink → getInitialAppNotification.

8. handleStripeDeepLink
handleStripeDeepLink = url => {
  if (url?.url && url.url.includes(STRIPE_ONBOARDING_DYNAMIC_LINK)) {
    actions.accessToken({});
  }
};

✔ What it does:

url is the deep link event object from React Native Linking.
Example:

{ url: "myapp://stripe/onboarding?some_token=123" }


It checks:

if (url?.url && url.url.includes(STRIPE_ONBOARDING_DYNAMIC_LINK))


Meaning:

The URL exists

The URL contains a specific string
(e.g., "stripe_onboarding")

If true → it calls:

actions.accessToken({})


✔ Most likely triggers an API call to refresh Stripe onboarding session or verify onboarding.

9. handleUserWentOffline
handleUserWentOffline = userId => {
  if (!userId) {
    return;
  }
  userWentOffline(userId);
};

✔ What it does:

Triggered by socket event "offlineUser".

Checks if the event provided a userId.

Calls a helper function userWentOffline(userId) which most likely:

Updates Redux store

Marks the user as “offline”

Updates UI (green dot removed, etc.)

10. getInitialAppNotification
getInitialAppNotification = async () => {
  try {
    const initialFirebaseNotification =
      await messaging().getInitialNotification();

✔ What it does:

When the app is opened by tapping a notification, this function captures that first notification.

Step A — Check Firebase notification (FCM)

messaging().getInitialNotification() returns:

A notification object if the app was opened from a Firebase push

null otherwise

Example:

{
  data: { type: "chat", roomId: "abc" }
}


Then:

if (initialFirebaseNotification?.data) {
  this.navigateOnNotification(initialFirebaseNotification.data);
}

✔ If that notification contains routing info → navigate immediately.

Step B — Check Notifee (local notifications)
const initialNotificationNotifee = await notifee.getInitialNotification();
Notifee handles:

Local notifications
Foreground notifications
Background notifications

Then:

if (initialNotificationNotifee?.notification?.data) {
  this.navigateOnNotification(
    initialNotificationNotifee.notification.data,
  );
}

✔ Same logic:
If user tapped a Notifee notification → navigate accordingly.

Error handling
} catch (error) {}

✔ Silently ignores errors
(Not recommended, but used here to avoid crashes.)


✅ FUNCTION-BY-FUNCTION EXPLANATION
1. checkNotificationsPermission
checkNotificationsPermission = async () => {
  try {
    let notificationsPermissionEnabled = false;

✔ What is happening here?

A boolean flag notificationsPermissionEnabled is created.

It will become true only if the user grants notification permissions.

✔ iOS permission check
if (Platform.OS === 'ios') {
  const authStatus = await messaging().requestPermission();
  const enabled =
    authStatus === messaging.AuthorizationStatus.AUTHORIZED ||
    authStatus === messaging.AuthorizationStatus.PROVISIONAL;

  if (enabled) {
    notificationsPermissionEnabled = true;
  }
}

Step-by-step:

If the device is iOS, request notification permission.

requestPermission() returns an authorization status:

AUTHORIZED → user fully allowed notifications

PROVISIONAL → allowed silently

If either is true → set flag to true.

✔ Android permission check
if (Platform.OS === 'android') {
  const {status} = await permissions.requestNotifications(['alert', 'sound']);
  if (status === 'granted') {
    notificationsPermissionEnabled = true;
  }
}

Step-by-step:

On Android, use permissions.requestNotifications.

Request "alert" + "sound".

If user grants → set flag = true.

✔ If permission enabled → get FCM token
if (notificationsPermissionEnabled) {
  const fcmToken = await messaging().getToken();
  await updateDeviceToken({device_token: fcmToken});
}


What this does:

Retrieves FCM device token from Firebase Messaging.

Calls an API (updateDeviceToken) to store token on server.
So the backend can send push notifications to this device.

✔ Error handling
} catch (error) {
  console.log('checkNotificationsPermission -> error', error);
}


If anything fails, log the error.

2. listNotification
listNotification = () => {
  this.setState({isDataLoading: true});


✔ Shows a loading state.

✔ Fetch notification list
actions
  .listNotification()
  .then(res => {


This calls an API via actions.listNotification().
res.data contains the list of notifications returned from backend.

✔ Compute unread notifications
const unreadNotifs = [...res.data.filter(i => i.is_read === false)];
this.setState({unreadNotifs: unreadNotifs.length});


Filters notifications where is_read === false

Updates state with the unread count

✔ Turn off loading
this.closeLoading();

✔ On error
.catch(error => {
  showSnackBar(error.message);
  this.closeLoading();
});


If API fails:

Show error message via snack bar

Stop loading

3. listMessage
listMessage = async () => {
  const {userData} = this.props;
  const params = {user_id: userData._id};
  const {data} = await actions.listMessages(params);
};

✔ What this function does:

Gets current user ID.

Sends { user_id: ... } to API listMessages.

The result (data) contains message threads or messages.

Note:
The function doesn’t update UI or save the message. Possibly incomplete or they rely on Redux inside the action.

4. setNotificationListener

This is one of the most important functions.
It handles foreground push notifications using Firebase Messaging + Notifee.

✔ Create a notification channel (Android)
const channelId = await notifee.createChannel({
  id: 'cheddify_channel',
  name: 'Cheddify',
  importance: AndroidImportance.HIGH,
});


For Android:

Every notification must be assigned a channel.

HIGH importance shows heads-up banner.

✔ Listen for incoming FCM messages while app is in foreground
this.unsubscribeFromNotificationListener = messaging().onMessage(
  async remoteMessage => {


This fires when:

You receive push notification while the app is open.

✔ Ignore some notifications (if chatting)

If user is currently chatting with someone, the app avoids showing notification popups for messages coming from that same person.

if (remoteMessage.data.type === NEW_MESSAGE && remoteMessage.data.sender_id === this.props.chattingWith ){
  return;
}


Similarly for LIKE_MESSAGE and silent push:

if (remoteMessage.data.type === LIKE_MESSAGE && remoteMessage.data.liked_by === this.props.chattingWith ){
  return;
}

if (remoteMessage.data.type === SILENT_PUSH) {
  notifee.setBadgeCount(parseInt(remoteMessage.data.badgeCount));
  return;
}

✔ Show badge + display notification
notifee.incrementBadgeCount();
notifee.displayNotification({
  title: remoteMessage.data.title,
  body: remoteMessage.data?.body ?? remoteMessage.data.message,
  ios: { sound: 'default' },
  data: remoteMessage.data,
  android: {
    channelId,
    smallIcon: 'chedda',
    pressAction: { id: 'default' },
  },
});


Breakdown:

Increase the unread count badge.

Show a local notification using Notifee.

Proper sound + icon + press action.

✔ Refresh user data
this.getUserData(true);


The app refreshes user stats (unread messages, etc.).

✔ Listen to Notifee foreground events
this.unsubscribeNotifeeForegroundListener = notifee.onForegroundEvent(
  ({type, detail}) => {


Fires when user interacts with the notification.

✔ If user taps the notification
if (type === EventType.PRESS) {
  if (detail.notification.data.type === 'AFFILIATION_AMOUNT_SENT') {
    actions
      .getWalletTransaction(...)
      .then(...)
      .catch(...)
  }
  this.navigateOnNotification(detail.notification.data);
}


If notification pressed → navigate to correct screen
(such as message, post, offer, order, feed, etc.)

5. listOtherUser
listOtherUser = async (notificationData) => {
  try {
    const { data } = await actions.otherFeedApi(notificationData?.reacted_by);


Fetches user details via API → gives profile data of the user.

✔ Build userDetails object
const userDetails = {
  _id: data.chat_connection_id,
  user_info: {
    profile_pic: data.profile_picture,
    user_id: data._id,
    name: data.name,
    available: data.available,
  },
};


This is formatted specifically for the chat screen.

✔ Navigate to messageListing
navigate("messageListing", {
  flag: data,
  userDetails: userDetails,
});


Opens chat UI.

6. navigateOnNotification

This is a router function.
It decides where the user should be navigated based on notification type.

Breakdown by conditions:

✔ 1. Recommendation received
if (notificationData.type === MADE_RECOMMENDATION) {
  navigate("friendProfile", { user_id: { other_user_id: notificationData.recommended_user_id } });
  return;
}

✔ 2. Chat message reaction
if (notificationData.type === "CHAT_MESSAGE_REACTION") {
  this.listOtherUser(notificationData);
}

✔ 3. Follower / Friend request
if (notificationData.push_type === NEW_FOLLOWER || REQUEST_SENT || MANAGE_FRND_REQUEST) {
  NavigationService.navigate('friendProfile', {
    user_id: {other_user_id: notificationData.sent_by},
  });
}

✔ 4. Feed events (like/comment/new post)
if (notificationData.push_type === FEED_COMMENT || FEED_LIKED || FEED_ADDED) {
  NavigationService.navigate('FavouriteViewListing', { ... });
}

✔ 5. Messages or contact joined
if (notificationData.push_type === NEW_MESSAGE || CONTACT_JOIN_CHEDDA) {
  const userDetails = {
    user_info: {
      ...notificationData,
      user_id: notificationData.msg_sent_by,
      profile_pic: JSON.parse(notificationData?.profile_picture ?? '[]'),
      fromContactJoinedPush: notificationData.push_type === CONTACT_JOIN_CHEDDA,
    },
  };
  NavigationService.navigate('messageListing', {userDetails});
}

✔ 6. App review
if (notificationData.push_type === REVIEW) {
  NavigationService.navigate('profile');
}

✔ 7. Orders (delivery, sales, etc.)

Parses payload → checks if it is a SALES or PURCHASE screen.

✔ 8. Share notifications
if (notificationData.push_type === SHARE) {
  const payload = JSON.parse(notificationData.payload);

  if (payload.share_type === dynamicLinkType.offer) {...}
  if (payload.share_type === dynamicLinkType.profile) {...}
  if (payload.share_type === dynamicLinkType.post) {...}
}


Each share type navigates to:

Offer screen

Profile

Feed listing

✔ 9. Post liked
if (notificationData.push_type === POST_LIKED) {
  NavigationService.navigate('SingleOfferAndRequest', {...});
}

✔ 10. Event Invitation
if (notificationData.push_type === EVENT_INVITATION) {
  const payload = JSON.parse(notificationData.payload);
  NavigationService.navigate('SingleOfferAndRequest', {...});
}

✔ 11. Message Like
if (notificationData.type === LIKE_MESSAGE) {
  const userDetails = {
    _id: notificationData._id,
    user_info: {
      ...notificationData,
      user_id: notificationData.liked_by,
      profile_pic: JSON.parse(notificationData?.profile_picture ?? '[]'),
    },
  };
  NavigationService.navigate('messageListing', { userDetails });
}

7. moveToNewScreen
moveToNewScreen = (id, data) => () => {
  this.props.navigation.navigate(id, {
    user_id: data,
    scrollToTop: this.scrollToTopFunction,
  });
};

✔ What it does:

This is a curried function:

First call gives route ID and data

Second call returns a function to trigger navigation

Used for onPress handlers:

onPress={this.moveToNewScreen('profile', userId)}

8. checkIfDynamicLinkOpenedByGuestUser
checkIfDynamicLinkOpenedByGuestUser = async () => {
  try {
    const cachedDynamicLink = await getItem(CACHED_DYNAMIC_LINK);

    if (!cachedDynamicLink) return;

    this.moveToCachedDynamicLinkScreen(cachedDynamicLink);
  } catch (error) {}
};

✔ What it does:

Reads local storage for saved dynamic link.

If exists → open the related screen.

This is used when app was opened by a dynamic link but user wasn't logged in at that time, so link is processed later.

9. _handleAppStateChange
_handleAppStateChange = nextAppState => {


Triggered whenever app moves between:

active

background

inactive

✔ App moved from background → active
if (
  this.appState.match(/inactive|background/) &&
  nextAppState === 'active'
) {


Checks if:

Old state = inactive OR background

New state = active

Then run code.

✔ Check OTA updates
checkIfNewOTAAvailable();


OTA = Over-the-Air updates for the app.

✔ Check dynamic links again (after delay)
setTimeout(async () => {
  const cachedDynamicLink = await getItem(CACHED_DYNAMIC_LINK);
  if (cachedDynamicLink) {
    this.moveToCachedDynamicLinkScreen(cachedDynamicLink);
  }
}, 500);


Why delay?
To ensure app state is fully restored before navigating.

✔ Check permissions again
this.checkPermission();

Maybe user changed permissions while app was backgrounded.

✔ Update stored app state
this.appState = nextAppState;


✅ FUNCTION-BY-FUNCTION EXPLANATION — PART 3
1. moveToCachedDynamicLinkScreen
moveToCachedDynamicLinkScreen = cachedDynamicLink => {
  const {url: urlType, id} = cachedDynamicLink;
  const {userData} = this.props;

✔ What’s happening?

A dynamic link was previously saved in storage because the user wasn’t logged in.

After login, this function decides where to navigate based on:

The link type: urlType

The link payload: id (could be userId, offerId, eventId, etc.)

✔ A. Profile link
if (urlType === strings.profile) {
  let user_id = { other_user_id: id };
  if (userData._id != id) {
    NavigationService.navigate('friendProfile', { user_id });
  } else {
    // If it's the user's own profile → do nothing
  }
}


If the link points to a profile, navigate to that user’s profile.

But if user clicked their own profile link → skip navigation.

✔ B. Offer or Request link
else if (urlType === strings.offer || urlType === strings.requests) {
  NavigationService.navigate('SingleOfferAndRequest', {
    offerAndRequestId: id,
    offerOrRequest: urlType,
  });
}


Opens Offer / Request detail page.

✔ C. Event QR link
else if (urlType === dynamicLinkType.eventQr) {
  const offerDetails = id.split('&');
  const offerId = offerDetails[0];
  const offerDate = offerDetails[1];

  NavigationService.navigate('EventGuestCheckinByQR', {
    offerId,
    date: offerDate,
  });
}


This link came from scanning an event QR code.

id looks like "offerId&timestamp"

It routes to Event check-in screen.

✔ D. Share Event Dashboard
else if (urlType === dynamicLinkType.shareEventDashboard) {
  const offerDetails = id.split('&');
  const offerId = offerDetails[0];
  const offerDate = offerDetails[1];

  NavigationService.navigate('GuestList', {
    offerId,
    date: moment(parseInt(offerDate, 10)).toISOString(),
  });
}


Navigates to event guest list.

Converts timestamp into ISO format.

✔ E. Guest Review Offer
else if (urlType === dynamicLinkType.guestReviewOffer) {
  this.handleGuestReviewDeepLink(id);
}


Guest needs to leave a review after attending a host’s event.

Opens review modals.

✔ F. Invite Friend To Order
else if (urlType === dynamicLinkType.inviteFriendToOrder) {
  NavigationService.navigate('InvitedOfferDetails', {
    eventInvitationId: id,
  });
}

✔ G. Default → feed post
else {
  let _id = {
    feed_id: id,
    name: strings.empty,
    allSelect: true,
  };

  NavigationService.navigate('FavouriteViewListing', {_id});
}


If no special type matches → treat link as a post link.

✔ H. Clear Cached Link
removeItem(CACHED_DYNAMIC_LINK);


The cached link is removed so it doesn’t re-trigger.

2. handleGuestReviewDeepLink
handleGuestReviewDeepLink = async id => {
  try {
    const offerDetails = id.split('&');
    const offerId = offerDetails[0];
    const offerDate = offerDetails[1];


Dynamic link includes offerId + offerDate.

Example: "43542534&1702030000000"

✔ Fetch offer details
const {data} = await actions.getOfferDetails(offerId);


Backend returns info about the offer/event.

✔ If offer not found → exit
if (!data.length) return;

✔ Save info + show review modal
this.setState({
  guestOfferReviewData: {
    ...data[0].user_info,
    offerId: data[0].post_id,
    offerDate,
  },
  isGuestReviewVisible: true,
});


This shows a bottom sheet/modal where guest writes a review.

3. onGuestReviewAdded

Runs after guest submits review.

✔ Start loading
this.setState({
  isGuestReviewVisible: false,
  isDataLoading: true,
});

✔ Prepare review payload
const {data} = await actions.addReview({
  user_id: this.state.guestOfferReviewData._id,
  comment: reviewData.comment,
  ratings: reviewData.ratings,
  offer_id: this.state.guestOfferReviewData.offerId,
  date: moment(parseInt(this.state.guestOfferReviewData.offerDate, 10)).toISOString(),
});


Payload fields:

user_id = who is being reviewed

comment

ratings

offer_id = event/post ID

date = original event timestamp formatted properly

✔ Show success message
showSnackBar(strings.reviewAddedSuccessfully);

✔ Reset state
this.setState({
  isDataLoading: false,
  guestOfferReviewData: null,
});

4. checkPermission
checkPermission = async () => {
  const hasPermission = await checkLocationPermission();
  this.props.checkLocationPermissionIsGranted(hasPermission);
};


Checks if user granted GPS permission.

Sends result to Redux (checkLocationPermissionIsGranted).

5. scrollToTopFunction
scrollToTopFunction = () => {
  this.scrollTop.scrollToTop();
};


This calls a child component’s scrollToTop() method.
Used by refresh buttons and navigation.

6. getUserLocation
getUserLocation = async () => {
  try {
    const {lat, lng} = await GetLocation();
    return {lat, lng};
  } catch (error) {
    return {lat: 0, lng: 0};
  }
};


Attempts to fetch precise location.

If fails → return default (0,0) to avoid crashes.

7. getUserData
getUserData = async (skipLocationFetch = false) => {
  try {
    let userData = {};

    if (!skipLocationFetch) {
      const location = await this.getUserLocation();
      userData = location;
    }

    await actions.accessToken(userData);
  } catch (error) {}
};

✔ What this does:

Fetches location unless skipLocationFetch===true

Calls actions.accessToken(userData):

This usually updates:

Geo info

Online status

Auth token refresh

User session data

8. closeLoading
closeLoading = () => {
  this.setState({ isDataLoading: false });
};


Stops loading spinner.

9. getEventReservation
getEventReservation = async () => {
  try {
    const eventReservation = await getItem('EVENT_CHECKIN');


Fetch event check-in data stored locally.

✔ Update state based on saved data
if (eventReservation && !this.state.eventReservation) {
  this.setState({eventReservation});
}
if (!eventReservation && this.state.eventReservation) {
  this.setState({eventReservation: null});
}

10. onCancelReview
onCancelReview = () =>
  this.setState({
    isGuestReviewVisible: false,
    guestOfferReviewData: null,
  });


Closes the review modal.

11. checkAndShowReviewPrompt
checkAndShowReviewPrompt = async () => {
  const { userData } = this.props;
  const hasSeenPrompt = await AsyncStorage.getItem("hasSeenReviewPrompt");


Checks if app should display in-app rating popup.

Only shown once.

✔ Show only if:
if (!hasSeenPrompt && userData?.email?.endsWith("@testingcheddify.co")) {
  this.showCustomReviewPrompt();
}


This is likely a test condition (internal testers domain).

12. showCustomReviewPrompt

Shows a native alert asking for feedback.

Alert.alert(
  "Are you liking Cheddify so far?",
  "Your feedback helps us improve.",
  [
    { text: "Not really", style: "cancel" },
    {
      text: "Yes",
      onPress: async () => {
        await AsyncStorage.setItem("hasSeenReviewPrompt", "true");
        this.requestReview();
      },
    },
  ]
);

13. requestReview
requestReview = async () => {
  const isAvailable = InAppReview.isAvailable();
  if (!isAvailable) return;
  await InAppReview.RequestInAppReview();
};


Triggers Google Play / Apple Store in-app review dialog.

14. showContactsPermissionDeclarationModal / hideContactsPermissionDeclarationModal

Self-explanatory:

showContactsPermissionDeclarationModal = () => 
  this.setState({shouldShowContactsPermissionDeclarationModal: true});

hideContactsPermissionDeclarationModal = () => 
  this.setState({shouldShowContactsPermissionDeclarationModal: false});


Used to show popup explaining why contact permission is needed.

15. render()
✔ Key logic:
if (!userData?._id) {
  return null;
}


Meaning:

Don't render UI until userData is available.

✔ Freeze screen if account has high deduction
if (userData?.has_high_deduction) {
  return (
    <Text style={...}>
      {strings.yourAccountIsFreezed}
    </Text>
  );
}


If backend marks account as frozen → show message.

(UI continues below, not provided)
✅ Summary

You now have a fully detailed explanation of EVERY function:

✔ Dynamic link routing
✔ Guest review flow
✔ Permission logic
✔ Event check-in storage
✔ In-app review logic
✔ Location fetching
✔ User data / sync
✔ State updates
✔ Rendering conditions


🔵 FLOWCHART: Home Component (High-Level Overview)

                                        ┌────────────────────────┐
                                        │      APP STARTS        │
                                        └────────────┬───────────┘
                                                     │
                                                     ▼
                                    ┌──────────────────────────────────┐
                                    │    Home Component Constructor    │
                                    └─────────────────┬────────────────┘
                                                     │
                 ┌────────────────────────────────────┼────────────────────────────────────┐
                 │                                    │                                     │
                 ▼                                    ▼                                     ▼
     Initialize sockets                     Set Crashlytics user                 Register Analytics
(with userId + token)                       Set Analytics identify           Register default properties
(socketServices)                                 (user_id)                       (userData object)
                 └───────────────────────────────┬──────────────────────────────┘
                                                 │
                                                 ▼
                        ┌────────────────────────────────────────────┐
                        │         COMPONENT DID MOUNT               │
                        └───────────────────────────┬────────────────┘
                                                    │
       ┌────────────────────────────────────────────┼─────────────────────────────┐
       │                                            │                             │
       ▼                                            ▼                             ▼
Add blur listener                     Load User Data                     Load Event Reservation
(FastImage cache clear)               (getUserData())                    (getEventReservation())

       │                                            │                             │
       ├────────────────────────────────────────────┼─────────────────────────────┤
       ▼                                            ▼                             ▼
Check if dynamic link exists     Listen for AppState change           Stripe Deep Link Listener
(checkIfDynamicLinkOpenedBy...)  (foreground/background)             (Linking.addEventListener)

       │                                            │                             │
       ├────────────────────────────────────────────┼─────────────────────────────┤
       ▼                                            ▼                             ▼
Load Notifications                Load Messages                     Handle App opening notification
(listNotification)                (listMessage)                     (getInitialAppNotification)

       │                                            │                             │
       ├────────────────────────────────────────────┼─────────────────────────────┤
       ▼                                            ▼                             ▼
Set Notification Listener         Check Permissions                 Navigation Focus Event
(setNotificationListener)         (checkPermission)                 (refresh user data on focus)

       │                                            │                             │
       ├────────────────────────────────────────────┼─────────────────────────────┤
       ▼                                            ▼                             ▼
Sync Contacts If Needed        Show Review Prompt (if eligible)       Start Location Listener
(shouldSyncContacts)            checkAndShowReviewPrompt()         (on location → reload feeds)




🔵 FLOW AFTER A DYNAMIC LINK IS DETECTED

                    ┌────────────────────────────────────────┐
                    │  moveToCachedDynamicLinkScreen(link)   │
                    └───────────────────────────┬────────────┘
                                                │
     ┌──────────────────────────────────────────┼──────────────────────────────────────────┐
     │                                          │                                          │
     ▼                                          ▼                                          ▼
If link.type = profile               If link.type = offer/request           If link.type = eventQr
Navigate to user profile             Navigate to offer screen               Extract offerId + date
(unless it’s own profile)            (SingleOfferAndRequest)                 Navigate event check-in

     │                                          │                                          │
     ├──────────────────────────────────────────┼──────────────────────────────────────────┤
     ▼                                          ▼                                          ▼
If type = shareEventDashboard        If type = guestReviewOffer             If type = inviteFriend
Extract offerId + date               Open guest review modal                Navigate to InvitedOffer
Navigate to GuestList                (handleGuestReviewDeepLink)            screen

     │
     ▼
Else → navigate to feed post

Finally:
removeItem(CACHED_DYNAMIC_LINK)


🔵 FLOW OF GUEST REVIEW
                      ┌───────────────────────────────────┐
                      │ handleGuestReviewDeepLink(id)     │
                      └─────────────────────┬─────────────┘
                                            │
                        Extract offerId + date from id
                                            │
                                            ▼
                         Fetch offer details from backend
                                            │
                                            ▼
                              If offer exists → show modal
                        (isGuestReviewVisible = true)



🔵 FLOW OF LOCATION UPDATES
                 ┌──────────────────────────────┐
                 │ handleLocationChange(location)│
                 └───────────────┬──────────────┘
                                 │
                     getCurrentLocation()
                                 │
                                 ▼
               Call getHomeFeeds() with new coordinates
                                 │
                                 ▼
                 actions.accessToken({lat,lng})
               (updates user location on backend)



🔵 FLOW OF CONTACT PERMISSIONS
                  ┌──────────────────────────┐
                  │    shouldSyncContacts()  │
                  └──────────────┬───────────┘
                                 │
                         If iOS → skip check, sync
                                 │
                                 ▼
                   Android: Check READ_CONTACTS permission
                                 │
                 ┌───────────────┴────────────────┐
                 ▼                                ▼
        If granted → syncContacts()        If not granted → show modal


🔵 FLOW WHEN APP IS FOCUSED AGAIN

When navigation.focus triggers:
    │
    ├── getUserData(true)    (skip location)
    ├── getEventReservation
    └── checkNotificationsPermission



🔵 TOP LEVEL STRUCTURE
return (
  <SafeAreaView>
    <View>
      <Header />   ← custom header area
    </View>

    <HomeScreenFlantList />  ← main feed

    <RatingDialog />         ← modal for guest review
    <ContactsPermissionDeclarationModal /> ← modal for contacts permission
    <Video />                ← tutorial video popup
  </SafeAreaView>
)


🔵 2. HOME FEED LIST
✔ This is the main feed showing:

Posts
Offers
Event promotions
User-generated content


items

🔵 3. RATING DIALOG (Guest review)
<RatingDialog
  ratingData={this.onGuestReviewAdded}
  seller={this.props.selectedSlide}
  showData={guestOfferReviewData}
  memberShipRating
  visibleOrNot={isGuestReviewVisible}
  onCancelPress={this.onCancelReview}
/>

✔ When does it appear?

When user gets a guest review deep link

e.g., they attended an event and need to leave a review