# Instruction on how to setup my monitoring app tool



# Devnetproject — Operations Guide

A Django-based network monitoring and alerting platform with built-in monitoring, ticketing, notifications, and network management.

---

## Project Structure

```
Devnetproject-main/
├── myproject/          # Main Django project settings & URLs
├── core/               # Core app
├── accounts/           # User accounts
├── login/              # Authentication
├── homepage/           # Dashboard/home
├── monitor/            # Custom monitoring app (Grafana/Prometheus-like)
├── network/            # Network management
├── network_api/        # Network API endpoints
├── notifications/      # Notifications system
├── tickets/            # Ticketing system
├── department/         # Department management
├── knowledge/          # Knowledge base
├── manage.py           # Django management tool
├── requirements.txt    # Python dependencies
└── db.sqlite3          # SQLite database
```

---

## Requirements

- Python 3.8+
- pip
- virtualenv

---

## Installation

### 1. Clone the project
```bash
git clone <your-repo-url>
cd Devnetproject-main
```

### 2. Create and activate virtual environment
```bash
# Create virtual environment
python -m venv venv

# Activate — Linux/Mac
source venv/bin/activate

# Activate — Windows
venv\Scripts\activate
```

### 3. Install dependencies
```bash
pip install -r requirements.txt
```

### 4. Set up environment variables

Create a `.env` file in the project root:
```env
SECRET_KEY=your-secret-key-here
DEBUG=True
ALERTMANAGER_URL=http://localhost:9093
LOKI_URL=http://localhost:3100
```

### 5. Apply database migrations
```bash
python manage.py migrate
```

### 6. Create a superuser (admin account)
```bash
python manage.py createsuperuser
```

---

## Starting the Application

```bash
python manage.py runserver
```

Then open your browser and go to:

```
http://127.0.0.1:8000
```

---

## Accessing the Apps

| Module | URL | Description |
|---|---|---|
| Home Dashboard | http://127.0.0.1:8000/ | Main dashboard |
| Admin Panel | http://127.0.0.1:8000/admin/ | Django admin |
| Monitor | http://127.0.0.1:8000/monitor/ | Custom monitoring dashboard |
| Network | http://127.0.0.1:8000/network/ | Network management |
| Tickets | http://127.0.0.1:8000/tickets/ | Ticketing system |
| Notifications | http://127.0.0.1:8000/notifications/ | Alerts & notifications |
| Knowledge Base | http://127.0.0.1:8000/knowledge/ | Knowledge base |
| API - Alerts | http://127.0.0.1:8000/api/alerts/ | Alertmanager API endpoint |

---

## Alert System

The alerts API connects to Alertmanager to fetch active alerts.

### API Endpoint
```
GET /api/alerts/
```

### Example Response
```json
{
  "alerts": [
    {
      "name": "HighCPUUsage",
      "severity": "warning",
      "state": "active",
      "summary": "CPU usage above 80%",
      "description": "CPU has been above 80% for 5 minutes",
      "labels": {}
    }
  ]
}
```

### Test the alerts endpoint
```bash
curl http://127.0.0.1:8000/api/alerts/
```

---

## External Services (Optional)

The app integrates with these services if available:

| Service | Default Port | Purpose |
|---|---|---|
| Alertmanager | 9093 | Alert management |
| Loki | 3100 | Log aggregation |

To verify they are running:
```bash
# Check Alertmanager
curl http://localhost:9093/api/v2/alerts

# Check Loki
curl http://localhost:3100/ready
```

> The app will still run without these services. Alert API calls will return an empty list with an error message if they are unreachable.

---

## Daily Operations

### Start the server
```bash
source venv/bin/activate
python manage.py runserver
```

### Stop the server
Press `Ctrl + C` in the terminal.

### Run on a different port
```bash
python manage.py runserver 8080
```

### Collect static files (for production)
```bash
python manage.py collectstatic
```

### Create a new admin user
```bash
python manage.py createsuperuser
```

---

## Troubleshooting

| Problem | Solution |
|---|---|
| `ModuleNotFoundError` | Run `pip install -r requirements.txt` |
| `Migration errors` | Run `python manage.py migrate` |
| `Port already in use` | Run on different port: `python manage.py runserver 8080` |
| Alerts return empty | Check Alertmanager is running on port 9093 |
| Static files missing | Run `python manage.py collectstatic` |

---

## Production Deployment

For production, make these changes in `settings.py`:

```python
DEBUG = False
ALLOWED_HOSTS = ['your-domain.com', 'your-server-ip']
```

Then use Gunicorn as the WSGI server:
```bash
pip install gunicorn
gunicorn myproject.wsgi:application --bind 0.0.0.0:8000
```

---

## License

Internal use — Devnetproject.
