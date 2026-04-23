# NameInputGroup Validation Complete Guide

**Document Version:** 1.0  
**Last Updated:** April 23, 2026  
**Component:** NameInputGroup with React Hook Forms + Zod Validation

---

## Table of Contents

1. [Overview](#overview)
2. [Architecture](#architecture)
3. [Step-by-Step Validation Flow](#step-by-step-validation-flow)
4. [Key Concepts](#key-concepts)
5. [Code Breakdown](#code-breakdown)
6. [Real-World Examples](#real-world-examples)
7. [Future Schema Integration](#future-schema-integration)
8. [Troubleshooting](#troubleshooting)

---

## Overview

The `NameInputGroup` component is a **multi-field form input** that manages three related fields (firstName, middleName, lastName) with integrated **React Hook Forms** and **Zod validation**.

### Key Features

- ✅ **Three-field input group**: firstName, middleName, lastName
- ✅ **Real-time validation**: Validates as user types or on blur
- ✅ **Cross-field validation**: Aggregates errors from all 3 fields
- ✅ **Responsive design**: Stacks on mobile, horizontal on desktop
- ✅ **Schema-agnostic**: Works with any Zod schema
- ✅ **Accessibility**: ARIA labels, descriptions, error associations

### Two Component Variations

| Component | Purpose | Usage |
|-----------|---------|-------|
| `NameInputGroup` | Standalone/manual validation | Uncontrolled, no form |
| `FormNameInputGroup` | React Hook Forms wrapper | Inside a form with validation |

---

## Architecture

### Component Hierarchy

```
FormProvider (provides form context)
    ↓
<form> element
    ↓
FormNameInputGroup (uses useFormContext)
    ├─ watches form state
    ├─ runs validation
    ├─ aggregates errors
    ↓
NameInputGroup (base component)
    ├─ renders 3 inputs
    ├─ displays errors
    └─ manages UI state
```

### Data Flow

```
User Input (typing)
    ↓
onChange handler fires
    ↓
setValue() + trigger() in FormNameInputGroup
    ↓
Zod schema validation runs
    ↓
Validation results stored in formState.errors
    ↓
getFieldState() retrieves error for each field
    ↓
FormNameInputGroup aggregates errors
    ↓
isInvalid + errorMessage props passed to NameInputGroup
    ↓
NameInputGroup renders with error styling/text
```

---

## Step-by-Step Validation Flow

### Phase 1: Setup (Application Initialize)

#### 1.1 Define Validation Schema

**File:** `app/components/Aria/NameInputGroup.stories.tsx`

```typescript
const nameSchema = z.object({
  firstName: z
    .string()
    .min(1, "First name is required")              // Error #1: Empty check
    .min(2, "First name must be at least 2 chars"), // Error #2: Length check
  
  middleName: z
    .string()
    .optional()                                     // Can be empty
    .or(z.literal("")),                            // Or explicitly empty string
  
  lastName: z
    .string()
    .min(1, "Last name is required")
    .min(2, "Last name must be at least 2 chars"),
})
```

**What happens here:**
- Zod creates a **validation schema object**
- Each field has **validation rules**
- Each rule has an **error message**
- Rules are **chained** (each must pass)

**Example validation chain for firstName:**
```
Input: "J"
  ↓ Rule 1: .min(1) → PASS ✅ (length 1 >= 1)
  ↓ Rule 2: .min(2) → FAIL ❌ (length 1 < 2)
  → Error: "First name must be at least 2 chars"
```

---

#### 1.2 Create TypeScript Type from Schema

```typescript
type NameFormData = z.infer<typeof nameSchema>

// This generates:
// {
//   firstName: string
//   middleName: string | undefined
//   lastName: string
// }
```

**What happens here:**
- Zod **infers the TypeScript type** from schema
- Ensures type safety across the form
- IDE autocomplete knows the fields

---

#### 1.3 Initialize Form with useForm Hook

```typescript
const formMethods = useForm<NameFormData>({
  resolver: zodResolver(nameSchema),    // Step 1: Connect schema
  mode: "onBlur",                       // Step 2: When to validate
  defaultValues: {                      // Step 3: Initial values
    firstName: "",
    middleName: "",
    lastName: "",
  }
})
```

**What happens here:**

| Property | Purpose | Example |
|----------|---------|---------|
| `resolver` | Connects Zod schema to React Hook Forms | `zodResolver(nameSchema)` converts Zod → RHF format |
| `mode` | Controls when validation runs | `"onBlur"` = validate after user leaves field |
| `defaultValues` | Initial field values | All empty strings at start |

**Available validation modes:**
- `onSubmit`: Only validate when form submitted
- `onChange`: Validate on every keystroke (real-time)
- `onBlur`: Validate when field loses focus
- `onTouched`: Validate after field is touched and dirty

**formMethods contains:**
```typescript
{
  watch,              // Subscribe to field changes
  setValue,           // Update field value programmatically
  trigger,            // Run validation manually
  getFieldState,      // Get validation state for one field
  handleSubmit,       // Handle form submission
  formState: {
    errors,           // { firstName: { message: '...' } }
    isValid,          // true if all fields pass validation
    isDirty,          // true if form has been modified
    isSubmitting,     // true while submitting
  }
}
```

---

#### 1.4 Wrap Form with FormProvider

```typescript
<FormProvider {...formMethods}>
  {/* All children can now access formMethods via useFormContext() */}
  <form onSubmit={handleSubmit(onSubmit)}>
    <FormNameInputGroup ... />
  </form>
</FormProvider>
```

**What happens here:**
- React Context broadcasts `formMethods` to all children
- Child components can call `useFormContext()` to access form state
- No need to pass `control` prop or `formMethods` down
- **This is the key to the validation system!**

---

### Phase 2: User Interaction

#### 2.1 User Clicks Input & Types Character

**Action:** User clicks the firstName input field and types "J"

```
UI: <Input id="firstName" onChange={handleChange} />

What the browser does:
1. Input gains focus
2. User presses "J" key
3. Browser fires onChange event
4. Event object contains new value "J"
```

---

#### 2.2 onChange Handler Executes

**Handler Location:** Inside `FormNameInputGroup` component

```typescript
const handleChangeFirstName = (value: string) => {
  // value = "J" (what user typed)
  
  // Step 1: Update the form state in React Hook Forms
  setValue(fieldNames.firstName, value as never)
  
  // Step 2: Immediately re-run validation
  trigger(allFieldPaths)
}
```

**What happens here:**

1. **setValue()** - Updates form internal state
   ```
   React Hook Forms internal state BEFORE:
   { firstName: "", middleName: "", lastName: "" }
   
   React Hook Forms internal state AFTER:
   { firstName: "J", middleName: "", lastName: "" }
   ```

2. **trigger()** - Runs validation on specified fields
   ```
   trigger([fieldNames.firstName, fieldNames.middleName, fieldNames.lastName])
   
   This tells Zod: "Validate these 3 fields NOW"
   ```

---

#### 2.3 Zod Schema Runs Validation

**What happens inside Zod:**

```typescript
// Zod receives the value "J"

const result = nameSchema.parse({
  firstName: "J",           // ← This value
  middleName: "",
  lastName: ""
})

// Zod checks each rule in order:
```

**Rule 1: `.min(1)` - Is minimum 1 character?**
```
"J".length = 1
1 >= 1 ? YES ✅ PASS
Continue to next rule...
```

**Rule 2: `.min(2)` - Is minimum 2 characters?**
```
"J".length = 1
1 >= 2 ? NO ❌ FAIL

Stop here and generate error:
{
  type: "too_small",
  message: "First name must be at least 2 characters",
  path: ["firstName"]
}
```

**Final Zod Result:**
```typescript
{
  success: false,
  error: {
    issues: [
      {
        code: "too_small",
        minimum: 2,
        type: "string",
        message: "First name must be at least 2 characters",
        path: ["firstName"]
      }
    ]
  }
}
```

---

#### 2.4 React Hook Forms Stores Error

**React Hook Forms receives Zod result:**

```typescript
// Zod said firstName is invalid

formState.errors = {
  firstName: {
    type: "too_small",
    message: "First name must be at least 2 characters"
  }
}

// formState.isValid = false (because firstName has error)
```

---

### Phase 3: Component Updates

#### 3.1 FormNameInputGroup Gets Updated State

**Inside FormNameInputGroup:**

```typescript
// useFormContext gives us access to formState
const { getFieldState, formState } = useFormContext<T>()

// Get the validation state for firstName
const firstNameState = getFieldState(fieldNames.firstName, formState)

// firstNameState now contains:
{
  invalid: true,                    // ← Is this field invalid?
  error: {
    type: "too_small",
    message: "First name must be at least 2 characters"
  },
  isDirty: true,                    // ← User modified this field
  isTouched: false,                 // ← User hasn't left field yet
}
```

---

#### 3.2 FormNameInputGroup Aggregates Errors

**Purpose:** Create a single error message from 3 fields

```typescript
const isInvalid = useMemo(() => {
  // Check if ANY field is invalid
  const allStates = [firstNameState, middleNameState, lastNameState]
  return allStates.some((state) => state.invalid)
}, [firstNameState, middleNameState, lastNameState])

// Result: true (because firstNameState.invalid = true)
```

```typescript
const errorMessage = useMemo(() => {
  // Find the FIRST error message
  const fields = [
    { label: "First Name", state: firstNameState },
    { label: "Middle Name", state: middleNameState },
    { label: "Last Name", state: lastNameState },
  ]
  
  for (const { label, state } of fields) {
    if (state.error?.message) {
      return `${label}: ${state.error.message}`
    }
  }
  
  return undefined
}, [firstNameState, middleNameState, lastNameState])

// Result: "First Name: First name must be at least 2 characters"
```

---

#### 3.3 FormNameInputGroup Passes Props to NameInputGroup

```typescript
return (
  <NameInputGroup
    firstName={firstNameValue ?? ""}    // "J"
    middleName={middleNameValue ?? ""} // ""
    lastName={lastNameValue ?? ""}     // ""
    onChangeFirstName={handleChangeFirstName}
    onChangeMiddleName={handleChangeMiddleName}
    onChangeLastName={handleChangeLastName}
    isInvalid={isInvalid}              // ← true (VALIDATION ERROR)
    errorMessage={errorMessage}        // ← "First Name: ..." (ERROR MESSAGE)
  />
)
```

---

### Phase 4: UI Rendering

#### 4.1 NameInputGroup Receives Props

```typescript
export function NameInputGroup({
  isInvalid,         // true
  errorMessage,      // "First Name: First name must be..."
  ...rest
}: NameInputGroupProps) {
  // Now render based on these props
}
```

---

#### 4.2 FieldGroup Gets Error Styling

```typescript
<FieldGroup
  isInvalid={isInvalid}  // true
  className="h-auto flex-col..."
>
  {/* Renders inputs here */}
</FieldGroup>
```

**What FieldGroup does with `isInvalid={true}`:**
- Adds red border styling
- Changes focus color to red
- Sets `aria-invalid="true"` for accessibility

---

#### 4.3 Error Message Displays

```typescript
{isInvalid && errorMessage && (
  <div className="pt-1 text-sm text-red-500">
    {errorMessage}  {/* Shows: "First Name: First name must be at least 2 characters" */}
  </div>
)}
```

---

### Phase 5: User Types More

#### 5.1 User Types 'o' → firstName = "Jo"

```
Same cycle repeats:
1. onChange fires
2. setValue("firstName", "Jo")
3. trigger() runs validation
4. Zod checks: "Jo".length = 2, 2 >= 2? ✅ PASS
5. No error generated
6. formState.errors.firstName = undefined
7. getFieldState returns { invalid: false }
8. isInvalid = false
9. errorMessage = undefined
10. NameInputGroup renders WITHOUT error styling
```

**Visual Change:**
```
BEFORE (isInvalid=true):
┌─────────────────┐
│ ❌ J            │  Red border
└─────────────────┘
First Name: First name must be at least 2 characters

AFTER (isInvalid=false):
┌─────────────────┐
│ Jo              │  Gray border (normal)
└─────────────────┘
                    (no error text)
```

---

#### 5.2 User Adds "hn" → firstName = "John"

```
Still valid because:
"John".length = 4
4 >= 2 ✅ PASS
```

---

#### 5.3 User Fills All Required Fields

```
firstName = "John"   ✅ 4 >= 2
lastName = "Doe"     ✅ 3 >= 2
middleName = ""      ✅ Optional field

formState.isValid = true
```

---

### Phase 6: Form Submission

#### 6.1 User Clicks Submit Button

```typescript
<form onSubmit={handleSubmit(onSubmit)}>
  <Button type="submit">Submit</Button>
</form>
```

**What handleSubmit does:**

```typescript
const onSubmit = (data: NameFormData) => {
  // data = {
  //   firstName: "John",
  //   middleName: "",
  //   lastName: "Doe"
  // }
  console.log("Form submitted:", data)
  alert(JSON.stringify(data, null, 2))
}

const handleSubmit = handleSubmit(onSubmit)
// handleSubmit runs:
// 1. Validate ALL fields against schema
// 2. If ALL valid → call onSubmit(validData)
// 3. If ANY invalid → show errors, don't submit
```

---

#### 6.2 Validation Check

```typescript
// handleSubmit does final validation
const isValid = await zodResolver(nameSchema).validate({
  firstName: "John",
  middleName: "",
  lastName: "Doe"
})

// Result: ✅ VALID - All fields pass

// Since valid: call onSubmit(data)
```

---

#### 6.3 Submit Handler Executes

```typescript
const onSubmit = (data: NameFormData) => {
  console.log("Form submitted:", data)
  // Output:
  // {
  //   firstName: "John",
  //   middleName: "",
  //   lastName: "Doe"
  // }
  
  alert(JSON.stringify(data, null, 2))
  // Shows: { "firstName": "John", "middleName": "", "lastName": "Doe" }
}
```

---

## Key Concepts

### 1. Zod Schema

**What it is:** A validation library that defines rules

```typescript
z.string()              // Must be a string
  .min(1, "msg1")      // Minimum 1 character (error: msg1)
  .min(2, "msg2")      // Minimum 2 characters (error: msg2)
```

**How it works:**
- Rules are **chainable**
- Each rule must pass in order
- First failure stops validation
- Returns error with message

---

### 2. zodResolver

**What it is:** Adapter that connects Zod to React Hook Forms

```typescript
resolver: zodResolver(nameSchema)
```

**Why needed:**
- React Hook Forms has its own resolver system
- Zod is a separate validation library
- zodResolver translates between them
- Allows RHF to use Zod validation

---

### 3. React Hook Forms

**What it is:** Library that manages form state and validation

**Key methods:**
- `watch()` - Subscribe to field changes
- `setValue()` - Update field value
- `trigger()` - Run validation
- `getFieldState()` - Get validation state
- `handleSubmit()` - Validate + submit

---

### 4. FormProvider + useFormContext

**What it is:** React Context for sharing form state

```typescript
<FormProvider {...formMethods}>
  <FormNameInputGroup />  {/* Can call useFormContext() */}
</FormProvider>
```

**Why used:**
- Avoids prop drilling
- Child components access form state easily
- Cleaner architecture
- Used across the codebase

---

### 5. Validation Modes

| Mode | When it runs | Use case |
|------|-------------|----------|
| `onSubmit` | Only on submit click | Complex forms, expensive validations |
| `onChange` | Every keystroke | Real-time feedback |
| `onBlur` | After field loses focus | Most common, balanced approach |
| `onTouched` | After field touched + dirty | Polite, only after interaction |

---

## Code Breakdown

### NameInputGroup Component

**Purpose:** Base presentational component

**Responsibilities:**
1. Render 3 input fields
2. Display group label
3. Display error message
4. Apply error styling

**Props it accepts:**
```typescript
interface NameInputGroupProps {
  label?: string                           // "Full Name"
  description?: string                     // "Please enter your complete name"
  errorMessage?: string                    // "First Name: must be 2+ chars"
  isInvalid?: boolean                      // true/false
  isRequired?: boolean                     // Show asterisk on label
  isDisabled?: boolean                     // Disable all inputs
  isReadOnly?: boolean                     // Read-only inputs
  firstName: string                        // Current value
  middleName: string
  lastName: string
  onChangeFirstName: (v: string) => void   // Change handler
  onChangeMiddleName: (v: string) => void
  onChangeLastName: (v: string) => void
  placeholder?: string                     // Input placeholder
}
```

**Code flow:**
```typescript
export function NameInputGroup({
  label,
  description,
  errorMessage,
  isInvalid,
  // ... other props
}: NameInputGroupProps) {
  
  // 1. Create unique IDs for accessibility
  const labelId = useId()
  const descriptionId = useId()
  const errorId = useId()
  
  // 2. Map field values to change handlers
  const names = {
    firstName: { value: firstName, onChange: onChangeFirstName },
    middleName: { value: middleName, onChange: onChangeMiddleName },
    lastName: { value: lastName, onChange: onChangeLastName },
  }
  
  // 3. Render
  return (
    <div>
      {/* Render label if provided */}
      {label && <SharedLabel>{label}</SharedLabel>}
      
      {/* Render FieldGroup container with error styling */}
      <FieldGroup isInvalid={isInvalid}>
        {/* Render 3 inputs */}
        {NAME_FIELDS.map((field, index) => (
          <Input value={names[field].value} onChange={names[field].onChange} />
        ))}
      </FieldGroup>
      
      {/* Render description if provided */}
      {description && <SharedDescription>{description}</SharedDescription>}
      
      {/* Render error if invalid */}
      {isInvalid && errorMessage && <div className="text-red-500">{errorMessage}</div>}
    </div>
  )
}
```

---

### FormNameInputGroup Component

**Purpose:** React Hook Forms wrapper

**Responsibilities:**
1. Access form context
2. Watch field values
3. Get validation state
4. Aggregate errors
5. Handle changes with validation
6. Pass to base component

**Code flow:**
```typescript
export function FormNameInputGroup<T extends FieldValues>({
  fieldNames,
  ...rest
}: FormNameInputGroupProps<T>) {
  
  // 1. Get form methods from React Context
  const { watch, setValue, trigger, getFieldState, formState } = useFormContext<T>()
  
  // 2. Watch field values (re-render on change)
  const firstNameValue = watch(fieldNames.firstName)
  const middleNameValue = watch(fieldNames.middleName)
  const lastNameValue = watch(fieldNames.lastName)
  
  // 3. Get validation state for each field
  const firstNameState = getFieldState(fieldNames.firstName, formState)
  const middleNameState = getFieldState(fieldNames.middleName, formState)
  const lastNameState = getFieldState(fieldNames.lastName, formState)
  
  // 4. Determine if group is invalid (ANY field invalid)
  const isInvalid = [firstNameState, middleNameState, lastNameState].some(s => s.invalid)
  
  // 5. Aggregate error messages (first error found)
  const errorMessage = useMemo(() => {
    const states = [
      { label: "First Name", state: firstNameState },
      { label: "Middle Name", state: middleNameState },
      { label: "Last Name", state: lastNameState },
    ]
    for (const { label, state } of states) {
      if (state.error?.message) {
        return `${label}: ${state.error.message}`
      }
    }
    return undefined
  }, [firstNameState, middleNameState, lastNameState])
  
  // 6. Create change handlers with validation
  const handleChangeFirstName = (value: string) => {
    setValue(fieldNames.firstName, value as never)
    trigger([fieldNames.firstName, fieldNames.middleName, fieldNames.lastName])
  }
  const handleChangeMiddleName = (value: string) => { /* ... */ }
  const handleChangeLastName = (value: string) => { /* ... */ }
  
  // 7. Render base component with validation props
  return (
    <NameInputGroup
      {...rest}
      firstName={firstNameValue ?? ""}
      middleName={middleNameValue ?? ""}
      lastName={lastNameValue ?? ""}
      onChangeFirstName={handleChangeFirstName}
      onChangeMiddleName={handleChangeMiddleName}
      onChangeLastName={handleChangeLastName}
      isInvalid={isInvalid}
      errorMessage={errorMessage}
    />
  )
}
```

---

## Real-World Examples

### Example 1: User Leaves Field Empty

**Scenario:** User clicks firstName, then immediately clicks lastName without typing

```
Initial: firstName = ""
User action: Click away
Validation trigger: onBlur mode

Zod validation:
- Value: ""
- Rule 1: .min(1) → FAIL ❌ (length 0 < 1)
- Error: "First name is required"

Result:
- firstNameState.invalid = true
- firstNameState.error.message = "First name is required"
- isInvalid = true
- errorMessage = "First Name: First name is required"

UI:
┌─────────────────┐
│ ❌              │  Red border
└─────────────────┘
First Name: First name is required  ← Red text
```

---

### Example 2: Valid Submission

**Scenario:** User fills all fields correctly and submits

```
firstName: "Alice"    ✅ 5 >= 2
middleName: ""        ✅ Optional
lastName: "Smith"     ✅ 5 >= 2

All fields valid:
- firstNameState.invalid = false
- middleNameState.invalid = false
- lastNameState.invalid = false
- isInvalid = false
- errorMessage = undefined
- formState.isValid = true

User clicks Submit:
handleSubmit validates → all pass
onSubmit(data) called with:
{
  firstName: "Alice",
  middleName: "",
  lastName: "Smith"
}
```

---

### Example 3: Multiple Field Errors

**Scenario:** FirstName = "X", LastName = "Y"

```
Validation results:
- firstName: "X" → INVALID (too short)
  Error: "First name must be at least 2 characters"
- lastName: "Y" → INVALID (too short)
  Error: "Last name must be at least 2 characters"

Aggregation:
- isInvalid = true (both fields invalid)
- errorMessage = "First Name: First name must be at least 2 characters"
  (Returns FIRST error found, not all)

UI shows:
┌─────────────────┐
│ ❌ X            │
└─────────────────┘
First Name: First name must be at least 2 characters

User fixes firstName to "Alice":
- isInvalid = true (still, because lastName invalid)
- errorMessage = "Last Name: Last name must be at least 2 characters"
  (Now shows second error)

┌─────────────────┐
│ ✓ Alice         │
└─────────────────┘
Last Name: Last name must be at least 2 characters

User fixes lastName to "Smith":
- isInvalid = false (all valid!)
- errorMessage = undefined

┌─────────────────┐
│ ✓ Alice | Smith │
└─────────────────┘
(no error)
```

---

## Future Schema Integration

### Current State (Stories Only)

```typescript
// app/components/Aria/NameInputGroup.stories.tsx
const nameSchema = z.object({
  firstName: z.string().min(1).min(2, "First name must be at least 2 characters"),
  middleName: z.string().optional().or(z.literal("")),
  lastName: z.string().min(1).min(2, "Last name must be at least 2 characters"),
})
```

---

### Future State (Shared Schema)

**Step 1: Create shared schema file**
```typescript
// app/schema/name-schema.ts
export const nameFieldsSchema = z.object({
  firstName: z
    .string()
    .min(1, "First name is required")
    .min(2, "First name must be at least 2 characters"),
  
  middleName: z
    .string()
    .optional()
    .or(z.literal("")),
  
  lastName: z
    .string()
    .min(1, "Last name is required")
    .min(2, "Last name must be at least 2 characters"),
})

export type NameFieldsData = z.infer<typeof nameFieldsSchema>
```

**Step 2: Import in stories**
```typescript
// app/components/Aria/NameInputGroup.stories.tsx
import { nameFieldsSchema } from "~/schema/name-schema"

const formMethods = useForm({
  resolver: zodResolver(nameFieldsSchema),  // ← Just import and use!
  mode: "onBlur",
})
```

**Step 3: Import in actual forms**
```typescript
// app/routes/consign.vehicle.applicant._index/Form.tsx
import { nameFieldsSchema } from "~/schema/name-schema"

export function ApplicantForm() {
  const form = useForm({
    resolver: zodResolver(nameFieldsSchema),
  })
  
  return (
    <FormProvider {...form}>
      <FormNameInputGroup
        fieldNames={{
          firstName: "applicant.firstName",
          middleName: "applicant.middleName",
          lastName: "applicant.lastName",
        }}
        label="Applicant Name"
      />
    </FormProvider>
  )
}
```

---

## Troubleshooting

### Issue 1: "Cannot destructure property 'setValue' of 'useFormContext(...)' as it is null"

**Cause:** `FormNameInputGroup` used without `FormProvider`

**Solution:**
```typescript
// ❌ WRONG
<FormNameInputGroup fieldNames={{ firstName: "firstName" }} />

// ✅ CORRECT
const formMethods = useForm({ ... })
<FormProvider {...formMethods}>
  <FormNameInputGroup fieldNames={{ firstName: "firstName" }} />
</FormProvider>
```

---

### Issue 2: Validation not triggering on change

**Cause:** `mode` set to `"onSubmit"` or `"onBlur"` instead of `"onChange"`

**Solution:**
```typescript
// ❌ WRONG (only validates on submit)
useForm({ mode: "onSubmit" })

// ✅ CORRECT (validates on every change)
useForm({ mode: "onChange" })
```

---

### Issue 3: Error message shows for both firstName and lastName

**Cause:** Error aggregation returns first error only

**Why:** To avoid overwhelming user with multiple errors

**Solution:** Show one error at a time (by design)

```typescript
// Current behavior: Shows first error
errorMessage = "First Name: First name must be at least 2 characters"

// User fixes it, then sees:
errorMessage = "Last Name: Last name must be at least 2 characters"
```

---

### Issue 4: Validation not running after setValue

**Cause:** Forgot to call `trigger()`

**Solution:**
```typescript
// ❌ WRONG (only updates value, doesn't validate)
setValue(fieldNames.firstName, value)

// ✅ CORRECT (updates and validates)
setValue(fieldNames.firstName, value)
trigger([fieldNames.firstName, fieldNames.middleName, fieldNames.lastName])
```

---

## Summary

| Stage | Component | Action | Result |
|-------|-----------|--------|--------|
| **Setup** | Story | Create schema, form, provider | Form context ready |
| **Input** | User | Type in input field | onChange fires |
| **Process** | FormNameInputGroup | Call setValue + trigger | Zod validates |
| **Store** | React Hook Forms | Store in formState.errors | Error available |
| **Aggregate** | FormNameInputGroup | Get field state, aggregate | isInvalid + errorMessage |
| **Render** | NameInputGroup | Pass props to base component | UI shows error |
| **Display** | FieldGroup | Apply styling based on isInvalid | Red border + error text |

---

## Quick Reference

### Key Files
- **Component:** `app/components/Aria/NameInputGroup.tsx`
- **Stories:** `app/components/Aria/NameInputGroup.stories.tsx`
- **Future schema:** `app/schema/name-schema.ts` (to be created)

### Key Methods
- `useForm()` - Initialize form with validation
- `zodResolver()` - Connect Zod to React Hook Forms
- `FormProvider` - Share form state via context
- `useFormContext()` - Access form state in components
- `watch()` - Subscribe to field changes
- `setValue()` - Update field value
- `trigger()` - Run validation
- `getFieldState()` - Get validation result

### Validation Modes
- `onChange` - Real-time validation
- `onBlur` - Validate when field loses focus
- `onSubmit` - Validate only on submission
- `onTouched` - Validate after interaction

---

**End of Document**
