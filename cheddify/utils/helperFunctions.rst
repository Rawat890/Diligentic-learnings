1. startLocationListener

2. getCurrentLocation

3. showSnackBar

4. OpenURL

5. onShare

6. formatNumberWithCommas

7. getTimeInterval - FriendProfile, HomeFlantList, NotificationScreen, PurchaseAndSalesClass, FavouriteViewFlatListing, ChatListing, SmallComponent

8. showAllowLocationPermissionsAlert - AddPost, HomeFlantList,LocalFeedsFilter, Search, SearchGroups, 

9. getTimeIntervalSmallValue - NotificationScreen, ChatListing

10. generateDatesForNextWeeks - Category Filters

11. minutesIntohhmm - Invoice
12. GetLocation - homefeeds.js, FavouriteViewFlatListing, Home, EnableLocation

13. deleteObjectFromArray - ChatListing, FavouriteViewFlatListing, HomeFlantList, BlockUserListing, Search

14. getPostForValue- OverlayComponent , FriendOfferFlatList, OffetFlantList, RequestFlatList, SearchFlatList, SmallComponent, PurchaseAndSalesClass

15. generateDynamicLink - GroupQRModal, PreviewOffer, GroupDetailPreview, GuestList, FavouriteViewFlatListing, FriendProfile, HomeFlantList, SingleofferAndRequest, ShareLinkScreen, ProfileORModal, Profile, Search, OrderSummary, InviteAFriend, NewInvite

16. isRemoteFile - Create, PreviewOffer

17. filterCommentsBasedOnPrivacy - FriendProfile, Comment, HomeFlantList, FavouriteViewFlatListing

18. confirmationPopup - SelectPaymentCard, GuestList, PurchaseAndSalesClass, listCards

✅ High-Level Purpose

You are building a system that:

Checks if a Dub short link already exists for a given value + flag combination.

Returns the existing short link if found (optionally matching metadata).

Otherwise creates a new short link on Dub.com.

Uses that link when the user presses a UI button.

This avoids creating duplicate short links and saves API calls.

🔍 PART 1 — getDynamicLink(value, flag, metaData)
What it does

Takes value (e.g., a user ID) and flag (e.g., "profile").

Builds a long URL:

const longUrl = `${DUB_DOMAIN_URL}/${flag}/${value}`;


Calls Dub’s search API:

GET  /links?search=<encoded longUrl>

If a matching link exists

Two possibilities:

A. No metadata filtering

If metaData is missing → return the first matching short link:

return data[0].shortLink;

B. Metadata filtering

If metaData exists:

const shortURL = data.find(linkData =>
  metaData.title === linkData.title &&
  metaData.images.includes(linkData.image)
);


This ensures that you only reuse a link if:

Title matches

Image is in the list
(ideal for content that may have multiple variations)

🔍 PART 2 — generateDynamicLink(value, flag, creatorId, metaData)
Main flow
✔️ 1. Validate inputs

If missing → return nothing.

✔️ 2. If metadata itself is already a URL → return it
if (isString(metaData) && urlRegex.test(metaData)) {
  return metaData;
}


This avoids unnecessary Dub API calls.

✔️ 3. Check if a short link already exists
const existingShortLink = await getDynamicLink(value, flag, metaData);
if (existingShortLink) {
  return existingShortLink;
}

✔️ 4. Create a new link through Dub API

Body example:

{
  url: `https://cheddify.link/profile/12345`,
  domain: "cheddify.link",
  ios: APP_STORE_LINK,
  android: PLAY_STORE_LINK,
  proxy: true,
  image: metaData?.images?.[0],
  title: metaData?.title
}


POST to Dub’s create endpoint:

const response = await fetch(DUB_SHORT_URL_GENERATE, options);


Return new short link:

return data.shortLink;

🔍 PART 3 — UI Pressable usage
<Pressable
  onPress={async () => {
    this.setState({ shareSelected: item });

    if (!this.state.selectedOffer) {
      this.setState({
        link: await generateDynamicLink(
          this.props.route.params.user_id.other_user_id,
          dynamicLinkType.profile,
        ),
      });
    }
  }}
/>

✔️ What happens when pressed

shareSelected changes immediately.

If selectedOffer is not set, it generates a profile link for the user.

It sets:

this.state.link = <short link>


This will cause a re-render where the link is available for sharing.

⚠️ Important Notes / Improvements
1. Multiple setState() calls in one function

These run asynchronously and can merge unpredictably.

Better:

onPress={async () => {
  const link = !this.state.selectedOffer
    ? await generateDynamicLink(
        this.props.route.params.user_id.other_user_id,
        dynamicLinkType.profile
      )
    : null;

  this.setState({
    shareSelected: item,
    ...(link && { link })
  });
}}

2. metaData.images.includes(linkData.image) may break

Dub’s API may return absolute URLs

Your metadata may contain relative or different-size URLs

Consider using .some() with normalization.

3. avoid double-splitting hostnames

You use both:

DUB_DOMAIN_URL.replace(/\/+$/, '')

cheddify.link literal inside the body

Make sure all URLs remain consistent.

4. Search API may return multiple records

Your current filtering only checks:

data[0]


But the link you want may not be first.

Your find() handles metadata, but for no metadata it is safer to match by URL:

data.find(l => l.url === longUrl)
