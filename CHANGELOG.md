# Changelog

All notable changes to the Secret Animals Society pet adoption app will be documented in this file.

---

## [2025-12-05 11:30] - Fixed Adoption Questionnaire Input Bug

### Problem Identified
The adoption questionnaire had a critical bug where text input fields only accepted one character before losing focus. Users couldn't type their name, email, or phone number properly. The "Report Cruelty" form worked fine, which helped identify the issue was specific to the questionnaire.

### Root Cause
This was a **classic React component re-mounting issue** caused by defining components inside another component's body:

```javascript
// BEFORE (BROKEN):
const QuestionnaireScreen = () => {
    const Step1_Contact = () => (...);  // ❌ Defined INSIDE parent
    // When state updates, Step1_Contact is recreated as a new function
    // React sees it as a different component and unmounts/remounts it
}
```

**What was happening:**
1. User types a character in the input
2. State updates (via `onFormUpdate`)
3. `QuestionnaireScreen` re-renders
4. All inner component definitions (`Step1_Contact`, `Step2_HomeLife`, etc.) are recreated as NEW functions
5. React compares old vs new and sees different function references
6. React thinks these are different components, so it unmounts the old one and mounts the new one
7. Input loses focus during unmount/remount
8. User can only type one character at a time

### Solution Applied
**Moved all component definitions outside the parent component** to module-level scope:

```javascript
// AFTER (FIXED):
const Step1_Contact = ({ formData, onFormUpdate }) => (...);  // ✅ Defined at module level
const Step2_HomeLife = ({ localAnswers, handleLocalUpdate }) => (...);
const Step3_Lifestyle = ({ localAnswers, handleLocalUpdate, formData, onFormUpdate }) => (...);
const Step4_Preferences = ({ formData, onFormUpdate }) => (...);
const FormNavigation = ({ currentStep, totalSteps, onBack, onNext, onFinish, isStepValid }) => (...);
const StepIndicator = ({ currentStep, totalSteps }) => (...);

const QuestionnaireScreen = () => {
    // Components are now defined outside, so they maintain stable references
    // React doesn't unmount/remount them on every state change
}
```

### Components Extracted
- `FormNavigation` - Back/Next/Submit buttons
- `StepIndicator` - Progress circles showing current step
- `Step1_Contact` - Contact information form (name, email, phone)
- `Step2_HomeLife` - Home environment questions (own/rent, dwelling type, yard)
- `Step3_Lifestyle` - Lifestyle and pet type preferences
- `Step4_Preferences` - Personality trait selection

### Files Changed
- `index.html` - Lines 314-550
  - Extracted 6 component definitions from inside `QuestionnaireScreen` to module level
  - Added beginner-friendly comments explaining WHY components must be defined outside
  - Updated all component calls to pass required props
  - Updated `StepIndicator` call to pass `currentStep` and `totalSteps` props

### Educational Notes
This is an important learning moment for React developers:

**Rule:** Never define components inside other components unless you have a very specific reason and understand the implications.

**Why?** In React, component identity is determined by function reference. When you define a component inside another component:
- Every render creates a new function reference
- React sees a "different" component and resets its state
- This causes unmounting/remounting, breaking features like input focus, scroll position, animations, etc.

**Best Practice:** Always define components at the module level (outside of other components) and pass data via props.

### Testing Performed
- ✅ Verified all 4 steps of questionnaire accept multi-character input
- ✅ Name field allows full names to be typed
- ✅ Email field accepts full email addresses
- ✅ Phone field accepts complete phone numbers
- ✅ All button selections still work correctly
- ✅ Form validation still functions properly
- ✅ Navigation between steps works as expected

### Impact
- **User Experience:** Users can now complete the adoption questionnaire without frustration
- **Code Quality:** Better React patterns demonstrate proper component architecture for beginners
- **Maintainability:** Components are now easier to test and reuse independently

---

*This changelog documents changes to help future developers (and AI assistants) understand the evolution of this codebase.*
