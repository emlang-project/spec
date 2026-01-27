# Emlang Specification v0.3.0

Emlang is a YAML-based DSL for describing systems with Event Modeling patterns.

## Overview

Emlang uses standard YAML syntax, making it compatible with all YAML tooling (syntax highlighting, formatting,
validation). The domain-specific semantics come from the structure and element prefixes.

An Emlang file contains one or more YAML documents (separated by `---`), each with `slices:` and/or `tests:` top-level
keys.

```yaml
---
slices:
  # Direct form: list of elements
  SliceName:
    - t: Swimlane/TriggerName
    - c: DoSomething
    - e: SomethingDone

  # Extended form: steps with attached tests
  AnotherSlice:
    steps:
      - t: Swimlane/Trigger
      - c: DoAction
      - e: ActionDone
    tests:
      TestName:
        when:
          - c: DoAction
        then:
          - e: ActionDone

# Exploratory tests (not attached to a slice)
tests:
  ExploratoryTest:
    given:
      - e: PreviousEvent
    when:
      - c: DoSomething
    then:
      - e: ExpectedEvent
```

## Elements

Emlang has 5 element types, each with multiple prefix forms:

| Type      | Short | Acronym | Long         | Description           |
|-----------|-------|---------|--------------|------------------------|
| Trigger   | `t:`  | `trg:`  | `trigger:`   | Initiates an action    |
| Command   | `c:`  | `cmd:`  | `command:`   | Action/intent          |
| Event     | `e:`  | `evt:`  | `event:`     | Past fact              |
| Exception | `x:`  | `err:`  | `exception:` | Failure/error          |
| View      | `v:`  | —       | `view:`      | Projection/Read Model  |

All prefix forms are semantically equivalent. Choose based on readability preference.

### Element Format

Each element is a YAML list item with exactly one type key and optional properties:

```yaml
- c: PlaceOrder
  props:
    orderId: uuid
    items: array
```

An element must have exactly one type key (`t:`, `c:`, `e:`, `x:`, or `v:`). Multiple type keys in the same item is invalid:

```yaml
# ❌ Invalid: two type keys in the same item
- c: RegisterUser
  e: UserRegistered

# ✅ Valid: separate items
- c: RegisterUser
- e: UserRegistered
```

### Naming

Element names are free-form text. Spaces, accents, and special characters are allowed:

```yaml
- t: Customer/RegistrationForm
- c: RegisterUser
- e: Utilisateur enregistré
```

### Swimlanes

Swimlanes are optional and specified with `/` before the element name:

```yaml
- t: Customer/RegistrationForm   # Swimlane: Customer
- e: User/UserRegistered         # Swimlane: User
- c: RegisterUser                # No swimlane
```

### Properties

Properties are optional and defined under `props:`. The structure is free-form — consuming tools decide how to interpret
them.

```yaml
- c: RegisterUser
  props:
    email: string
    password: string

- e: User/UserRegistered
  props:
    userId: uuid
    email: string
    registeredAt: iso8601
```

Properties can also be a list:

```yaml
- c: SubmitOrder
  props:
    - orderId: uuid
    - customerId: uuid
    - items
```

## Slices

A slice is a named sequence of elements representing a business scenario.

### Direct Form

When a slice has no attached tests, use the direct form (list of elements):

```yaml
---
slices:
  UserRegistration:
    - t: Customer/RegistrationForm
    - c: RegisterUser
    - e: User/UserRegistered
    - v: UserProfile
```

### Extended Form

When a slice has attached tests, use the extended form with `steps:` and `tests:`:

```yaml
---
slices:
  UserRegistration:
    steps:
      - t: Customer/RegistrationForm
      - c: RegisterUser
      - e: User/UserRegistered
      - v: UserProfile
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

Both forms are valid and can coexist in the same document. A parser detects the form by checking if the slice value is a list (direct) or a mapping with `steps:` (extended).

### Multiple Slices

Multiple slices can be defined in the same document:

```yaml
---
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

### Anonymous Slices

A slice without a name is valid but not referenceable:

```yaml
---
slices:
  - t: Customer/ContactForm
  - c: SubmitInquiry
  - e: InquiryReceived
```

## Tests

A test verifies behavior using a Given-When-Then structure.

Tests can be defined in two places:

| Location | Purpose |
|----------|---------|
| Inside a slice (`tests:` in extended form) | Attached to a slice |
| At document root (`tests:`) | Exploratory tests, not yet attached to a slice |

Exploratory tests at the root level are useful during early modeling when slices are not yet well-defined. As the model matures, tests can be moved into their respective slices.

### Test Structure

Each test has three sections:

| Section  | Required | Contains                          |
|----------|----------|-----------------------------------|
| `given:` | No       | Pre-conditions (`e:`, `v:`)       |
| `when:`  | Yes      | Single command (`c:`)             |
| `then:`  | Yes      | Expected results (`e:`, `v:`, `x:`) |

```yaml
---
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

### Test with Exception

```yaml
---
tests:
  RateLimitExceeded:
    given:
      - e: User/PasswordResetRequested
        props:
          userId: 123
          requestedAt: 2024-01-24T10:00:00Z
    when:
      - c: RequestPasswordReset
        props:
          userId: 123
    then:
      - x: TooManyResetAttempts
```

### Test with View

Verify both events and view projections:

```yaml
---
tests:
  ProfileUpdatedAfterRegistration:
    when:
      - c: RegisterUser
        props:
          email: joe@example.com
    then:
      - e: UserRegistered
      - v: UserProfile
        props:
          email: joe@example.com
          status: pending
```

### Multiple Tests

```yaml
---
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

  EmailMustBeValid:
    when:
      - c: RegisterUser
        props:
          email: not-an-email
    then:
      - x: InvalidEmailFormat
```

## Document Structure

### Single Document

A document can contain both slices and tests:

```yaml
---
slices:
  UserRegistration:
    - t: Customer/RegistrationForm
    - c: RegisterUser
    - e: User/UserRegistered
    - v: UserProfile

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
```

### Multiple Documents

Use `---` to separate YAML documents in a single file:

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
    - t: Admin/UserManagement
    - c: DeleteUser
    - e: User/UserDeleted

---
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

Standard YAML comments with `#`:

```yaml
---
slices:
  OrderPlacement:
    - t: Customer/ProductPage    # User clicks "Buy Now"
    - c: PlaceOrder              # Validates inventory
      props:
        productId: uuid
        quantity: int
    - e: Order/OrderPlaced       # Core domain event
    - v: OrderConfirmation       # Shows order details
```

## Complete Example

```yaml
---
slices:
  # Extended form: slice with attached tests
  UserRegistration:
    steps:
      - t: Customer/RegistrationForm
      - c: RegisterUser
        props:
          email: string
          password: string
      - e: User/UserRegistered
        props:
          userId: uuid
          email: string
          registeredAt: iso8601
      - e: User/WelcomeEmailSent
      - v: UserProfile
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

      PasswordMustBeStrong:
        when:
          - c: RegisterUser
            props:
              email: jane@example.com
              password: "123"
        then:
          - x: PasswordTooWeak

  # Direct form: slice without attached tests
  EmailVerification:
    - t: User/VerificationLink
    - c: VerifyEmail
      props:
        token: string
    - e: User/EmailVerified
    - v: UserProfile

---
slices:
  OrderPlacement:
    steps:
      - t: Customer/Cart
      - c: PlaceOrder
        props:
          customerId: uuid
          items: array
      - e: Order/OrderPlaced
        props:
          orderId: uuid
          totalAmount: decimal
      - e: Inventory/StockReserved
      - e: Payment/PaymentRequested
      - v: OrderConfirmation
    tests:
      CannotCancelShippedOrder:
        given:
          - e: Order/OrderShipped
            props:
              orderId: order-123
        when:
          - c: CancelOrder
            props:
              orderId: order-123
        then:
          - x: OrderAlreadyShipped

  # Direct form
  OrderCancellation:
    - t: Customer/OrderDetails
    - c: CancelOrder
      props:
        orderId: uuid
        reason: string
    - e: Order/OrderCancelled
    - e: Inventory/StockReleased
    - e: Payment/RefundInitiated
    - v: OrderDetails

# Exploratory tests at root level (not yet attached to a slice)
tests:
  SomeEarlyIdea:
    when:
      - c: RefundOrder
    then:
      - e: Order/OrderRefunded
```