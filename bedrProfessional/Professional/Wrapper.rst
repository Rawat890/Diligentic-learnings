Start
 │
 │
 ├─> Is user role DOCTOR or READER AND clinics.length <= 0?
 │     └─ Yes → Render nothing (return null)
 │     └─ No  → Continue
 │
 ├─> If showHeader is true → Render Header
 │
 ├─> Call renderChildren()
 │     │
 │     ├─> If canBeBlocked is false → Render children
 │     │
 │     ├─> If user is OPTOMETRIST AND has no practiceId?
 │     │     └─ Yes → Show guest user message
 │     │
 │     ├─> If user is DOCTOR or READER AND hasn't accepted terms?
 │     │     └─ Yes → Show clinic card with T&C checkbox + link to clinic rules PDF
 │     │
 │     ├─> If not OPTOMETRIST AND (canBeBlocked OR no clinics) AND hasn't accepted terms?
 │     │     └─ Yes → Render nothing
 │     │
 │     └─> Else → Render children
 │
 ├─> Render SafeAreaView with background, styles, and renderChildren()
 │
 ├─> Always include hidden BottomSheet for urgent booking
 │
 ├─> If loading is true → Show Loader
 │
 └─> Done
