# Emlang Specification v0.2.0

Emlang is a YAML-based DSL for describing systems with Event Modeling patterns.

## Overview

Emlang v0.2.0 uses standard YAML syntax, making it compatible with all YAML tooling (syntax highlighting, formatting,
validation). The domain-specific semantics come from the structure and element prefixes.

An Emlang file contains one or more YAML documents (separated by `---`), each with `flows:` and/or `tests:` top-level
keys.

```yaml
---
flows:
  FlowName:
    - t: Swimlane/TriggerName
    - c: DoSomething
    - e: SomethingDone

tests:
  TestName:
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

Each element is a YAML list item with a type prefix and optional properties:

```yaml
- c: PlaceOrder
  props:
    orderId: uuid
    items: array
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

## Flows

A flow is a named sequence of elements representing a business scenario.

```yaml
---
flows:
  UserRegistration:
    - t: Customer/RegistrationForm
    - c: RegisterUser
    - e: User/UserRegistered
    - v: UserProfile
```

### Multiple Flows

Multiple flows can be defined in the same document:

```yaml
---
flows:
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

### Anonymous Flows

A flow without a name is valid but not referenceable:

```yaml
---
flows:
  - t: Customer/ContactForm
  - c: SubmitInquiry
  - e: InquiryReceived
```

## Tests

A test verifies behavior using a Given-When-Then structure.

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

A document can contain both flows and tests:

```yaml
---
flows:
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
flows:
  UserRegistration:
    - t: Customer/RegistrationForm
    - c: RegisterUser
    - e: User/UserRegistered

---
flows:
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
flows:
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
flows:
  UserRegistration:
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

  EmailVerification:
    - t: User/VerificationLink
    - c: VerifyEmail
      props:
        token: string
    - e: User/EmailVerified
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

---
flows:
  OrderPlacement:
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
```