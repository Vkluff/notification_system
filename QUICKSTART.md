# Quick Start Guide

Get your notification system up and running in minutes!

## Prerequisites

- Docker and Docker Compose
- Python 3.8+

## 1. Clone the Repository

```bash
git clone <repository-url>
cd notification_system
```

## 2. Set Up Environment

1. Copy the example environment file:
   ```bash
   cp .env.example .env
   ```

2. Update the `.env` file with your configuration.

## 3. Start Services

```bash
# Start all services
docker-compose up -d

# Or start specific services
docker-compose up -d rabbitmq redis consul
```

## 4. Set Up the Application

1. Create a virtual environment:
   ```bash
   python -m venv venv
   source venv/bin/activate  # On Windows: venv\Scripts\activate
   ```

2. Install dependencies:
   ```bash
   pip install -r requirements.txt
   ```

3. Run migrations:
   ```bash
   python manage.py migrate
   ```

4. Create a superuser:
   ```bash
   python manage.py createsuperuser
   ```

## 5. Start the Development Server

```bash
python manage.py runserver
```

## 6. Start Workers

In separate terminals:

```bash
# Start Celery worker for processing tasks
celery -A core worker -l info

# Start Celery beat for scheduled tasks
celery -A core beat -l info

# Start email consumer
python manage.py run_email_consumer

# Start push notification consumer
python manage.py run_push_consumer
```

## 7. Access the System

- **Django Admin**: http://localhost:8000/admin
- **API Documentation**: http://localhost:8000/api/docs/
- **RabbitMQ Management**: http://localhost:15672 (guest/guest)
- **Consul UI**: http://localhost:8500
- **Prometheus**: http://localhost:9090
- **Grafana**: http://localhost:3000 (admin/admin)

## 8. Send Your First Notification

### Send an Email

```python
from django.core.mail import send_mail

send_mail(
    'Test Subject',
    'This is a test email.',
    'from@example.com',
    ['to@example.com'],
    fail_silently=False,
)
```

### Send a Push Notification

```python
from fcm_django.models import FCMDevice

# Get user's device
device = FCMDevice.objects.get(registration_id="device_registration_id")

# Send message
device.send_message(
    title="Test Notification",
    body="This is a test push notification",
    data={"key": "value"}
)
```

## Next Steps

1. Explore the [API Documentation](http://localhost:8000/api/docs/)
2. Check out the [Tutorial](TUTORIAL.md) for detailed implementation details
3. Configure monitoring in [Grafana](http://localhost:3000)
4. Set up production deployment

## Need Help?

- Check the [Troubleshooting Guide](TUTORIAL.md#troubleshooting)
- Open an issue in the repository
