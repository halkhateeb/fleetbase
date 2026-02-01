# Development Guide

This guide will help you set up your local development environment for contributing to Fleetbase.

## Table of Contents

- [Prerequisites](#prerequisites)
- [Local Setup](#local-setup)
- [Development Workflow](#development-workflow)
- [Code Standards](#code-standards)
- [Testing](#testing)
- [Debugging](#debugging)
- [Common Tasks](#common-tasks)
- [Troubleshooting](#troubleshooting)

## Prerequisites

### Required Software

- **Docker** (latest version) - [Install Docker](https://docs.docker.com/get-docker/)
- **Docker Compose** (latest version) - Usually included with Docker Desktop
- **Git** - [Install Git](https://git-scm.com/downloads)
- **Node.js** (v18+) - [Install Node.js](https://nodejs.org/)
- **PHP** (8.1+) - For local development without Docker (optional)
- **Composer** - For PHP dependency management (optional)

### Recommended Tools

- **Visual Studio Code** - Recommended IDE with excellent extension support
- **Postman** or **Insomnia** - For API testing
- **TablePlus** or **DBeaver** - Database GUI tools
- **Git GUI** - GitKraken, SourceTree, or GitHub Desktop

### VS Code Extensions

```
- PHP Intelephense
- Laravel Extra Intellisense
- Ember Language Server
- Tailwind CSS IntelliSense
- Docker
- GitLens
- ESLint
- Prettier
```

## Local Setup

### Quick Start (Docker)

The fastest way to get started is using Docker:

```bash
# Clone the repository
git clone git@github.com:fleetbase/fleetbase.git
cd fleetbase

# Run the installation script
./scripts/docker-install.sh

# Wait for services to start (5-10 minutes)
# Access the console at http://localhost:4200
# API will be available at http://localhost:8000
```

### Manual Setup (Without Docker)

If you prefer to run services directly on your machine:

#### 1. Backend Setup

```bash
cd api

# Install PHP dependencies
composer install

# Copy environment file
cp .env.example .env

# Generate application key
php artisan key:generate

# Configure your database in .env
# DB_CONNECTION=mysql
# DB_HOST=127.0.0.1
# DB_PORT=3306
# DB_DATABASE=fleetbase
# DB_USERNAME=root
# DB_PASSWORD=

# Run migrations
php artisan migrate

# Seed the database (optional)
php artisan db:seed

# Start the development server
php artisan serve --port=8000
```

#### 2. Frontend Setup

```bash
cd console

# Install Node.js dependencies
npm install

# Copy environment file
cp .env.example .env.development

# Configure API host in .env.development
# API_HOST=http://localhost:8000

# Start the development server
npm start
# or
ember serve

# Console will be available at http://localhost:4200
```

#### 3. WebSocket Server Setup

```bash
cd socket

# Install dependencies
npm install

# Copy environment file
cp .env.example .env

# Configure the socket server
# PORT=8001
# CONSOLE_HOST=http://localhost:4200

# Start the socket server
npm start
```

## Development Workflow

### Creating a New Feature

1. **Create a feature branch**
   ```bash
   git checkout -b feature/your-feature-name
   ```

2. **Make your changes**
   - Follow code standards (see below)
   - Write tests for new functionality
   - Update documentation as needed

3. **Test your changes**
   ```bash
   # Backend tests
   cd api && php artisan test
   
   # Frontend tests
   cd console && npm test
   ```

4. **Commit your changes**
   ```bash
   git add .
   git commit -m "feat: add your feature description"
   ```

5. **Push and create a pull request**
   ```bash
   git push origin feature/your-feature-name
   ```

### Commit Message Convention

We follow [Conventional Commits](https://www.conventionalcommits.org/):

```
<type>(<scope>): <subject>

<body>

<footer>
```

**Types:**
- `feat`: New feature
- `fix`: Bug fix
- `docs`: Documentation changes
- `style`: Code style changes (formatting, etc.)
- `refactor`: Code refactoring
- `test`: Test additions or changes
- `chore`: Build process or auxiliary tool changes

**Examples:**
```
feat(orders): add bulk order import functionality
fix(auth): resolve token refresh issue
docs(api): update API endpoint documentation
```

## Code Standards

### Backend (PHP/Laravel)

#### Coding Style

Follow [PSR-12](https://www.php-fig.org/psr/psr-12/) coding standards:

```php
<?php

namespace App\Services;

use App\Models\Order;
use Illuminate\Support\Facades\DB;

class OrderService
{
    /**
     * Create a new order.
     *
     * @param array $data Order data
     * @return Order Created order instance
     */
    public function createOrder(array $data): Order
    {
        return DB::transaction(function () use ($data) {
            $order = Order::create($data);
            
            // Additional logic here
            
            return $order;
        });
    }
}
```

#### Best Practices

- Use type hints for parameters and return types
- Add PHPDoc comments for methods
- Follow single responsibility principle
- Use dependency injection
- Validate input data
- Handle exceptions appropriately
- Write tests for business logic

#### Running PHP Code Style Fixer

```bash
cd api

# Check code style
./vendor/bin/php-cs-fixer fix --dry-run --diff

# Fix code style issues
./vendor/bin/php-cs-fixer fix
```

### Frontend (Ember.js)

#### Coding Style

Follow [Ember.js Style Guide](https://github.com/ember-best-practices/ember-best-practices):

```javascript
import Component from '@glimmer/component';
import { tracked } from '@glimmer/tracking';
import { action } from '@ember/object';
import { inject as service } from '@ember/service';

export default class OrderListComponent extends Component {
    @service store;
    @service notifications;
    
    @tracked selectedOrders = [];
    
    /**
     * Load orders from the API.
     */
    @action
    async loadOrders() {
        try {
            const orders = await this.store.findAll('order');
            return orders;
        } catch (error) {
            this.notifications.error('Failed to load orders');
        }
    }
}
```

#### Best Practices

- Use Ember Octane patterns (tracked properties, @action decorators)
- Prefer Glimmer components over classic components
- Use services for shared state and functionality
- Follow data down, actions up pattern
- Use ember-concurrency for async operations
- Write acceptance and integration tests
- Use Tailwind CSS utility classes consistently

#### Running Linters

```bash
cd console

# Run ESLint
npm run lint:js

# Fix ESLint issues
npm run lint:js:fix

# Run template linter
npm run lint:hbs

# Fix template issues
npm run lint:hbs:fix
```

## Testing

### Backend Testing

#### Running Tests

```bash
cd api

# Run all tests
php artisan test

# Run specific test file
php artisan test tests/Feature/OrderTest.php

# Run tests with coverage
php artisan test --coverage

# Run tests in parallel
php artisan test --parallel
```

#### Writing Tests

```php
<?php

namespace Tests\Feature;

use Tests\TestCase;
use App\Models\User;
use App\Models\Order;

class OrderTest extends TestCase
{
    public function test_user_can_create_order()
    {
        $user = User::factory()->create();
        
        $response = $this->actingAs($user)
            ->postJson('/api/v1/orders', [
                'customer_name' => 'John Doe',
                'items' => [
                    ['product' => 'Item 1', 'quantity' => 2]
                ]
            ]);
        
        $response->assertStatus(201)
            ->assertJsonStructure(['order' => ['id', 'customer_name']]);
        
        $this->assertDatabaseHas('orders', [
            'customer_name' => 'John Doe'
        ]);
    }
}
```

### Frontend Testing

#### Running Tests

```bash
cd console

# Run all tests
npm test

# Run tests in watch mode
npm run test:watch

# Run specific test file
npm test tests/integration/components/order-list-test.js

# Run tests with coverage
npm test -- --coverage
```

#### Writing Tests

**Component Integration Test:**

```javascript
import { module, test } from 'qunit';
import { setupRenderingTest } from 'ember-qunit';
import { render, click } from '@ember/test-helpers';
import { hbs } from 'ember-cli-htmlbars';

module('Integration | Component | order-list', function(hooks) {
    setupRenderingTest(hooks);
    
    test('it displays orders', async function(assert) {
        this.set('orders', [
            { id: 1, customerName: 'John Doe' },
            { id: 2, customerName: 'Jane Smith' }
        ]);
        
        await render(hbs`<OrderList @orders={{this.orders}} />`);
        
        assert.dom('[data-test-order]').exists({ count: 2 });
    });
});
```

**Acceptance Test:**

```javascript
import { module, test } from 'qunit';
import { setupApplicationTest } from 'ember-qunit';
import { visit, click, fillIn } from '@ember/test-helpers';

module('Acceptance | orders', function(hooks) {
    setupApplicationTest(hooks);
    
    test('user can create an order', async function(assert) {
        await visit('/orders/new');
        
        await fillIn('[data-test-customer-name]', 'John Doe');
        await click('[data-test-submit]');
        
        assert.equal(currentURL(), '/orders');
        assert.dom('[data-test-notification]').hasText('Order created successfully');
    });
});
```

## Debugging

### Backend Debugging

#### Using dd() and dump()

```php
// Dump and die
dd($variable);

// Dump without stopping
dump($variable);

// Ray debugging (if installed)
ray($variable);
```

#### Query Debugging

```php
// Enable query logging
DB::enableQueryLog();

// Your queries here
$orders = Order::where('status', 'pending')->get();

// Get executed queries
dd(DB::getQueryLog());

// Or use query builder debugging
$query = Order::where('status', 'pending');
dd($query->toSql(), $query->getBindings());
```

#### Laravel Telescope

```bash
# Install Telescope (if not already installed)
composer require laravel/telescope --dev
php artisan telescope:install
php artisan migrate

# Access at http://localhost:8000/telescope
```

### Frontend Debugging

#### Ember Inspector

Install the [Ember Inspector](https://guides.emberjs.com/release/ember-inspector/) browser extension:
- Chrome: [Chrome Web Store](https://chrome.google.com/webstore/detail/ember-inspector/bmdblncegkenkacieihfhpjfppoconhi)
- Firefox: [Firefox Add-ons](https://addons.mozilla.org/en-US/firefox/addon/ember-inspector/)

#### Console Debugging

```javascript
// In components or routes
import { debug } from '@ember/debug';

debug('Current value:', this.selectedOrders);

// Breakpoint debugging
debugger; // Browser will pause here

// Service inspection
console.log(this.store.peekAll('order'));
```

#### Network Debugging

Use browser DevTools Network tab to inspect API requests:
1. Open DevTools (F12)
2. Go to Network tab
3. Filter by XHR/Fetch
4. Inspect request/response details

## Common Tasks

### Adding a New API Endpoint

1. **Create a route:**
   ```php
   // api/routes/api.php
   Route::get('/orders/{id}/status', [OrderController::class, 'getStatus']);
   ```

2. **Create/update controller:**
   ```php
   // api/app/Http/Controllers/OrderController.php
   public function getStatus(string $id)
   {
       $order = Order::findOrFail($id);
       return response()->json(['status' => $order->status]);
   }
   ```

3. **Add tests:**
   ```php
   // api/tests/Feature/OrderTest.php
   public function test_can_get_order_status()
   {
       $order = Order::factory()->create(['status' => 'pending']);
       
       $response = $this->getJson("/api/v1/orders/{$order->id}/status");
       
       $response->assertOk()
           ->assertJson(['status' => 'pending']);
   }
   ```

### Adding a New Frontend Route

1. **Generate route:**
   ```bash
   cd console
   ember generate route orders/details
   ```

2. **Update router:**
   ```javascript
   // console/app/router.js
   Router.map(function() {
       this.route('orders', function() {
           this.route('details', { path: '/:order_id/details' });
       });
   });
   ```

3. **Implement route:**
   ```javascript
   // console/app/routes/orders/details.js
   import Route from '@ember/routing/route';
   import { inject as service } from '@ember/service';
   
   export default class OrdersDetailsRoute extends Route {
       @service store;
       
       model(params) {
           return this.store.findRecord('order', params.order_id);
       }
   }
   ```

### Adding Translations

1. **Add to English base:**
   ```yaml
   # console/translations/en-us.yaml
   orders:
     title: Orders
     create: Create Order
     status:
       pending: Pending
       completed: Completed
   ```

2. **Add to other languages:**
   ```yaml
   # console/translations/ar-ae.yaml
   orders:
     title: الطلبات
     create: إنشاء طلب
     status:
       pending: قيد الانتظار
       completed: مكتمل
   ```

3. **Use in templates:**
   ```handlebars
   <h1>{{t "orders.title"}}</h1>
   <button>{{t "orders.create"}}</button>
   ```

### Database Migrations

```bash
cd api

# Create a new migration
php artisan make:migration create_orders_table

# Run migrations
php artisan migrate

# Rollback last migration
php artisan migrate:rollback

# Reset database
php artisan migrate:fresh

# Seed database
php artisan db:seed
```

## Troubleshooting

### Docker Issues

**Issue: Containers won't start**
```bash
# Check logs
docker-compose logs

# Restart containers
docker-compose restart

# Rebuild containers
docker-compose up --build
```

**Issue: Port conflicts**
```bash
# Check what's using a port (e.g., 4200)
lsof -i :4200

# Kill the process
kill -9 <PID>
```

### API Issues

**Issue: 500 Internal Server Error**
- Check `api/storage/logs/laravel.log`
- Verify environment variables in `.env`
- Ensure database connection is working
- Clear cache: `php artisan cache:clear`

**Issue: Database connection refused**
- Verify database is running
- Check database credentials in `.env`
- Ensure MySQL port is correct (default: 3306)

### Frontend Issues

**Issue: Build failures**
```bash
# Clear npm cache
npm cache clean --force

# Remove node_modules and reinstall
rm -rf node_modules package-lock.json
npm install
```

**Issue: Module not found**
```bash
# Ensure all dependencies are installed
npm install

# Check for missing peer dependencies
npm ls
```

### Performance Issues

**Issue: Slow API responses**
- Enable query logging to identify slow queries
- Add database indexes
- Use Redis caching
- Profile with Laravel Telescope

**Issue: Slow frontend**
- Use Ember Inspector to profile component rendering
- Check for unnecessary data fetching
- Optimize images and assets
- Enable production build: `ember build --environment=production`

## Getting Help

- **Documentation**: [https://docs.fleetbase.io](https://docs.fleetbase.io)
- **GitHub Issues**: [https://github.com/fleetbase/fleetbase/issues](https://github.com/fleetbase/fleetbase/issues)
- **Discord Community**: [Join Discord](https://discord.gg/V7RVWRQ2Wm)
- **GitHub Discussions**: [https://github.com/orgs/fleetbase/discussions](https://github.com/orgs/fleetbase/discussions)

## Next Steps

- Read [CONTRIBUTING.md](CONTRIBUTING.md) for contribution guidelines
- Check out [ARCHITECTURE.md](ARCHITECTURE.md) to understand the system design
- Review [TRANSLATING.md](TRANSLATING.md) for adding new language support
- Explore the [Extension Development Guide](https://docs.fleetbase.io/developers/building-an-extension)
