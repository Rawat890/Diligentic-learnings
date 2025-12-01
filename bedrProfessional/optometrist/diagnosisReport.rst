[Component Mounted]
       |
       v
[Check if file.id exists]
       |
       v
[Call getPatientDetails()]
       |
       v
[Set diagnosisFormHtml]
       |
       v
[Check Post-Op Completion / Email Status]
       |
       +-------------------------+
       |                         |
       v                         v
[Post-Op Completed or Email Sent?] -- Yes --> [Hide Post-Op Buttons]
       |
      No
       |
       v
[Show Post-Op Buttons]
       |
       v
[Render WebView with diagnosisFormHtml]
       |
       +-----------------------------+
       |                             |
       v                             v
[Referral Needed?] -- Yes --> [Show "Confirm Referral" Button]
       |
      No
       |
       v
[Private Category + Post-Op Action?] -- Yes --> [Show Yes/No Post-Op Exam Buttons]
       |
       v
[User clicks any confirm button]
       |
       v
[Show Confirmation Modal]
       |
       +-------------------------------+
       |                               |
       v                               v
[isPrivateCategory?]           [Not Private Category]
       |                               |
       v                               v
[Send Post-Op Email (Yes/No)]    [Call referPatient API]
       |                               |
       v                               v
[Set canButtonsBeDisplayed = true]
       |
       v
[Show "File Updated" + Close Button]
       |
       v
[User clicks "Close File"]
       |
       v
[goBack()]
