# Notification System Implementation Guide

## Table of Contents
1. [Architecture Overview](#architecture-overview)
2. [Core Components](#core-components)
3. [Setup and Configuration](#setup-and-configuration)
4. [Sending Notifications](#sending-notifications)
5. [Monitoring and Maintenance](#monitoring-and-maintenance)
6. [Scaling the System](#scaling-the-system)
7. [Troubleshooting](#troubleshooting)

## Architecture Overview

Our notification system follows a microservices architecture with these key components:

1. **API Gateway**: Entry point for all client requests
2. **User Service**: Manages user data and preferences
3. **Email Service**: Handles email notifications
4. **Push Service**: Manages push notifications
5. **Template Service**: Stores and manages notification templates
6. **Message Queue (RabbitMQ)**: Handles asynchronous communication
7. **Service Discovery (Consul)**: Manages service registration and discovery
8. **Monitoring (Prometheus/Grafana)**: System observability

## Core Components

### 1. Message Queue (RabbitMQ)

RabbitMQ acts as the backbone of our asynchronous communication:

- **Exchanges**: `notifications.direct` for routing messages
- **Queues**: 
  - `email.queue` for email notifications
  - `push.queue` for push notifications
  - Dead-letter queues for failed messages

### 2. Circuit Breaker Pattern

Implemented to prevent cascading failures:

```python
@circuit_breaker(failure_threshold=5, recovery_timeout=60)
def process_message(self, ch, method, properties, body):
    # Message processing logic
```

### 3. Retry Mechanism

Exponential backoff for handling transient failures:

```python
@retry_with_backoff(retries=3, backoff_in_seconds=1)
def send_email(self, to_email, subject, template_name, context):
    # Email sending logic
```

### 4. Service Discovery

Consul is used for service registration and discovery:

```python
from core.service_discovery import service_discovery

# Register service
service_discovery.register_service(port=8000, tags=['api', 'v1'])

# Discover a service
service_url = service_discovery.get_service_url('user-service', '/api/v1/users/')
```

## Setup and Configuration

### Environment Variables

Create a `.env` file with these variables:

```ini
# Django
SECRET_KEY=your-secret-key
DEBUG=True
ALLOWED_HOSTS=localhost,127.0.0.1

# Database
DATABASE_URL=postgres://user:password@localhost:5432/notification_db

# RabbitMQ
RABBITMQ_HOST=localhost
RABBITMQ_PORT=5672
RABBITMQ_USERNAME=guest
RABBITMQ_PASSWORD=guest

# Redis
REDIS_HOST=localhost
REDIS_PORT=6379

# Email
EMAIL_HOST=smtp.example.com
EMAIL_PORT=587
EMAIL_USE_TLS=True
EMAIL_HOST_USER=your-email@example.com
EMAIL_HOST_PASSWORD=your-email-password

# FCM (for push notifications)
FCM_SERVER_KEY=your-fcm-server-key
```

### Database Setup

1. Apply migrations:
   ```bash
   python manage.py migrate
   ```

2. Create a superuser:
   ```bash
   python manage.py createsuperuser
   ```

## Sending Notifications

### 1. Sending an Email

```python
from django.core.mail import send_mail
from django.template.loader import render_to_string

def send_welcome_email(user):
    subject = "Welcome to Our Service"
    context = {
        'user': user,
        'welcome_message': 'Thank you for joining us!'
    }
    
    # Render email template
    html_message = render_to_string('emails/welcome.html', context)
    plain_message = strip_tags(html_message)
    
    # Send email
    send_mail(
        subject=subject,
        message=plain_message,
        from_email='noreply@example.com',
        recipient_list=[user.email],
        html_message=html_message
    )
```

### 2. Sending a Push Notification

```python
from fcm_django.models import FCMDevice

def send_push_notification(user, title, message, data=None):
    devices = FCMDevice.objects.filter(user=user, active=True)
    if devices.exists():
        devices.send_message(
            title=title,
            body=message,
            data=data or {}
        )
```

## Monitoring and Maintenance

### 1. Health Checks

Access health check endpoint:
```
GET /health/
```

### 2. Metrics

Prometheus metrics are available at:
```
/metrics
```

### 3. Logging

Logs are written to `logs/notification_system.log` with rotation.

## Scaling the System

### 1. Horizontal Scaling

1. **API Servers**:
   - Run multiple instances behind a load balancer
   - Use `gunicorn` or `uWSGI` in production

2. **Workers**:
   - Scale Celery workers based on queue length
   - Use `celery -A core worker -l info -Q email,push -c 4`

### 2. Database Scaling

1. **Read Replicas**: For read-heavy workloads
2. **Connection Pooling**: Using `pgbouncer` for PostgreSQL

## Troubleshooting

### Common Issues

1. **RabbitMQ Connection Issues**:
   - Check if RabbitMQ is running
   - Verify credentials in `.env`
   - Check network connectivity

2. **Email Not Sending**:
   - Verify SMTP settings
   - Check spam folder
   - Enable debug logging

3. **Push Notifications Failing**:
   - Verify FCM server key
   - Check device registration
   - Review FCM quotas

### Logs

Check logs for detailed error messages:
```bash
tail -f logs/notification_system.log
```

## Conclusion

This notification system provides a robust, scalable solution for handling both email and push notifications. The microservices architecture allows for independent scaling of components, while the circuit breaker and retry mechanisms ensure reliability. The system is fully containerized and ready for deployment in production environments.

For additional help, please refer to the API documentation or open an issue in the repository.
