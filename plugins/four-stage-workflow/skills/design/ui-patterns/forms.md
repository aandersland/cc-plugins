---
name: forms
description: Form patterns that feel like a conversation, minimizing effort and maximizing clarity.
---

# Form Patterns

## Source
- Book: Designing Interfaces (3rd Edition)
- Author: Jenifer Tidwell
- Key Chapters: 8 (Getting Input from Users)

## Core Principle
Forms should feel like a conversation, not an interrogation. Minimize effort, maximize clarity.

## Patterns

### Pattern 1: Single-Column Layout
**When:** Any form
**Apply:** One field per row, labels above

```
Label
┌────────────────────────────────┐
│ Input                          │
└────────────────────────────────┘

Label
┌────────────────────────────────┐
│ Input                          │
└────────────────────────────────┘

        [Primary Action]
```

Rules:
- Labels above fields (not beside)
- Full width on mobile
- Group related fields visually

### Pattern 2: Inline Validation
**When:** Validation rules are clear
**Apply:** Validate on blur, not on keystroke

```
Email
┌────────────────────────────────┐
│ invalid-email                  │ ✗
└────────────────────────────────┘
  Please enter a valid email address

Password
┌────────────────────────────────┐
│ ••••••••                       │ ✓
└────────────────────────────────┘
```

Rules:
- Show success state for valid fields
- Error message below field, not in popup
- Don't validate while typing (frustrating)

### Pattern 3: Progressive Disclosure
**When:** Form has optional/advanced fields
**Apply:** Hide complexity until needed

```
Basic Information
┌────────────────────────────────┐
│ [Required fields]              │
└────────────────────────────────┘

[+ Show Advanced Options]
    ↓ (when clicked)
┌────────────────────────────────┐
│ [Optional fields revealed]     │
└────────────────────────────────┘
```

### Pattern 4: Smart Defaults
**When:** Reasonable defaults exist
**Apply:** Pre-fill with sensible values

```
Country: [🇺🇸 United States ▼]  ← Based on IP
Timezone: [America/New_York ▼]  ← Based on browser
Date format: [MM/DD/YYYY ▼]     ← Based on locale
```

Rules:
- Use browser/system info
- Remember previous user choices
- Make it easy to change

### Pattern 5: Input Masks & Formatting
**When:** Specific format required
**Apply:** Format as user types

```
Phone: (555) 123-4567     ← Auto-formatted
Card:  4242 4242 4242 4242
Date:  01/25/2024
```

Rules:
- Show expected format in placeholder
- Allow paste of unformatted data
- Strip formatting on submission

### Pattern 6: Autosave
**When:** Long forms or data entry
**Apply:** Save automatically, show status

```
┌─────────────────────────────────────┐
│                        Saved ✓     │
│  [Form content...]                 │
│                                    │
│  Last saved: 2 seconds ago         │
└─────────────────────────────────────┘
```

Rules:
- Save on blur or after pause in typing
- Show clear save status
- Handle offline gracefully

### Pattern 7: Error Summary
**When:** Multiple validation errors
**Apply:** Summary at top + inline errors

```
┌─────────────────────────────────────┐
│ ⚠ Please fix 2 errors:             │
│ • Email is required                │
│ • Password must be 8+ characters   │
└─────────────────────────────────────┘

Email *
┌────────────────────────────────┐
│                                │ ← Highlighted
└────────────────────────────────┘
  Email is required
```

## Anti-Patterns

- **Inline Labels (Placeholder as Label):** Disappears, accessibility issues
- **Asterisk Overload:** Mark optional fields instead if most are required
- **Reset Buttons:** Almost never needed, causes accidental data loss
- **All Fields Required:** Question every "required" - is it really?
- **CAPTCHA Overuse:** Adds friction, use invisible alternatives

## Checklist

- [ ] Labels above fields, not inline
- [ ] Error messages specific and actionable
- [ ] Tab order logical
- [ ] Primary action clearly distinguished
- [ ] Form not wider than needed
- [ ] Optional fields marked (if minority)
- [ ] Validation on blur, not keystroke

## Claude Code Integration
**Command:** /ux-review
**Input:** Form code or design
**Output:** Form pattern violations and fixes
