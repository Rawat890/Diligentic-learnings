Component Initialization -
[Component Mount / Focus]
      |
      v
[useFocusEffect Hook]
      |
      v
[Dispatch getAllChats()]
      |
      v
[API Call to fetch user chats]
      |
      v
[Redux store updates userChatsReducer → chats & loading]


State Initialization - 
[Initialize local state variables]
    - searchQuery = ""
    - refreshing = false
    - searchedDoctorsAndReaders = []


Render UI (Wrapper -> Card)
[Wrapper Component]
    |
    v
[Card Component]
    |
    v
[FlatList → Data = chats]
    |
    |-- If chats exist:
    |     └── renderChat() for each chat
    |             ├── Extract otherUserDetails
    |             ├── Show Avatar
    |             ├── Show Name
    |             ├── Show Last Message (text or attachment)
    |             ├── Show Unread Count (if any)
    |             └── Show Timestamp (e.g., "2 hours ago")
    |
    |-- If chats is empty:
          └── Show <EmptyListPlaceholder />


Pull to refresh logic - 
[User pulls FlatList down]
      |
      v
[setRefreshing(true)]
      |
      v
[Dispatch getAllChats()]
      |
      v
[Wait for API call → Redux updates]
      |
      v
[setRefreshing(false)]



Search Functionality - 
[User types into SearchBar]
      |
      v
[onSearchChange(searchTerm)]
      |
      |-- If searchTerm is empty:
      |     └── Clear searchedDoctorsAndReaders
      |
      |-- If searchTerm is not empty:
            └── Make API call to getAllDoctorsAndReaders
                    |
                    v
            ┌── If success:
            |     └── set searchedDoctorsAndReaders = API response
            └── If error:
                  └── Show error toast



Render Search Suggesstions ->
[If searchQuery is not empty AND searchedDoctorsAndReaders.length > 0]
      |
      v
[Render FlatList]
    └── Each suggestion shows:
          ├── Avatar
          ├── Full name
          └── OnPress:
              ├── Navigate to Chat screen
              ├── Pass otherUserDetails as params
              └── Clear searchQuery and searchedDoctorsAndReaders


On chat Item pressed ->
[User taps a chat item in main list]
      |
      v
[navigate(SCREEN_NAMES.Chat, {
    chatId: item.id,
    otherUserDetails: ...
})]
