# SOLID Principles & Design Patterns in Python

This repository is an educational codebase that shows how to evolve a payment-processing app by applying **SOLID principles** and a few classic design patterns.

## What you are looking at

There are two major areas:

1. `src/solid_principles/`
   - Small, focused examples for each SOLID principle.
   - Most principles are shown in `before.py` and `after.py` files to highlight improvements.
2. `src/payment_service/`
   - A larger "realistic" payment service that applies those ideas with protocols, builders, factories, validators, listeners, and decorators.

---

## Repository structure

```text
src/
  solid_principles/
    initial_code.py
    single_responsability/
      before.py
      after.py
    open_close/
      before.py
      after.py
    liskov_substitution/
      before.py
      after.py
    interfaces_segregation/
      before.py
      after.py
    dependency_inversion/
      before.py
      after.py

  payment_service/
    main.py
    service.py
    builder.py
    factory.py
    logging_service.py
    service_protocol.py
    decorator_protocol.py

    commons/
    processors/
    notifiers/
    validators/
    listeners/
    loggers/
```

---

## How the payment service works (end-to-end)

The default flow is orchestrated in `src/payment_service/main.py`:

1. Create domain inputs (`CustomerData`, `PaymentData`).
2. Use `PaymentServiceBuilder` to wire dependencies.
3. Build a `PaymentService` instance.
4. Call `process_transaction`.

Inside `PaymentService.process_transaction`:

1. Build a `Request` object.
2. Execute validator chain.
3. Process payment with selected processor.
4. Notify listeners (event style).
5. Send customer notification (email/SMS).
6. Persist transaction logs.

This gives you a single high-level entrypoint while keeping responsibilities separated.

---

## Important modules to learn first

### 1) `commons/`: core data contracts

`commons` defines the Pydantic models used throughout the project:
- `CustomerData`
- `ContactInfo`
- `PaymentData` and `PaymentType`
- `PaymentResponse`
- `Request`

If you understand these models, the rest of the code becomes much easier to follow.

### 2) `processors/`: payment backends

Payment backends implement `PaymentProcessorProtocol`:
- `StripePaymentProcessor`
- `LocalPaymentProcessor`
- `OfflinePaymentProcessor`

Some processors also implement:
- `RefundProcessorProtocol`
- `RecurringPaymentProcessorProtocol`

This is a practical example of interface segregation and substitutability.

### 3) `factory.py`: processor selection strategy

`PaymentProcessorFactory` chooses the correct processor based on `PaymentData.type` and currency.

Great place to study Open/Closed Principle tradeoffs.

### 4) `validators/`: chain-of-responsibility style validation

`ChainHandler` defines the chain structure; `CustomerHandler` is one concrete step.

Current chain validates customer data and demonstrates composition of validation steps.

### 5) `notifiers/`: pluggable user notifications

`NotifierProtocol` abstracts confirmation delivery.
`EmailNotifier` and `SMSNotifier` are interchangeable implementations.

### 6) `listeners/`: event subscribers

`ListenersManager` keeps a list of listeners and broadcasts events.

This decouples event reactions from the core transaction path.

### 7) `builder.py`: composition root

`PaymentServiceBuilder` constructs the service by selecting concrete dependencies and validating required parts before `build()`.

This is the central wiring point for runtime behavior.

---

## SOLID examples section (`solid_principles/`)

The `solid_principles` folder is your concept-first area:

- `initial_code.py`: a monolithic baseline that mixes concerns.
- Each principle folder:
  - `before.py`: code shape that violates or strains the principle.
  - `after.py`: refactored code showing improvement.

Recommended reading order:
1. `initial_code.py`
2. `single_responsability/before.py` then `after.py`
3. `open_close/before.py` then `after.py`
4. `liskov_substitution/before.py` then `after.py`
5. `interfaces_segregation/before.py` then `after.py`
6. `dependency_inversion/before.py` then `after.py`

---

## Local setup and run

### Install dependencies

Using `uv`:

```bash
uv sync
```

### Run the payment service example

```bash
cd src/payment_service
python main.py
```

> Note: Stripe paths need environment variables like `STRIPE_API_KEY` (and `STRIPE_PRICE_ID` for recurring setup).

---

## What to learn next (practical roadmap)

1. **Dependency Injection patterns in Python**
   - Move more wiring into a single composition root.
   - Consider container-based DI once the project grows.

2. **Typed protocols and static checks**
   - Add `mypy` and enforce protocol compatibility across processors/notifiers/listeners.

3. **Testing strategy by layer**
   - Unit tests for validators/factory/notifiers.
   - Integration tests for `PaymentService` orchestration.
   - Contract tests for processors.

4. **Error handling and observability**
   - Replace `print` calls with structured logging.
   - Introduce domain exceptions for validation/business failures.

5. **Packaging/import hygiene**
   - Convert some absolute imports to package-relative imports so the app is easier to run from project root.

6. **Extension exercises**
   - Add a new notifier (e.g., push notifications).
   - Add another processor (e.g., PayPal).
   - Add a `PaymentDataHandler` to chain validations.
   - Add persistence abstraction for logs/events.

---

## Quick mental model

- `commons`: shared data shapes.
- `processors`: *how* payment happens.
- `factory`: chooses processor.
- `validators`: gate input correctness.
- `notifiers`: tell customer outcome.
- `listeners`: side effects on domain events.
- `loggers`: persistence/trace.
- `service`: orchestration.
- `builder`: configuration and assembly.

If you start with that model, most files in this project will feel intuitive very quickly.
