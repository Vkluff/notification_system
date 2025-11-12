# Notification System

A robust, scalable notification system built with Django, RabbitMQ, and microservices architecture.

## Features

- **Multi-channel Notifications**: Email and push notifications
- **Microservices Architecture**: Independent services for better scalability
- **Message Queues**: Reliable message delivery with RabbitMQ
- **Circuit Breaker Pattern**: Prevents cascading failures
- **Retry Mechanism**: With exponential backoff
- **Service Discovery**: Using Consul
- **Monitoring & Metrics**: Prometheus and Grafana integration
- **API Documentation**: Swagger/OpenAPI support
- **Containerization**: Docker and Docker Compose ready

## Prerequisites

- Python 3.8+
- Docker and Docker Compose
- RabbitMQ
- Redis
- Consul (for service discovery)
- Firebase Cloud Messaging (FCM) account (for push notifications)

## Installation

1. **Clone the repository**
   ```bash
   git clone <repository-url>
   cd notification_system
   ```

2. **Set up environment variables**
   ```bash
   cp .env.example .env
   # Edit .env with your configuration
   ```

3. **Install dependencies**
   ```bash
   python -m venv venv
   source venv/bin/activate  # On Windows: venv\Scripts\activate
   pip install -r requirements.txt
   ```

4. **Run services**
   ```bash
   # Start infrastructure
   docker-compose up -d
   
   # Run migrations
   python manage.py migrate
   
   # Start the development server
   python manage.py runserver
   
   # Start Celery worker (in a new terminal)
   celery -A core worker -l info
   
   # Start Celery beat (in a new terminal)
   celery -A core beat -l info
   ```

## API Documentation

- Swagger UI: `http://localhost:8000/api/docs/`
- ReDoc: `http://localhost:8000/api/redoc/`

## Monitoring

- Prometheus: `http://localhost:9090`
- Grafana: `http://localhost:3000` (admin/admin)
- RabbitMQ Management: `http://localhost:15672` (guest/guest)
- Consul UI: `http://localhost:8500`

## Project Structure

```
notification_system/
├── core/                     # Main Django project
│   ├── api_docs.py           # API documentation setup
│   ├── celery.py             # Celery configuration
│   ├── settings/             # Django settings
│   ├── service_discovery/    # Service discovery with Consul
│   ├── monitoring/           # Monitoring and metrics
│   └── health/               # Health checks
│
├── email_service/            # Email notification service
│   ├── consumers/            # RabbitMQ consumers
│   └── templates/            # Email templates
│
├── push_service/             # Push notification service
│   └── consumers/            # Push notification consumers
│
├── template_app/             # Template management
├── user_app/                 # User management
└── api_gateway/              # API Gateway
```

## Contributing

1. Fork the repository
2. Create your feature branch (`git checkout -b feature/AmazingFeature`)
3. Commit your changes (`git commit -m 'Add some AmazingFeature'`)
4. Push to the branch (`git push origin feature/AmazingFeature`)
5. Open a Pull Request

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.
