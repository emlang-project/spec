# Emlang Specification v0.1.0

Emlang is a text-based DSL for describing systems with Event Modeling patterns.

File extension: `.emlang`

## Elements

Emlang has 7 element types:

| Notation | Type      | Description           |
|----------|-----------|-----------------------|
| `/.../`  | Flow      | Business scenario     |
| `%...%`  | Trigger   | Initiates an action   |
| `[...]`  | Command   | Action/intent         |
| `<...>`  | Event     | Past fact             |
| `!...!`  | Exception | Failure/error         |
| `{...}`  | View      | Projection/Read Model |
| `?...?`  | Test      | Given-When-Then test  |

### Naming

Element names allow letters (`a-z`, `A-Z`), digits (`0-9`), hyphen (`-`), underscore (`_`), and period (`.`). No spaces.
Case-sensitive.

Flow names also allow `/` for hierarchy: `/UserManagement/RegisterUser/`

### Swimlanes

Triggers and Events can be placed in named swimlanes using `:`.

```
%Customer:RegistrationForm%
```

```
<User:UserRegistered>
```

### Attributes

Triggers, Commands, Events, Exceptions, and Views can have attributes. Attributes are free-form text, each on its own
indented line.

```
[RegisterUser]
  userId:uuid
  email:string
  password:string
```

Flows and Tests do not have attributes.

## Document Structure

An Emlang document contains **flows** and **tests** at root level, separated by `---`:

```emlang
/RegisterUser/
  %RegistrationForm%
  [RegisterUser]
  <UserRegistered>
  {UserProfile}
---
/DeleteUser/
  %UserProfile%
  [DeleteUser]
  <UserDeleted>
---
?EmailMustBeUnique?
  <UserRegistered>
    email:joe@example.com
  [RegisterUser]
    email:joe@example.com
  !EmailAlreadyUsed!
```

## Flows

A flow is a chronological sequence of elements representing a business scenario.

All elements (Trigger, Command, Event, Exception, View) must be inside a flow. Contents are indented one level. Elements
within a flow are at the same indentation level.

```emlang
/RegisterUser/
  %RegistrationForm%
  [RegisterUser]
    email
    password
  <UserRegistered>
```

A flow can contain multiple **sequences**, separated by `---`. This represents a chronological break:

```emlang
/UserLifecycle/
  [RegisterUser]
  <UserRegistered>
  ---
  [DeleteUser]
  <UserDeleted>
```

## Tests

A test verifies behavior using a Given-When-Then structure.

Elements allowed in tests: Event, View, Command, Exception.

Interpretation:

- Elements before Command = **Given** (pre-existing state)
- Command = **When** (action)
- Elements after Command = **Then** (expected results)

```emlang
?EmailMustBeUnique?
  <UserRegistered>
    email:joe@example.com
  [RegisterUser]
    email:joe@example.com
  !EmailAlreadyUsed!
```

## Comments

Comments start with `;` and continue to end of line.

```
; User registration flow
[RegisterUser]  ; Requires valid email
```

## Whitespace

Blank lines have no semantic meaning and can be used freely.
