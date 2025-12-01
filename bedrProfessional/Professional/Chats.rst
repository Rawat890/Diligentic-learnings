
Start
  |
  v
Initialize States & Props
  |
  v
User types in SearchBar?
  |--> Yes --> Filter users --> Update `chats`
  |
  v
Render UI:
  |
  |--> chats.length || searchQuery?
        |--> Yes --> Render SearchBar + FlatList
        |--> No --> Render EmptyListPlaceholder
  |
  v
User taps chat item?
  |
  |--> Is user optometrist?
        |--> Yes --> Do Nothing
        |--> No --> Navigate to Chat screen



Chat screen - >

START
  |
  v
[Component Mount / Focus]
  |
  ├── Is `chatId` present in route?
  |       |
  |       ├─ YES: Fetch Chat Messages (getChatMessages)
  |       └─ NO: Initialize First Chat (initializeFirstChat)
  |
  v
[Socket Events Subscription]
  ├── /chat/on-new-message → Add new message to chat
  ├── /chat/mark-messages-as-read → Mark messages as read
  └── /chat/mark-messages-as-delivered → Mark messages as delivered
  |
  v
[Render GiftedChat UI]
  |
  ├── User Sends Message
  |       ├─ Encrypt Message
  |       ├─ Emit via socket → /chat/new-message
  |       └─ Append to messages on ACK
  |
  ├── User Uploads File/Image/Video
  |       ├─ Upload to S3
  |       ├─ Construct message content
  |       ├─ Encrypt & emit via socket
  |       └─ Append message on ACK
  |
  ├── Message Status Updates
  |       ├─ Delivered → update messageStatus
  |       └─ Read → update messageStatus
  |
  ├── Message Deletion
  |       ├─ Send deletion request
  |       └─ Update local message list
  |
  └── Handle Picker / Camera / Gallery Errors
          └─ Show alert / prompt user to open settings

  |
  v
[UI Components]
  ├── Avatar + Name + Online Status
  ├── Message Bubbles (text/image/video/file)
  ├── Input toolbar (attachments, send)
  └── Alerts (permission errors, delete confirm, upload progress)

  |
  v
 END
