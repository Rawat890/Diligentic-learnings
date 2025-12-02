HomeFlatntScreen.js ----

Below is a clean, structured explanation of the code you shared.
I break everything into sections, functions, and what each is doing in simple terms — basically a full annotated breakdown.

✅ CLASS OVERVIEW
class HomeScreenFlantList extends PureComponent

This is a React Native class component that:

Displays the Home Feed (list of posts)

Handles feed interactions (like, comment, share, repost, etc.)

Handles online/offline sockets

Manages favourites, followers, muted feeds, etc.

Manages video autoplay and stopping when screen loses focus

Retrieves users, friends, feed data, shares profile links, opens bottom sheets, etc.

It is a very large, multipurpose feed controller.

🧩 CONSTRUCTOR & STATE
constructor(props)

Initializes the component and sets a large state to manage UI, feed actions, dialogs, and user actions.

State Variables Explained

Here is what each state value represents:

State	Purpose
isLoading	Show loading indicator for API calls
length	Generic tracker (unknown context)
addTestimonial	Stores testimonial data
zoomImage	Boolean: show full-screen image viewer
zoomableImage	Image URL for zoom viewer
location	User location
onlineUser	List of online users
folder_name	Name of bookmark folder user creates
favouriteObject	Feed item selected for favorite actions
selectedItemAlreadySavedIn	Folder(s) where feed is already saved
favouriteIndex	Track index of selected feed
testimonial	Flags testimonial UI
feedActionDialog	Show feed action menu (report, block, etc.)
isMyFeed	True if feed belongs to logged-in user
repostDialog	Repost modal
repostSelectedValue	Selected report option
description	User’s description input
uploadedScreenShots	Screenshots uploaded for report
showSavedToSnackbar	Snackbar UI toggle
profileQRVisible	Show QR code modal
totalFeedNumber	Count of feeds
friendsNumber	Count of friends
followingNumber	Count of following
mentionsNumber	Count of mentions
shareModal	Show share modal
myFollowers	User’s followers list
shareSelected	Selected friend for sharing
myId	Current user ID
myName	Current user name
link	Dynamic share link
item	Current feed item
searchFriends	Search query
searchFriendsArray	Filtered friends
muteModal	Mute modal visibility
mutedFeeds	Muted feeds list
selectedFeedToShare	Feed user is sharing
sharingMyProfile	Whether user shares a profile
selectedCategory	“Friends”, “Following”, etc.
userOnline	Online user list
currentlyPlayingPostID	Which video is currently playing
screenFocused	If screen is active
stopPlaying	Stop video autoplay
Refs

bottomSheet → Bottom sheet modal reference

searchSheet → Search modal reference

🔵 LIFECYCLE METHODS
componentDidMount()

Runs when screen loads:

Adds focus listener → refresh data + start playing videos

Adds blur listener → stop playing videos

Listens for sockets:

"onlineUser" → update who is online

"newPostAdded" → refresh feeds when someone posts

"offlineUserWithLastOnline" → update last seen status

"receiveGroupChannelMessage" → update groups list

Fetches:

muted feeds

online users

home feed

group list

componentWillUnmount()

Removes socket listeners and navigation listeners when leaving screen.

🟢 CORE LOGIC FUNCTIONS
fetchInitialHomeFeeds()

If audience = local → check location permission

Calls API → getHomeFeeds

Loads or refreshes feed items.

onAddNewPost()

Triggered on socket event "newPostAdded"
→ Refreshes number counters.

handleUserCameOnline(userId)

When socket detects user online:

Refresh online user list

Calls userCameOnline(userId) utility

Navigation Helpers
moveToNewScreen(id, data)

Navigate to another screen with parameters.

moveToNewScreenForSingleOffer(id, data)

Navigation for offer-related screens.

scrollToTop()

Scroll the FlatList to index 0.

❤️ LIKE, COMMENT, FAVOURITE FUNCTIONS
likeAndComment(likeObject, index)

Calls API to update feed like count.

likeFeed(data, index)

Handles UI side of liking a post before sending to server.

bookMarkFeed(saveFeed)

Adds/Removes a post from bookmarks

Calls API

Updates selected folders

Updates feed UI

favouriteSenario(favouriteObject, index)

Opens bottom sheet for selecting bookmark folder.

📌 FRIENDS, SHARING & INVITES
myFriends()

Fetches logged-in user friend list.

shareProfileLink()

Gets user invite token

Generates Firebase dynamic link

Opens share sheet

Tracks analytics

Updates remaining invites

renderFriend()

UI component for showing a friend bubble (avatar in a circle)

📁 FOLDERS & FEED MANAGEMENT
createFolder()

Creates a new bookmark folder and updates UI.

manageFeed(manageFeedObject, flag)

Handles:

Report Feed → flag = 1

Block User → flag = 2

Delete Feed → flag = 3

Updates feed list accordingly.

feedActionDialogOpen(data, index)

Opens/Closes the “•••” menu for reporting/blocking.

checkIsMyFeed(favouriteObject)

Determines if feed belongs to logged-in user.

feedDialogActionPerfrom(flag)

Performs report/block/delete based on flag.

👤 PROFILE & AUTHOR ACTIONS
profileClick(_id)

If user clicks on profile picture
→ load own profile OR other’s profile

📝 RENDERING HELPERS
likedBy(item)

Formats a string like:

“John and 2 others”

getAvatar(friend, size)

Returns:

Friend’s profile picture
or

First letter of their name

🖼️ UI RENDERING OF POSTS
_renderItem({ item, index })

This is the main feed renderer.

It renders:

1. Feed Header

(profile picture, name, timestamps)

2. Titles & descriptions

Handles links, hashtags, mentions.

3. Images / Videos

Uses:

VideoPlayerForLists

CurouselComponentWithThumbnale

FeedBoxComponent

FeedBoxComponentImage

4. Reposts

Separate UI if post is a repost.

5. Shared user profiles

When someone shared a profile.

6. Offers & Requests

Special UI layout for offers.

7. Ratings / Reviews

Special layout for reviews.

8. Groups

Special layout for group posts.

9. Footer

Contains:

Like button

Comment button

Share button

Repost button

10. Likes and Comments list
11. Timestamp
📌 Summary: What the Whole Component Does

This class:

Manages the entire Home Feed UI

Handles all data fetching & socket events

Manages likes, comments, shares, reposts

Handles image zoom, video autoplay control

Allows bookmarking, creating folders

Manages feed reporting, blocking, deleting

Handles navigation to profiles, offers, comments

Manages online users & followers

Renders MANY different feed types:

text posts

photo posts

video posts

group posts

shared profiles

offers/requests

reposts