[Start]
   |
   v
[Render ClinicCard Component]
   |
   v
[Initialize States]
   - isDiagnosisCheckboxSelected
   - isApprovalCheckboxSelected
   - hiddenCategoriesVisible
   |
   v
[useEffect: Sync checkboxes]
   |
   v
[Compute]
   - mergedFiles from clinics
   - hiddenFiles from clinics
   - allCategories (unique by category_id)
   - hiddenCategories (unique list of hidden ids)
   - hiddenCategoriesArrays (visible category details)
   |
   v
[Render UI Components]
   |
   v
[Card Header]
   |
   v
[Check File Status]
   |             \
   |              \
[SUBMITTED]     [PENDING_APPROVAL]
   |                 |
   v                 v
[Render Sort    [Render Sort Checkbox (Approval)]
Checkbox 
(Diagnosis)]
   |
   v
[Render Visible Categories]
   |
   v
[For Each Category]
   |
   v
[Render Category Button]
   |
   v
[Button Click → onButtonPress(category_id)]
   |
   v
[Render File Count and Hide Button]
   |
   v
[Hide Button Click → onHidePress({categoryId, isHidden: true})]
   |
   v
[Render "View Hidden Categories" Button if any hidden exist]
   |
   v
["View Hidden Categories" → setHiddenCategoriesvisible(true)]
   |
   v
[BottomSheetComponent Visible]
   |
   v
[For Each Hidden Category]
   |
   v
[Render Name + "Show" Button]
   |
   v
["Show" → onHidePress({categoryId, isHidden: false}) + close sheet]
   |
   v
[BottomSheet Close Button]
   |
   v
[End]
