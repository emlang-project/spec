# Emlang Specification

![License](https://img.shields.io/github/license/emlang-project/spec)
![GitHub release](https://img.shields.io/github/v/release/emlang-project/spec)

Emlang is a YAML-based DSL for describing systems with Event Modeling patterns.

## Quick Reference

| Type      | Short | Acronym | Long         | Description            |
|-----------|-------|---------|--------------|------------------------|
| Trigger   | `t:`  | `trg:`  | `trigger:`   | Initiates an action    |
| Command   | `c:`  | `cmd:`  | `command:`   | Action/intent          |
| Event     | `e:`  | `evt:`  | `event:`     | Past fact              |
| Exception | `x:`  | `err:`  | `exception:` | Failure/error          |
| View      | `v:`  | —       | `view:`      | Projection/Read Model  |

See [SPEC.md](SPEC.md) for the complete specification and [schema.json](schema.json) for validation.

## Examples

### User Registration

A complete flow showing a user registering through a form:

```yaml
---
flows:
  RegisterUser:
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
    - v: UserProfile
```

### E-Commerce Checkout

Multiple flows representing the checkout process:

```yaml
---
flows:
  StartCheckout:
    - t: Customer/Cart
    - c: StartCheckout
    - e: Order/CheckoutStarted
      props:
        orderId: uuid
        customerId: uuid
        items: array
    - v: CheckoutSummary

  SubmitPayment:
    - t: Customer/CheckoutSummary
    - c: SubmitPayment
      props:
        orderId: uuid
        paymentMethod: string
    - e: Order/PaymentSubmitted
    - e: Payment/PaymentProcessed
    - v: OrderConfirmation
```

### Multiple Flows with Tests

A document containing related flows and their tests:

```yaml
---
flows:
  RegisterUser:
    - t: Customer/RegistrationForm
    - c: RegisterUser
      props:
        email: string
        password: string
    - e: User/UserRegistered
      props:
        userId: uuid
        email: string
    - v: UserProfile

  LoginUser:
    - t: Customer/LoginForm
    - c: LoginUser
      props:
        email: string
        password: string
    - e: User/UserLoggedIn
      props:
        userId: uuid
        sessionId: uuid
    - v: Dashboard

tests:
  RegistrationCreatesProfile:
    when:
      - c: RegisterUser
        props:
          email: alice@example.com
          password: secret123
    then:
      - e: User/UserRegistered
        props:
          userId: user-1
          email: alice@example.com

  DuplicateEmailRejected:
    given:
      - e: User/UserRegistered
        props:
          email: alice@example.com
    when:
      - c: RegisterUser
        props:
          email: alice@example.com
          password: newpassword
    then:
      - x: EmailAlreadyUsed
```

### Event Storming Notes

Partial flows are valid — useful for early exploration:

```yaml
---
flows:
  OrderLifecycle:
    - e: OrderPlaced
    - e: PaymentReceived
    - e: OrderShipped
    - e: OrderDelivered

  OrderCancellation:
    - e: OrderPlaced
    - e: OrderCancelled
    - e: RefundIssued
```

## Tools

A linter is currently in development. More tools coming soon.

## Contributing

Open an issue before submitting a pull request.

## License

[MIT](LICENSE)
