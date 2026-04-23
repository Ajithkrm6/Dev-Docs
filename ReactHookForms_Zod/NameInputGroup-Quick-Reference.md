# NameInputGroup Quick Reference Guide

**Version:** 1.0  
**Last Updated:** April 23, 2026

---

## 📋 Table of Contents

1. [Quick Start](#quick-start)
2. [Component API](#component-api)
3. [Validation Modes](#validation-modes)
4. [Common Patterns](#common-patterns)
5. [Schema Examples](#schema-examples)
6. [Error Handling](#error-handling)
7. [Performance Tips](#performance-tips)

---

## 🚀 Quick Start

### Minimal Example

```typescript
import { FormNameInputGroup } from "~/components/Aria/NameInputGroup"
import { zodResolver } from "@hookform/resolvers/zod"
import { useForm, FormProvider } from "react-hook-form"
import { z } from "zod"

// 1. Define schema
const schema = z.object({
  firstName: z.string().min(1),
  middleName: z.string().optional(),
  lastName: z.string().min(1),
})

// 2. Create form
const methods = useForm({
  resolver: zodResolver(schema),
  mode: "onChange"
})

// 3. Use in component
export function MyForm() {
  return (
    <FormProvider {...methods}>
      <form onSubmit={methods.handleSubmit(onSubmit)}>
        <FormNameInputGroup
          fieldNames={{
            firstName: "firstName",
            middleName: "middleName",
            lastName: "lastName"
          }}
          label="Full Name"
        />
        <button type="submit">Submit</button>
      </form>
    </FormProvider>
  )
}

function onSubmit(data: any) {
  console.log("Valid data:", data)
}
```

---

## 📚 Component API

### NameInputGroup Props

```typescript
interface NameInputGroupProps {
  // Display
  label?: string
  description?: string
  placeholder?: string
  helpText?: ReactNode
  className?: string
  
  // Field Values (required)
  firstName: string
  middleName: string
  lastName: string
  
  // Change Handlers (required)
  onChangeFirstName: (v: string) => void
  onChangeMiddleName: (v: string) => void
  onChangeLastName: (v: string) => void
  
  // Validation State
  isInvalid?: boolean
  errorMessage?: string
  
  // Interaction State
  isRequired?: boolean
  isDisabled?: boolean
  isReadOnly?: boolean
}
```

### FormNameInputGroup Props

```typescript
interface FormNameInputGroupProps<T extends FieldValues> {
  // Required - Field paths in your form data
  fieldNames: {
    firstName: FieldPath<T>
    middleName: FieldPath<T>
    lastName: FieldPath<T>
  }
  
  // Optional - Display props (same as NameInputGroup)
  label?: string
  description?: string
  placeholder?: string
  helpText?: ReactNode
  className?: string
  isRequired?: boolean
  isDisabled?: boolean
  isReadOnly?: boolean
}
```

---

## 🎛️ Validation Modes

### onChange - Real-time Validation

```typescript
useForm({
  resolver: zodResolver(schema),
  mode: "onChange"  // ← Validate on every keystroke
})
```

**When to use:**
- Search fields
- Real-time feedback needed
- Quick confirmation

**Example:**
```
User types "J" → Validate immediately → Show error
User types "Jo" → Validate immediately → Still error
User types "John" → Validate immediately → Error gone
```

---

### onBlur - After Focus Loss

```typescript
useForm({
  resolver: zodResolver(schema),
  mode: "onBlur"  // ← Validate when field loses focus
})
```

**When to use:**
- Most common choice
- Balanced UX
- Reduce validation calls

**Example:**
```
User types "J" in field → No validation yet
User clicks other field → Field loses focus → Validate → Show error
User focuses firstName again → No error shown yet (not dirty)
User types "John" → Still no validation (only on blur)
User clicks other field → Validate → Error gone
```

---

### onSubmit - Only on Submit

```typescript
useForm({
  resolver: zodResolver(schema),
  mode: "onSubmit"  // ← Validate only on form submit
})
```

**When to use:**
- Complex forms
- Expensive validations
- Initial form view should be clean

**Example:**
```
Page loads → No validation, no errors shown
User types anything → No validation
User clicks Submit → Validate all fields → Show errors if any
```

---

### onTouched - After Interaction

```typescript
useForm({
  resolver: zodResolver(schema),
  mode: "onTouched"  // ← Validate after field touched AND dirty
})
```

**When to use:**
- Polite validation
- Don't overwhelm new users

**Example:**
```
Field focused → Nothing happens
User types one character → Nothing (not yet dirty)
User continues typing → Nothing (already dirty)
User clicks another field → Validate → Show error
```

---

## 🔄 Common Patterns

### Pattern 1: Basic Form

```typescript
export function ContactForm() {
  const schema = z.object({
    firstName: z.string().min(1, "Required"),
    middleName: z.string().optional(),
    lastName: z.string().min(1, "Required"),
  })
  
  const methods = useForm({
    resolver: zodResolver(schema),
    mode: "onBlur"
  })
  
  return (
    <FormProvider {...methods}>
      <form onSubmit={methods.handleSubmit(onSubmit)}>
        <FormNameInputGroup
          fieldNames={{
            firstName: "firstName",
            middleName: "middleName",
            lastName: "lastName"
          }}
          label="Contact Name"
          description="Enter the person's full name"
          isRequired
        />
        <button type="submit">Save</button>
      </form>
    </FormProvider>
  )
}
```

---

### Pattern 2: Nested Object

```typescript
export function ApplicantForm() {
  const schema = z.object({
    applicant: z.object({
      firstName: z.string().min(1),
      middleName: z.string().optional(),
      lastName: z.string().min(1),
    })
  })
  
  const methods = useForm({
    resolver: zodResolver(schema),
  })
  
  return (
    <FormProvider {...methods}>
      <FormNameInputGroup
        fieldNames={{
          firstName: "applicant.firstName",        // ← Nested path
          middleName: "applicant.middleName",
          lastName: "applicant.lastName"
        }}
        label="Applicant"
      />
    </FormProvider>
  )
}
```

---

### Pattern 3: Array Item

```typescript
export function ContactListForm() {
  const schema = z.object({
    contacts: z.array(z.object({
      firstName: z.string().min(1),
      middleName: z.string().optional(),
      lastName: z.string().min(1),
    }))
  })
  
  const methods = useForm({
    resolver: zodResolver(schema),
  })
  
  return (
    <FormProvider {...methods}>
      {methods.watch("contacts").map((_, index) => (
        <FormNameInputGroup
          key={index}
          fieldNames={{
            firstName: `contacts.${index}.firstName`,    // ← Array index
            middleName: `contacts.${index}.middleName`,
            lastName: `contacts.${index}.lastName`
          }}
          label={`Contact ${index + 1}`}
        />
      ))}
    </FormProvider>
  )
}
```

---

### Pattern 4: Conditional Validation

```typescript
const schema = z.object({
  firstName: z.string().min(1, "Required"),
  middleName: z.string().optional(),
  lastName: z.string().min(1, "Required"),
}).refine(
  (data) => {
    // Cross-field validation example
    if (data.firstName === data.lastName) {
      return false
    }
    return true
  },
  {
    message: "First and last name cannot be the same",
    path: ["firstName"]
  }
)
```

---

## 📝 Schema Examples

### Example 1: Basic Required Fields

```typescript
const basicSchema = z.object({
  firstName: z.string().min(1, "First name is required"),
  middleName: z.string().optional(),
  lastName: z.string().min(1, "Last name is required"),
})
```

---

### Example 2: With Length Constraints

```typescript
const lengthSchema = z.object({
  firstName: z
    .string()
    .min(1, "First name is required")
    .min(2, "First name must be at least 2 characters")
    .max(50, "First name cannot exceed 50 characters"),
  
  middleName: z
    .string()
    .max(50)
    .optional(),
  
  lastName: z
    .string()
    .min(1, "Last name is required")
    .min(2, "Last name must be at least 2 characters")
    .max(50, "Last name cannot exceed 50 characters"),
})
```

---

### Example 3: With Pattern/Format

```typescript
const formatSchema = z.object({
  firstName: z
    .string()
    .min(1)
    .regex(/^[a-zA-Z\s'-]+$/, "Only letters, spaces, hyphens, and apostrophes allowed"),
  
  middleName: z
    .string()
    .regex(/^[a-zA-Z\s'-]*$/, "Invalid characters")
    .optional(),
  
  lastName: z
    .string()
    .min(1)
    .regex(/^[a-zA-Z\s'-]+$/, "Invalid characters"),
})
```

---

### Example 4: Cross-Field Validation

```typescript
const crossFieldSchema = z.object({
  firstName: z.string().min(1),
  middleName: z.string().optional(),
  lastName: z.string().min(1),
}).refine(
  (data) => {
    // Example: Ensure at least one field has value
    return data.firstName || data.lastName
  },
  {
    message: "At least first or last name is required",
    path: ["firstName"]
  }
).refine(
  (data) => {
    // Example: Prevent duplicate names
    if (data.firstName && data.lastName) {
      return data.firstName.toLowerCase() !== data.lastName.toLowerCase()
    }
    return true
  },
  {
    message: "First and last names must be different",
    path: ["firstName"]
  }
)
```

---

## ⚠️ Error Handling

### Display Single Error

```typescript
// Component already does this
<div className="text-red-500">
  {errorMessage}  {/* Shows first error only */}
</div>
```

---

### Display All Errors (Custom)

```typescript
// If you need to show all field errors separately:
export function AllErrorsDisplay({ formState }: any) {
  return (
    <>
      {formState.errors.firstName && (
        <div className="text-red-500">{formState.errors.firstName.message}</div>
      )}
      {formState.errors.lastName && (
        <div className="text-red-500">{formState.errors.lastName.message}</div>
      )}
    </>
  )
}
```

---

### Handle Submit Errors

```typescript
const methods = useForm({...})

function onSubmit(data: NameFormData) {
  try {
    // Send to API
    await api.saveContact(data)
  } catch (error) {
    // Show error toast
    toast.error("Failed to save contact")
  }
}

function onError(errors: FieldErrors<NameFormData>) {
  // Called if validation fails
  console.error("Validation errors:", errors)
}

<form onSubmit={methods.handleSubmit(onSubmit, onError)}>
```

---

## ⚡ Performance Tips

### Tip 1: Use onBlur Instead of onChange

```typescript
// ❌ SLOWER - Validates on every keystroke
useForm({ mode: "onChange" })

// ✅ FASTER - Validates only on blur
useForm({ mode: "onBlur" })
```

---

### Tip 2: Memoize Field Names

```typescript
// ❌ RECREATES OBJECT EVERY RENDER
<FormNameInputGroup
  fieldNames={{
    firstName: "firstName",
    middleName: "middleName",
    lastName: "lastName"
  }}
/>

// ✅ MEMOIZE ONCE
const fieldNames = useMemo(() => ({
  firstName: "firstName",
  middleName: "middleName",
  lastName: "lastName"
}), [])

<FormNameInputGroup fieldNames={fieldNames} />
```

---

### Tip 3: Defer Non-Essential Validation

```typescript
// ❌ ALL fields validate on change
useForm({ mode: "onChange" })

// ✅ USE DEFERRED VALIDATION
useForm({ 
  mode: "onBlur",  // Only on blur
  shouldUseNativeValidation: true
})
```

---

## 🔗 Integration Examples

### With Bidder Registration Form

```typescript
// app/routes/bid/personal/_index/form.tsx
import { FormNameInputGroup } from "~/components/Aria/NameInputGroup"
import { bidRegisterSchema } from "~/schema/bid-register-schema"

export function PersonalDetailsForm() {
  const methods = useForm({
    resolver: zodResolver(bidRegisterSchema),
    mode: "onBlur"
  })
  
  return (
    <FormProvider {...methods}>
      <form onSubmit={methods.handleSubmit(submitRegistration)}>
        <FormNameInputGroup
          fieldNames={{
            firstName: "personalInfo.firstName",
            middleName: "personalInfo.middleName",
            lastName: "personalInfo.lastName"
          }}
          label="Full Name"
          description="Your legal name"
          isRequired
        />
        <button type="submit">Next</button>
      </form>
    </FormProvider>
  )
}
```

---

### With Vehicle Consignment Form

```typescript
// app/routes/consign.vehicle.applicant._index/form.tsx
export function ApplicantDetailsForm() {
  const schema = bidRegisterSchema.pick({
    personalInfo: true
  })
  
  const methods = useForm({
    resolver: zodResolver(schema),
  })
  
  return (
    <FormProvider {...methods}>
      <FormNameInputGroup
        fieldNames={{
          firstName: "personalInfo.firstName",
          middleName: "personalInfo.middleName",
          lastName: "personalInfo.lastName"
        }}
        label="Consignor Name"
      />
    </FormProvider>
  )
}
```

---

## 🎨 Styling

### Dark Mode Support

```typescript
<FormNameInputGroup
  fieldNames={fieldNames}
  className="dark:bg-gray-900"  // Dark background
/>
```

---

### Custom Width

```typescript
<FormNameInputGroup
  fieldNames={fieldNames}
  className="max-w-sm"  // Small: 384px
/>

<FormNameInputGroup
  fieldNames={fieldNames}
  className="max-w-md"  // Medium: 448px
/>

<FormNameInputGroup
  fieldNames={fieldNames}
  className="max-w-2xl"  // Large: 672px
/>
```

---

### Responsive Behavior

The component is already responsive:
- **Mobile (default):** Inputs stack vertically
- **Tablet (md:):** Inputs align horizontally

No additional styling needed!

---

## 📞 Support

For issues or questions:
1. Check the full guide: `NameInputGroup-Validation-Guide.md`
2. View the flowchart: `NameInputGroup-Validation-Flowchart.md`
3. Check component stories: `NameInputGroup.stories.tsx`

---

**End of Quick Reference**
