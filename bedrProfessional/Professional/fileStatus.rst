[Component Mount / Screen Focused]
            |
            v
[Check practiceId availability]
            |
            v
[Dispatch fetchFiles(practiceId)]
            |
            v
[Store files in Redux --> state.filesReducer.files]
            |
            v
[Render FlatList of files]
            |
            v
[Apply search filter (by first/last name)]
            |
            v
+--------------------------------------------------+
|                For Each File:                    |
|                                                  |
| [Show ID, Name, File Status]                     |
| [Show Eye-specific actions if status == approved]|
| [Show Submission Date]                           |
| [Show 'View Report' or 'View Response' button]   |
+--------------------------------------------------+
            |
            v
[User taps 'View Report' ➜ Navigates to DiagnosisReport]
