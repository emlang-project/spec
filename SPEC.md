# Emlang Specification v1.0.0

Emlang is a YAML-based DSL for describing systems with Event Modeling patterns.

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [RFC 2119](https://www.rfc-editor.org/rfc/rfc2119).

## Overview

- An Emlang file MUST be a valid YAML file
- Each YAML document MUST contain a `slices` top-level key
- `slices` MAY be empty (null value in YAML)
- A file MAY contain multiple YAML documents separated by `---`
- The root object MUST NOT contain properties other than `slices`

```yaml
---
slices:
  SliceName:
    - t: Swimlane/TriggerName
    - c: DoSomething
    - e: SomethingDone
```

## Elements

Emlang defines 5 element types, each with multiple prefix forms:

| Type      | Short | Acronym | Long         |
|-----------|-------|---------|--------------|
| Trigger   | `t:`  | `trg:`  | `trigger:`   |
| Command   | `c:`  | `cmd:`  | `command:`   |
| Event     | `e:`  | `evt:`  | `event:`     |
| Exception | `x:`  | `err:`  | `exception:` |
| View      | `v:`  | —       | `view:`      |

- All prefix forms of a given type are semantically equivalent
- An element MUST contain exactly one type key
- An element MUST NOT contain more than one type key
- An element MAY contain a `props` key
- An element MUST NOT contain keys other than one type key and `props`

```yaml
# ✅ Valid
- c: RegisterUser
- e: UserRegistered

# ❌ Invalid: two type keys in the same item
- c: RegisterUser
  e: UserRegistered
```

### Naming

- Element names are free-form text
- Names MUST NOT be empty
- Names MUST NOT end with `/` (a swimlane prefix alone is not a valid name)
- Names MAY contain spaces, accents, and special characters

```yaml
# ✅ Valid
- c: RegisterUser
- e: Utilisateur enregistré

# ❌ Invalid: empty name
- e:
- e: foo/
- e: /
```

### Swimlanes

- Swimlanes are OPTIONAL
- A swimlane MUST be specified as a prefix separated by `/` before the element name

```yaml
- t: Customer/RegistrationForm   # Swimlane: Customer
- e: User/UserRegistered         # Swimlane: User
- c: RegisterUser                # No swimlane
```

### Properties

- Properties are OPTIONAL
- Properties MUST be defined under a `props` key
- The `props` value MUST be an object or empty (null value in YAML)
- The structure of `props` is free-form — consuming tools decide how to interpret them

```yaml
- c: RegisterUser
  props:
    email: string
    password: string
```

## Slices

- A slice is a named sequence of elements representing a business scenario
- A slice MAY be empty (null value in YAML)
- A non-empty slice MUST be in either direct form or extended form

```yaml
# ✅ Valid: empty slices
slices:
  FooBar:
  BarBaz:
```

### Direct Form

- The direct form is a list of elements
- SHOULD be used when a slice has no attached tests

```yaml
slices:
  UserRegistration:
    - t: Customer/RegistrationForm
    - c: RegisterUser
    - e: User/UserRegistered
    - v: UserProfile
```

### Extended Form

- The extended form is an object with `steps` and optionally `tests`
- `steps` is REQUIRED but MAY be empty (null value in YAML)
- `tests` is OPTIONAL and MAY be empty (null value in YAML)
- The extended form MUST NOT contain properties other than `steps` and `tests`

```yaml
slices:
  UserRegistration:
    steps:
      - t: Customer/RegistrationForm
      - c: RegisterUser
      - e: User/UserRegistered
    tests:
      EmailMustBeUnique:
        given:
          - e: User/UserRegistered
            props:
              email: joe@example.com
        when:
          - c: RegisterUser
            props:
              email: joe@example.com
        then:
          - x: EmailAlreadyInUse
```

### Multiple Slices

- Multiple slices MAY be defined in the same document
- Both direct and extended forms MAY coexist in the same document

```yaml
slices:
  UserRegistration:
    - t: Customer/RegistrationForm
    - c: RegisterUser
    - e: User/UserRegistered

  UserLogin:
    - t: Customer/LoginForm
    - c: AuthenticateUser
    - e: User/UserAuthenticated
    - v: Dashboard
```

## Tests

- Tests MUST be defined inside a slice using the extended form
- `tests` MAY be empty (null value in YAML)
- Each test follows a Given-When-Then structure

### Test Structure

| Section  | Required | Allowed element types      |
|----------|----------|----------------------------|
| `given`  | No       | `e` (event), `v` (view)    |
| `when`   | No       | `c` (command)              |
| `then`   | No       | `e` (event), `v` (view), `x` (exception) |

- `given`, `when`, and `then` are all OPTIONAL and MAY be empty (null value in YAML)
- A test MAY be empty
- If `given` is present and non-empty, its elements MUST be events or views
- If `when` is present and non-empty, its elements MUST be commands
- If `then` is present and non-empty, its elements MUST be events, views, or exceptions
- A test MUST NOT contain properties other than `given`, `when`, and `then`

```yaml
tests:
  EmailMustBeUnique:
    given:
      - e: User/UserRegistered
        props:
          email: joe@example.com
    when:
      - c: RegisterUser
        props:
          email: joe@example.com
    then:
      - x: EmailAlreadyInUse
```

### Multiple Tests

- A slice MAY contain multiple named tests

```yaml
tests:
  EmailMustBeUnique:
    given:
      - e: UserRegistered
        props:
          email: joe@example.com
    when:
      - c: RegisterUser
        props:
          email: joe@example.com
    then:
      - x: EmailAlreadyInUse

  PasswordMustBeStrong:
    when:
      - c: RegisterUser
        props:
          password: "123"
    then:
      - x: PasswordTooWeak

  TodoCompleteRegistrationFlow:
```

## Document Structure

- A file MAY contain one or more YAML documents
- Documents MUST be separated by `---`
- Each document MUST independently conform to this specification

```yaml
---
slices:
  UserRegistration:
    - t: Customer/RegistrationForm
    - c: RegisterUser
    - e: User/UserRegistered

---
slices:
  UserDeletion:
    steps:
      - t: Admin/UserManagement
      - c: DeleteUser
      - e: User/UserDeleted
    tests:
      CannotDeleteActiveUser:
        given:
          - e: User/UserAuthenticated
        when:
          - c: DeleteUser
        then:
          - x: UserCurrentlyActive
```

## Comments

- Standard YAML comments (`#`) MAY be used anywhere

```yaml
slices:
  OrderPlacement:
    - t: Customer/ProductPage    # User clicks "Buy Now"
    - c: PlaceOrder              # Validates inventory
    - e: Order/OrderPlaced       # Core domain event
```
