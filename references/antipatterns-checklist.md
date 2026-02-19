# Test Antipatterns Checklist

## The Liar

**What**: Test that passes but doesn't actually verify anything meaningful.

**Signs**:
- No assertions at all
- Only asserts that code "doesn't throw"
- Asserts on a mock's return value (you set it up yourself!)
- Asserts on always-true conditions

```python
# BAD — The Liar: no real assertion
def test_process_order():
    order = Order(items=[Item(price=10)])
    result = process(order)
    assert result is not None  # This tells us nothing about correctness

# GOOD — meaningful assertion
def test_process_order_calculates_total():
    order = Order(items=[Item(price=10), Item(price=20)])
    result = process(order)
    assert result.total == 30
    assert result.status == "processed"
```

**Fix**: Every test must assert on a specific, meaningful expected outcome.

---

## The Giant

**What**: One massive test that verifies everything.

**Signs**:
- 50+ lines of setup
- Multiple Act steps
- 10+ assertions checking unrelated things
- Failures are hard to diagnose

```python
# BAD — The Giant
def test_order_system():
    # 30 lines of setup...
    order = create_order(...)
    order.validate()
    assert order.is_valid
    order.calculate_total()
    assert order.total == 100
    order.apply_discount(coupon)
    assert order.total == 85
    order.submit()
    assert order.status == "submitted"
    assert email_sent()
    assert inventory_decremented()
    # ... 15 more assertions
```

**Fix**: Split into focused tests — one behavior per test.

---

## The Mockery (Over-Mocking)

**What**: Test is entirely mocks — no real behavior being tested.

**Signs**:
- Mock setup exceeds test logic
- Mocking the class under test
- Mocking value objects
- Deeply chained mock returns
- Test passes regardless of actual implementation

```python
# BAD — The Mockery
def test_order_service():
    mock_repo = Mock()
    mock_validator = Mock()
    mock_calculator = Mock()
    mock_notifier = Mock()
    mock_repo.find.return_value = Mock(status="pending")
    mock_calculator.calculate.return_value = 100
    mock_validator.validate.return_value = True

    service = OrderService(mock_repo, mock_validator, mock_calculator, mock_notifier)
    service.process(order_id=1)

    mock_calculator.calculate.assert_called_once()  # So what?
    mock_notifier.notify.assert_called_once()        # Coupled to implementation
```

**Fix**: Use real objects for in-process dependencies. Mock only unmanaged out-of-process dependencies. If you need many mocks, the class has too many dependencies — refactor it.

---

## The Inspector (Testing Private Methods)

**What**: Test reaches into private/internal implementation to verify.

**Signs**:
- Accessing private fields (`obj._private_field`)
- Testing private/protected methods directly
- Using reflection to bypass access control
- Test name references internal methods

```python
# BAD — testing implementation detail
def test_internal_cache_populated():
    service = UserService()
    service.get_user(1)
    assert service._cache[1] is not None  # Private field!
    assert service._cache_hits == 0        # Implementation detail!

# GOOD — test observable behavior
def test_second_call_returns_same_user():
    service = UserService()
    user1 = service.get_user(1)
    user2 = service.get_user(1)
    assert user1 == user2  # Behavior: same result
```

**Fix**: Test only through public API. If private logic is complex enough to need its own tests, extract it into a separate class.

---

## The Slow Poke

**What**: Tests that take seconds or minutes to run.

**Signs**:
- `sleep()` or `time.sleep()` in tests
- Real network calls or database queries
- File system I/O in unit tests
- Spinning up containers in unit test suite

```python
# BAD — The Slow Poke
def test_retry_logic():
    service = Service(retry_delay=5)  # Real 5-second delay
    with pytest.raises(TimeoutError):
        service.call_external_api()
    # Test takes 15+ seconds (3 retries × 5 seconds)

# GOOD — inject time control
def test_retry_logic():
    clock = FakeClock()
    service = Service(retry_delay=5, clock=clock)
    service.call_external_api()
    assert clock.advanced_by == 15  # Verified timing without waiting
```

**Fix**: Inject clocks, use fakes for I/O, keep unit tests under 100ms each.

---

## Ice Cream Cone (Inverted Pyramid)

**What**: Mostly E2E/integration tests, few unit tests.

**Signs**:
- Test suite takes 20+ minutes
- Flaky tests everywhere
- Hard to pinpoint failures
- Developers skip tests locally
- CI is the bottleneck

**Fix**: Follow the test pyramid. Push logic into pure functions and domain objects. Test business rules with unit tests, boundaries with integration tests.

---

## Fragile Tests (Implementation Coupling)

**What**: Tests break when refactoring even though behavior is unchanged.

**Signs**:
- Verifying exact method call sequences
- Asserting on internal data structures
- Mocking internal collaborators
- Tests break when you rename a private method

```python
# BAD — Fragile: coupled to implementation
def test_user_creation():
    mock_repo = Mock()
    service = UserService(mock_repo)
    service.create_user("john@test.com")

    # These break if you refactor internal flow
    mock_repo.check_exists.assert_called_once_with("john@test.com")
    mock_repo.save.assert_called_once()
    mock_repo.save.assert_called_with(ANY)

# GOOD — test behavior, not implementation
def test_user_creation():
    repo = InMemoryUserRepo()
    service = UserService(repo)
    service.create_user("john@test.com")

    assert repo.find_by_email("john@test.com") is not None
```

**Fix**: Test observable outcomes (return values, state changes, external communications), not internal call sequences.

---

## Second Class Citizen

**What**: Test code treated as less important than production code.

**Signs**:
- Copy-pasted tests with minor variations
- Magic numbers without explanation
- No helper functions or builders
- Inconsistent naming
- Dead/commented-out tests
- No refactoring of test code

**Fix**: Apply the same quality standards to test code. Extract helpers, use builders, name clearly, refactor duplication.

---

## The 100% Coverage Trap

**What**: Pursuing 100% code coverage as a goal, leading to useless tests.

**Signs**:
- Tests for getters/setters/constructors
- Tests that exercise code without meaningful assertions
- Coverage is high but bugs still slip through
- Tests are written after-the-fact just for coverage numbers

```python
# BAD — testing trivial code for coverage
def test_user_name_getter():
    user = User(name="John")
    assert user.name == "John"  # Tests the language, not your logic

def test_user_constructor():
    user = User(name="John", email="j@t.com")
    assert user.name == "John"
    assert user.email == "j@t.com"
```

**Fix**: Coverage is a metric, not a goal. Aim for high coverage of **domain logic** and **critical paths**. Trivial code doesn't need tests. 80% meaningful coverage > 100% checkbox coverage.

---

## Testing Trivial Code

**What**: Writing tests for code with zero domain logic.

**What NOT to test**:
- Simple getters/setters
- Data transfer objects (DTOs)
- One-line delegations
- Configuration constants
- Auto-generated code (ORM models, protobuf)

**What TO test**:
- Calculated properties with logic
- Validation rules
- Custom serialization/deserialization
- Constructors with business rules
- Anything that could break in a meaningful way

---

## Shared Mutable State

**What**: Tests share state and affect each other.

**Signs**:
- Tests pass individually but fail when run together
- Test order matters
- Class-level or module-level variables mutated in tests
- Global singletons modified without reset

```python
# BAD — shared state
class TestOrderService:
    orders = []  # Shared between all tests!

    def test_add_order(self):
        self.orders.append(Order(id=1))
        assert len(self.orders) == 1

    def test_empty_orders(self):
        assert len(self.orders) == 0  # FAILS — previous test polluted state
```

**Fix**: Each test creates its own state. Use `setUp`/`beforeEach` to reset. Avoid class/module variables. Use fresh instances for every test.

---

## Conditional Logic in Tests

**What**: Tests contain `if/else`, loops, or try/catch.

**Signs**:
- `if` statements deciding what to assert
- Loops generating test data
- Try/catch hiding failures
- Complex helper logic in the test itself

```python
# BAD — conditional logic in test
def test_discounts():
    for customer in get_all_customers():
        discount = calculate_discount(customer)
        if customer.is_premium:
            assert discount > 0
        else:
            if customer.orders_count > 10:
                assert discount > 0
            else:
                assert discount == 0

# GOOD — separate, explicit tests
def test_premium_customer_gets_discount():
    customer = Customer(is_premium=True)
    assert calculate_discount(customer) > 0

def test_loyal_customer_gets_discount():
    customer = Customer(is_premium=False, orders_count=15)
    assert calculate_discount(customer) > 0

def test_new_customer_no_discount():
    customer = Customer(is_premium=False, orders_count=2)
    assert calculate_discount(customer) == 0
```

**Fix**: Tests should be linear — Arrange, Act, Assert. No branching. If you need different paths, write different tests. Parameterized tests are the right way to handle variations.

---

## Quick Reference: Detection Table

| Antipattern | Key Signal | Severity |
|-------------|-----------|----------|
| The Liar | No meaningful assert | P0 |
| The Giant | Multiple acts + many asserts | P1 |
| The Mockery | Mock setup > test logic | P1 |
| The Inspector | Testing private methods | P1 |
| The Slow Poke | sleep() or real I/O in unit test | P2 |
| Ice Cream Cone | Mostly E2E, few unit tests | P1 |
| Fragile Tests | Tests break on refactor | P1 |
| Second Class Citizen | Copy-paste, magic values, no helpers | P2 |
| 100% Coverage Trap | Tests for trivial code | P3 |
| Shared Mutable State | Tests affect each other | P0 |
| Conditional Logic | if/for in test body | P2 |
