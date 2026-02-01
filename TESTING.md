# Testing Guide

This guide covers testing strategies, frameworks, and best practices for Fleetbase.

## Table of Contents

- [Testing Philosophy](#testing-philosophy)
- [Test Types](#test-types)
- [Backend Testing](#backend-testing)
- [Frontend Testing](#frontend-testing)
- [Integration Testing](#integration-testing)
- [E2E Testing](#e2e-testing)
- [Testing Best Practices](#testing-best-practices)
- [CI/CD Testing](#cicd-testing)

## Testing Philosophy

Fleetbase follows a comprehensive testing approach:

1. **Test Pyramid**: More unit tests, fewer integration tests, minimal E2E tests
2. **Test Coverage**: Aim for 80%+ code coverage on critical paths
3. **Fast Feedback**: Tests should run quickly for rapid iteration
4. **Reliability**: Tests should be deterministic and not flaky
5. **Maintainability**: Tests should be easy to understand and maintain

## Test Types

### Unit Tests
- Test individual functions and methods in isolation
- Fast execution
- High coverage

### Integration Tests
- Test interaction between components
- Database interactions
- API endpoint testing

### End-to-End (E2E) Tests
- Test complete user workflows
- Browser automation
- Full stack testing

### Performance Tests
- Load testing
- Stress testing
- Benchmark testing

## Backend Testing

### Framework

Fleetbase backend uses **PHPUnit** for testing.

### Running Tests

```bash
cd api

# Run all tests
php artisan test

# Run specific test file
php artisan test tests/Feature/OrderTest.php

# Run specific test method
php artisan test --filter test_user_can_create_order

# Run with coverage
php artisan test --coverage

# Run in parallel (faster)
php artisan test --parallel

# Run specific test suite
php artisan test --testsuite=Feature
```

### Test Structure

```
api/tests/
├── Feature/          # Feature/integration tests
│   ├── OrderTest.php
│   ├── DriverTest.php
│   └── AuthTest.php
├── Unit/            # Unit tests
│   ├── Models/
│   ├── Services/
│   └── Helpers/
└── TestCase.php     # Base test case
```

### Writing Backend Tests

#### Feature Test Example

```php
<?php

namespace Tests\Feature;

use Tests\TestCase;
use App\Models\User;
use App\Models\Order;
use Illuminate\Foundation\Testing\RefreshDatabase;

class OrderTest extends TestCase
{
    use RefreshDatabase;

    /**
     * Test that authenticated user can create an order.
     */
    public function test_authenticated_user_can_create_order()
    {
        // Arrange
        $user = User::factory()->create();
        $orderData = [
            'customer_name' => 'John Doe',
            'customer_phone' => '+1234567890',
            'pickup_address' => '123 Main St',
            'dropoff_address' => '456 Oak Ave',
        ];

        // Act
        $response = $this->actingAs($user)
            ->postJson('/api/v1/orders', $orderData);

        // Assert
        $response->assertStatus(201)
            ->assertJsonStructure([
                'data' => ['id', 'customer_name', 'status']
            ]);

        $this->assertDatabaseHas('orders', [
            'customer_name' => 'John Doe',
            'status' => 'created'
        ]);
    }

    /**
     * Test that unauthenticated user cannot create order.
     */
    public function test_guest_cannot_create_order()
    {
        $response = $this->postJson('/api/v1/orders', [
            'customer_name' => 'John Doe'
        ]);

        $response->assertStatus(401);
    }

    /**
     * Test validation errors are returned.
     */
    public function test_create_order_validation_errors()
    {
        $user = User::factory()->create();

        $response = $this->actingAs($user)
            ->postJson('/api/v1/orders', [
                // Missing required fields
            ]);

        $response->assertStatus(422)
            ->assertJsonValidationErrors(['customer_name', 'pickup_address']);
    }
}
```

#### Unit Test Example

```php
<?php

namespace Tests\Unit\Services;

use Tests\TestCase;
use App\Services\DistanceCalculator;

class DistanceCalculatorTest extends TestCase
{
    /**
     * Test distance calculation between two coordinates.
     */
    public function test_calculates_distance_correctly()
    {
        $calculator = new DistanceCalculator();

        $distance = $calculator->calculate(
            40.7128, -74.0060,  // New York
            51.5074, -0.1278    // London
        );

        // Distance should be approximately 5570 km
        $this->assertEqualsWithDelta(5570, $distance, 10);
    }

    /**
     * Test same location returns zero distance.
     */
    public function test_same_location_returns_zero()
    {
        $calculator = new DistanceCalculator();

        $distance = $calculator->calculate(
            40.7128, -74.0060,
            40.7128, -74.0060
        );

        $this->assertEquals(0, $distance);
    }
}
```

### Database Testing

#### Using Factories

```php
<?php

namespace Database\Factories;

use Illuminate\Database\Eloquent\Factories\Factory;

class OrderFactory extends Factory
{
    public function definition()
    {
        return [
            'customer_name' => $this->faker->name(),
            'customer_phone' => $this->faker->phoneNumber(),
            'status' => 'pending',
            'pickup_address' => $this->faker->address(),
            'dropoff_address' => $this->faker->address(),
        ];
    }

    public function completed()
    {
        return $this->state(fn () => ['status' => 'completed']);
    }
}
```

**Usage:**

```php
// Create one order
$order = Order::factory()->create();

// Create multiple orders
$orders = Order::factory()->count(10)->create();

// Create with specific state
$completedOrder = Order::factory()->completed()->create();

// Create with custom attributes
$order = Order::factory()->create([
    'customer_name' => 'Specific Name'
]);
```

#### Database Assertions

```php
// Assert record exists
$this->assertDatabaseHas('orders', [
    'customer_name' => 'John Doe'
]);

// Assert record doesn't exist
$this->assertDatabaseMissing('orders', [
    'status' => 'deleted'
]);

// Assert count
$this->assertDatabaseCount('orders', 5);
```

### Mocking

```php
use Mockery;
use App\Services\NotificationService;

public function test_order_sends_notification()
{
    // Create mock
    $notificationMock = Mockery::mock(NotificationService::class);
    
    // Set expectations
    $notificationMock->shouldReceive('send')
        ->once()
        ->with('Order created', Mockery::any())
        ->andReturn(true);
    
    // Bind mock to container
    $this->app->instance(NotificationService::class, $notificationMock);
    
    // Test code that uses notification service
    // ...
}
```

## Frontend Testing

### Framework

Fleetbase frontend uses **QUnit** and **Ember Testing Helpers**.

### Running Tests

```bash
cd console

# Run all tests in browser
npm test

# Run tests in CLI (headless)
npm run test:ember

# Run specific test file
npm test -- --filter="order-list"

# Watch mode
npm run test:watch

# Run with coverage
npm test -- --coverage
```

### Test Structure

```
console/tests/
├── integration/        # Component integration tests
│   └── components/
│       ├── order-list-test.js
│       └── driver-map-test.js
├── unit/              # Unit tests
│   ├── services/
│   ├── models/
│   └── helpers/
└── acceptance/        # E2E acceptance tests
    ├── orders-test.js
    └── authentication-test.js
```

### Writing Frontend Tests

#### Component Integration Test

```javascript
import { module, test } from 'qunit';
import { setupRenderingTest } from 'ember-qunit';
import { render, click, fillIn } from '@ember/test-helpers';
import { hbs } from 'ember-cli-htmlbars';

module('Integration | Component | order-form', function(hooks) {
    setupRenderingTest(hooks);

    test('it renders order form', async function(assert) {
        await render(hbs`<OrderForm />`);

        assert.dom('[data-test-customer-name]').exists();
        assert.dom('[data-test-pickup-address]').exists();
        assert.dom('[data-test-submit]').exists();
    });

    test('it validates required fields', async function(assert) {
        await render(hbs`<OrderForm />`);

        await click('[data-test-submit]');

        assert.dom('[data-test-error]').exists();
        assert.dom('[data-test-error]').hasText('Customer name is required');
    });

    test('it submits form data', async function(assert) {
        this.set('onSubmit', (data) => {
            assert.equal(data.customerName, 'John Doe');
            assert.equal(data.pickupAddress, '123 Main St');
        });

        await render(hbs`<OrderForm @onSubmit={{this.onSubmit}} />`);

        await fillIn('[data-test-customer-name]', 'John Doe');
        await fillIn('[data-test-pickup-address]', '123 Main St');
        await click('[data-test-submit]');
    });
});
```

#### Acceptance Test

```javascript
import { module, test } from 'qunit';
import { setupApplicationTest } from 'ember-qunit';
import { visit, click, fillIn, currentURL } from '@ember/test-helpers';
import { setupMirage } from 'ember-cli-mirage/test-support';

module('Acceptance | orders', function(hooks) {
    setupApplicationTest(hooks);
    setupMirage(hooks);

    test('user can create a new order', async function(assert) {
        // Visit orders page
        await visit('/orders');

        // Click create button
        await click('[data-test-create-order]');

        // Verify we're on the new order page
        assert.equal(currentURL(), '/orders/new');

        // Fill in form
        await fillIn('[data-test-customer-name]', 'John Doe');
        await fillIn('[data-test-customer-phone]', '+1234567890');
        await fillIn('[data-test-pickup-address]', '123 Main St');
        await fillIn('[data-test-dropoff-address]', '456 Oak Ave');

        // Submit form
        await click('[data-test-submit]');

        // Verify redirect and success message
        assert.equal(currentURL(), '/orders');
        assert.dom('[data-test-notification]')
            .hasText('Order created successfully');

        // Verify order appears in list
        assert.dom('[data-test-order]').exists({ count: 1 });
        assert.dom('[data-test-order-customer]').hasText('John Doe');
    });

    test('user can view order details', async function(assert) {
        // Create test data via Mirage
        const order = this.server.create('order', {
            customerName: 'Jane Smith',
            status: 'pending'
        });

        await visit(`/orders/${order.id}`);

        assert.dom('[data-test-customer-name]').hasText('Jane Smith');
        assert.dom('[data-test-status]').hasText('Pending');
    });
});
```

#### Service Unit Test

```javascript
import { module, test } from 'qunit';
import { setupTest } from 'ember-qunit';

module('Unit | Service | distance-calculator', function(hooks) {
    setupTest(hooks);

    test('it calculates distance correctly', function(assert) {
        const service = this.owner.lookup('service:distance-calculator');

        const distance = service.calculate(
            { lat: 40.7128, lng: -74.0060 },  // New York
            { lat: 51.5074, lng: -0.1278 }    // London
        );

        assert.ok(distance > 5560 && distance < 5580, 
            'Distance should be approximately 5570 km');
    });
});
```

### Mocking API Requests (Mirage)

```javascript
// mirage/config.js
export default function() {
    this.namespace = '/api/v1';

    // GET /api/v1/orders
    this.get('/orders', (schema) => {
        return schema.orders.all();
    });

    // POST /api/v1/orders
    this.post('/orders', (schema, request) => {
        const attrs = JSON.parse(request.requestBody);
        return schema.orders.create(attrs);
    });

    // GET /api/v1/orders/:id
    this.get('/orders/:id', (schema, request) => {
        return schema.orders.find(request.params.id);
    });
}
```

## Integration Testing

### API Integration Tests

Test complete API workflows:

```php
public function test_complete_order_workflow()
{
    $user = User::factory()->create();

    // Create order
    $createResponse = $this->actingAs($user)
        ->postJson('/api/v1/orders', [/* ... */]);
    
    $orderId = $createResponse->json('data.id');

    // Assign driver
    $driver = Driver::factory()->create();
    $assignResponse = $this->actingAs($user)
        ->postJson("/api/v1/orders/{$orderId}/assign", [
            'driver_id' => $driver->id
        ]);
    
    $assignResponse->assertStatus(200);

    // Update status
    $statusResponse = $this->actingAs($user)
        ->patchJson("/api/v1/orders/{$orderId}", [
            'status' => 'in_progress'
        ]);
    
    $statusResponse->assertStatus(200);

    // Verify final state
    $this->assertDatabaseHas('orders', [
        'id' => $orderId,
        'status' => 'in_progress',
        'driver_id' => $driver->id
    ]);
}
```

## E2E Testing

### Tools

- **Selenium**: Browser automation
- **Cypress**: Alternative E2E framework (optional)

### Example E2E Test

```javascript
// Using Ember test helpers
test('complete order creation flow', async function(assert) {
    // Login
    await visit('/login');
    await fillIn('[data-test-email]', 'user@example.com');
    await fillIn('[data-test-password]', 'password');
    await click('[data-test-login]');

    // Navigate to orders
    await click('[data-test-nav-orders]');
    assert.equal(currentURL(), '/orders');

    // Create order
    await click('[data-test-create-order]');
    // ... fill form ...
    await click('[data-test-submit]');

    // Verify creation
    assert.dom('[data-test-notification]').containsText('created');
});
```

## Testing Best Practices

### General Principles

1. **AAA Pattern**: Arrange, Act, Assert
2. **One Assertion Per Test**: Focus each test
3. **Descriptive Names**: Test names should describe what they test
4. **Independent Tests**: Tests shouldn't depend on each other
5. **Fast Tests**: Keep tests quick for rapid feedback

### Naming Conventions

```php
// Good
test_authenticated_user_can_create_order()
test_guest_cannot_access_admin_panel()
test_order_validation_fails_with_invalid_data()

// Bad
test_order()
test_1()
test_feature()
```

### Data Builders

```php
class OrderBuilder
{
    private $attributes = [];

    public function withCustomer(string $name, string $phone)
    {
        $this->attributes['customer_name'] = $name;
        $this->attributes['customer_phone'] = $phone;
        return $this;
    }

    public function pending()
    {
        $this->attributes['status'] = 'pending';
        return $this;
    }

    public function build()
    {
        return Order::factory()->create($this->attributes);
    }
}

// Usage
$order = (new OrderBuilder())
    ->withCustomer('John Doe', '+1234567890')
    ->pending()
    ->build();
```

### Test Data Management

```php
// Use setUp for common data
protected function setUp(): void
{
    parent::setUp();
    
    $this->user = User::factory()->create();
    $this->actingAs($this->user);
}

// Clean up in tearDown if needed
protected function tearDown(): void
{
    // Cleanup
    parent::tearDown();
}
```

## CI/CD Testing

### GitHub Actions

```yaml
name: Tests

on: [push, pull_request]

jobs:
  backend-tests:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      
      - name: Setup PHP
        uses: shivammathur/setup-php@v2
        with:
          php-version: '8.1'
      
      - name: Install Dependencies
        run: cd api && composer install
      
      - name: Run Tests
        run: cd api && php artisan test --coverage

  frontend-tests:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      
      - name: Setup Node
        uses: actions/setup-node@v2
        with:
          node-version: '18'
      
      - name: Install Dependencies
        run: cd console && npm install
      
      - name: Run Tests
        run: cd console && npm test
```

### Running Tests Locally

```bash
# Backend
cd api
composer install
php artisan test

# Frontend
cd console
npm install
npm test

# Docker
docker-compose exec application php artisan test
docker-compose exec console npm test
```

## Coverage Reports

### Backend Coverage

```bash
cd api
php artisan test --coverage --min=80
```

### Frontend Coverage

```bash
cd console
npm test -- --coverage
```

### Viewing Coverage

Coverage reports are generated in:
- Backend: `api/coverage/`
- Frontend: `console/coverage/`

Open `index.html` in your browser to view detailed reports.

## Troubleshooting

### Common Issues

**Tests fail locally but pass in CI:**
- Check environment differences
- Verify dependencies versions
- Check database state

**Flaky tests:**
- Add wait conditions
- Use proper test isolation
- Check for race conditions

**Slow tests:**
- Use database transactions
- Mock external services
- Parallelize test execution

## Resources

- [PHPUnit Documentation](https://phpunit.de/documentation.html)
- [Laravel Testing](https://laravel.com/docs/testing)
- [Ember Testing Guide](https://guides.emberjs.com/release/testing/)
- [QUnit Documentation](https://qunitjs.com/)
