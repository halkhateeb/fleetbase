# Quick Start Guide

Get Fleetbase up and running in 5 minutes! This guide will help you install and start using Fleetbase quickly.

## Prerequisites

Before you begin, ensure you have:

- **Docker Desktop** installed ([Download here](https://www.docker.com/products/docker-desktop))
- **Git** installed ([Download here](https://git-scm.com/downloads))
- At least **4GB of free RAM**
- **20GB of free disk space**

## Installation (3 Steps)

### Step 1: Clone the Repository

```bash
git clone https://github.com/fleetbase/fleetbase.git
cd fleetbase
```

### Step 2: Run the Installation Script

```bash
./scripts/docker-install.sh
```

This script will:
- Pull all necessary Docker images
- Set up the database
- Configure the environment
- Start all services

⏱️ **Installation takes 5-10 minutes** depending on your internet speed.

### Step 3: Access Fleetbase

Once installation is complete, you can access:

- **📱 Console (UI)**: http://localhost:4200
- **🔌 API**: http://localhost:8000
- **🔄 WebSocket**: http://localhost:8001

## First Login

### Default Credentials

```
Email: admin@fleetbase.io
Password: (set during installation)
```

If you didn't set a password during installation, check the installation output or reset it:

```bash
docker-compose exec application php artisan fleetbase:create-user
```

## Quick Tour

### 1. Dashboard

After logging in, you'll see the main dashboard with:
- Quick stats overview
- Recent activity
- Quick actions

### 2. Create Your First Order

1. Click **Orders** in the sidebar
2. Click **New Order** button
3. Fill in:
   - Customer name and contact
   - Pickup location
   - Dropoff location
   - Items/cargo details
4. Click **Create Order**

### 3. Add a Driver

1. Navigate to **Fleet** → **Drivers**
2. Click **New Driver**
3. Enter driver details:
   - Name
   - Email
   - Phone number
   - Vehicle information
4. Click **Save**

### 4. Assign Order to Driver

1. Go back to **Orders**
2. Click on your created order
3. Click **Assign Driver**
4. Select the driver you created
5. The order status will update to "Assigned"

### 5. Track on Map

1. Click **Live Map** in the sidebar
2. View all active orders and driver locations in real-time
3. Click on markers for details

## Essential Features

### 🎯 Order Management

```
Orders → New Order
```

Create and manage delivery orders with full workflow support:
- Multiple waypoints
- Proof of delivery
- Real-time tracking
- Status updates

### 🚗 Fleet Management

```
Fleet → Drivers | Vehicles
```

Manage your fleet:
- Add drivers and vehicles
- Track locations
- View availability
- Performance metrics

### 📍 Places

```
Operations → Places
```

Save frequently used locations:
- Warehouses
- Customer locations
- Service areas
- Zones

### 📊 Analytics

```
Dashboard → Reports
```

View insights:
- Order statistics
- Driver performance
- Revenue metrics
- Custom reports

## Configuration

### Environment Variables

Key settings to configure in your `.env` file:

```bash
# Application
APP_NAME=Fleetbase
APP_URL=http://localhost:4200

# Mail
MAIL_MAILER=smtp
MAIL_HOST=your-smtp-host
MAIL_PORT=587
MAIL_USERNAME=your-username
MAIL_PASSWORD=your-password

# Services
GOOGLE_MAPS_API_KEY=your-google-maps-key
TWILIO_SID=your-twilio-sid (for SMS)
IPINFO_API_KEY=your-ipinfo-key
```

### Getting API Keys

1. **Google Maps**: https://console.cloud.google.com
2. **Twilio**: https://www.twilio.com/console
3. **IPInfo**: https://ipinfo.io/signup

## Common Commands

### Managing Services

```bash
# Start services
docker-compose up -d

# Stop services
docker-compose down

# View logs
docker-compose logs -f

# Restart specific service
docker-compose restart application

# Access application shell
docker-compose exec application bash
```

### Database Operations

```bash
# Run migrations
docker-compose exec application php artisan migrate

# Seed database with sample data
docker-compose exec application php artisan db:seed

# Reset database
docker-compose exec application php artisan migrate:fresh --seed
```

### Clear Cache

```bash
# Clear all caches
docker-compose exec application php artisan cache:clear
docker-compose exec application php artisan config:clear
docker-compose exec application php artisan route:clear
```

## Troubleshooting

### Services Won't Start

```bash
# Check service status
docker-compose ps

# View error logs
docker-compose logs application
docker-compose logs mysql
```

### Port Conflicts

If ports 4200, 8000, or 8001 are already in use:

1. Edit `docker-compose.yml`
2. Change port mappings:
   ```yaml
   ports:
     - "4201:4200"  # Change 4200 to 4201
   ```
3. Restart: `docker-compose up -d`

### Can't Login

```bash
# Reset admin password
docker-compose exec application php artisan fleetbase:create-user

# Or reset existing user
docker-compose exec application php artisan tinker
>>> $user = User::where('email', 'admin@fleetbase.io')->first();
>>> $user->password = bcrypt('new-password');
>>> $user->save();
```

### Database Connection Error

```bash
# Verify MySQL is running
docker-compose ps mysql

# Restart MySQL
docker-compose restart mysql

# Check database credentials in .env
docker-compose exec application cat .env | grep DB_
```

## What's Next?

### 📚 Learn More

- **[Full Documentation](https://docs.fleetbase.io/)** - Complete guides
- **[ARCHITECTURE.md](ARCHITECTURE.md)** - System architecture
- **[DEVELOPMENT.md](DEVELOPMENT.md)** - Development guide
- **[API_REFERENCE.md](API_REFERENCE.md)** - API documentation

### 🔌 Extensions

Install additional features:

```bash
# Install Fleetbase CLI
npm install -g @fleetbase/cli

# Login to registry
fleetbase auth login

# Install extensions
fleetbase install fleetops
fleetbase install pallet
fleetbase install ledger
```

### 🛠️ Customize

- **Theme**: Settings → Appearance
- **Branding**: Add your logo and colors
- **Workflows**: Create custom order workflows
- **Integrations**: Connect external services via webhooks

### 🌍 Internationalization

Change language:

1. Click your avatar (top right)
2. Select **Language**
3. Choose from 12+ supported languages
4. Interface updates immediately

### 📱 Mobile Apps

Deploy mobile apps for drivers and customers:

- **[Navigator App](https://github.com/fleetbase/navigator-app)** - Driver app
- **[Storefront App](https://github.com/fleetbase/storefront-app)** - Customer app

### 🚀 Deploy to Production

Ready for production? Deploy to AWS:

[![Deploy to AWS](https://img.shields.io/badge/Deploy%20to%20AWS-FF9900?style=for-the-badge&logo=amazon-aws&logoColor=white)](https://console.fleetbase.io/aws-marketplace)

Or see **[DEPLOYMENT.md](DEPLOYMENT.md)** for other deployment options.

## Getting Help

### Community Support

- **💬 [Discord](https://discord.gg/V7RVWRQ2Wm)** - Chat with community
- **💭 [GitHub Discussions](https://github.com/orgs/fleetbase/discussions)** - Ask questions
- **🐛 [GitHub Issues](https://github.com/fleetbase/fleetbase/issues)** - Report bugs

### Documentation

- **📖 [Official Docs](https://docs.fleetbase.io/)** - Comprehensive guides
- **🔧 [API Docs](https://docs.fleetbase.io/api)** - API reference
- **📺 [Video Tutorials](https://www.youtube.com/@fleetbase)** - Video guides

### Professional Support

- **📧 Email**: support@fleetbase.io
- **🗓️ [Book a Demo](https://tally.so/r/3NBpAW)** - Guided walkthrough
- **💼 Enterprise**: enterprise@fleetbase.io

## Contributing

Want to contribute? Check out:

- **[CONTRIBUTING.md](CONTRIBUTING.md)** - Contribution guidelines
- **[TRANSLATING.md](TRANSLATING.md)** - Add language support
- **[Good First Issues](https://github.com/fleetbase/fleetbase/labels/good%20first%20issue)** - Start here

## License

Fleetbase is open source software licensed under [AGPL-3.0](LICENSE).

---

**🎉 Congratulations!** You're now ready to use Fleetbase. Happy shipping! 📦
