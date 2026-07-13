# 🛡️ Home-SIEM - Security Information & Event Management

A lightweight, self-hosted SIEM solution for monitoring security events, detecting threats, and visualizing security data in real-time.

## Dashboard Preview
![image alt](https://github.com/Kohana23/Home-SIEM-Lab/blob/main/image.png?raw=true)


## 📋 Table of Contents
- [Overview](#overview)
- [Features](#features)
- [Architecture](#architecture)
- [Prerequisites](#prerequisites)
- [Installation](#installation)
- [Configuration](#configuration)
- [Running the System](#running-the-system)
- [Windows Sysmon Integration](#windows-sysmon-integration)
- [API Endpoints](#api-endpoints)
- [Troubleshooting](#troubleshooting)
- [Contributing](#contributing)
- [License](#license)

## 🎯 Overview

Home-SIEM is a comprehensive security monitoring solution that collects, analyzes, and visualizes security events from multiple sources. Built with FastAPI for the backend and React for the frontend, it provides real-time threat detection and alerting capabilities.

## ✨ Features

- **Real-time Log Collection**: Ingest logs from various sources including Windows Sysmon
- **Automated Alerting**: Generate alerts based on suspicious activities
- **Interactive Dashboard**: Visualize security events with charts, maps, and tables
- **Risk Scoring**: Automatically assign severity levels to events
- **Geographic Threat Visualization**: Map-based display of security events
- **Sysmon Integration**: Process Windows Sysmon events for detailed system monitoring
- **RESTful API**: Easy integration with other tools and scripts
- **SQLite Database**: Lightweight storage with no external dependencies

## 🏗️ Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    Home-SIEM Architecture                    │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────┐   │
│  │   Windows   │    │   Windows   │    │   Ubuntu    │   │
│  │     VM      │    │    Host     │    │   Server    │   │
│  │  (Sysmon)   │    │   (React)   │    │ (FastAPI)   │   │
│  └──────┬──────┘    └──────┬──────┘    └──────┬──────┘   │
│         │                  │                  │           │
│         └──────────────────┼──────────────────┘           │
│                            │                              │
│                     ┌──────┴──────┐                       │
│                     │  Database   │                       │
│                     │  (SQLite)   │                       │
│                     └─────────────┘                       │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

## 📦 Prerequisites

### System Requirements
- **Ubuntu 20.04+** or **Debian 11+** (Backend)
- **Windows 10/11** or **Windows Server 2019+** (Optional for Sysmon)
- **Python 3.8+**
- **Node.js 16+** and **npm**
- **Git**

### Required Packages (Ubuntu)
```bash
sudo apt update
sudo apt install -y python3-pip python3-venv nodejs npm curl git
```

### Required Packages (Windows)
- [PowerShell 5.1+](https://docs.microsoft.com/en-us/powershell/)
- [Sysmon](https://docs.microsoft.com/en-us/sysinternals/downloads/sysmon)

## 🚀 Installation

### Clone the Repository
```bash
git clone https://github.com/yourusername/home-siem-unique.git
cd home-siem-unique
```

### 1. Backend Setup (Ubuntu)

```bash
# Navigate to backend directory
cd backend

# Create virtual environment
python3 -m venv venv

# Activate virtual environment
source venv/bin/activate  # On Ubuntu
# venv\Scripts\activate  # On Windows

# Install dependencies
pip install -r requirements.txt

# Additional dependencies for development
pip install uvicorn fastapi sqlalchemy python-multipart
```

### 2. Frontend Setup (Windows or Ubuntu)

```bash
# Navigate to frontend directory
cd frontend

# Install dependencies
npm install

# Install additional dependencies
npm install axios react-router-dom recharts react-leaflet leaflet
```

## ⚙️ Configuration

### Backend Configuration

Create a `.env` file in the `backend` directory:
```bash
# backend/.env
DATABASE_URL=sqlite:///./siem.db
SECRET_KEY=your-secret-key-here
DEBUG=True
```

### Frontend Configuration

Create a `.env` file in the `frontend` directory:
```bash
# frontend/.env
REACT_APP_API_URL=http://localhost:5000/api
```

### VirtualBox Network Setup (Optional)

For connecting Windows VM with Sysmon:

1. **Create NAT Network**:
   - VirtualBox → File → Preferences → Network
   - Add NAT Network: Name: `LabNet`, CIDR: `10.0.2.0/24`

2. **Configure VMs**:
   - Set both VMs to use NAT Network
   - Linux IP: `10.0.2.4` (or check with `ip addr`)
   - Windows IP: `10.0.2.15` (or check with `ipconfig`)

## 🏃 Running the System

### 1. Start Backend Server (Ubuntu)

```bash
cd backend
source venv/bin/activate
uvicorn app:app --reload --host 0.0.0.0 --port 5000
```

**Expected Output:**
```
INFO:     Started server process [12345]
INFO:     Waiting for application startup.
INFO:     Application startup complete.
INFO:     Uvicorn running on http://0.0.0.0:5000 (Press CTRL+C to quit)
```

### 2. Start Frontend (Windows)

```bash
cd frontend
npm start
```

**Open in browser:** `http://localhost:3000`

### 3. Test API Connection (Ubuntu)

```bash
# Test health endpoint
curl http://localhost:5000/api/health

# Expected: {"ok": true}

# Test logs endpoint
curl http://localhost:5000/api/logs

# Test alerts endpoint
curl http://localhost:5000/api/alerts

# Test stats endpoint
curl http://localhost:5000/api/stats
```

## 🪟 Windows Sysmon Integration

### Install Sysmon (Windows VM)

```powershell
# Download Sysmon
Invoke-WebRequest -Uri "https://download.sysinternals.com/files/Sysmon.zip" -OutFile "Sysmon.zip"
Expand-Archive -Path "Sysmon.zip" -DestinationPath "C:\Sysmon"

# Install with default configuration
cd C:\Sysmon
.\Sysmon.exe -accepteula -i
```

### Run Sysmon Forwarder (Windows VM)

Create and run the forwarder script:

```powershell
# Create the forwarder script
$forwarderScript = @'
# sysmon-forwarder.ps1
$LOG_URL = "http://10.0.2.4:5000/api/logs"
$ALERT_URL = "http://10.0.2.4:5000/api/alerts"
$EVENT_LOG_NAME = "Microsoft-Windows-Sysmon/Operational"

function Send-Log {
    param($source, $message)
    $data = @{source=$source; message=$message} | ConvertTo-Json
    try {
        $response = Invoke-RestMethod -Uri $LOG_URL -Method Post -Body $data -ContentType "application/json"
        return $true
    } catch { return $false }
}

function Send-Alert {
    param($severity, $description)
    $data = @{severity=$severity; description=$description} | ConvertTo-Json
    try {
        $response = Invoke-RestMethod -Uri $ALERT_URL -Method Post -Body $data -ContentType "application/json"
        return $true
    } catch { return $false }
}

while ($true) {
    try {
        $events = Get-WinEvent -LogName $EVENT_LOG_NAME -MaxEvents 5 -ErrorAction Stop
        foreach ($event in $events) {
            Send-Log -source "Sysmon" -message "Event $($event.Id): $($event.Message)"
            
            # Generate alerts for suspicious events
            if ($event.Id -in @(1, 3, 7, 8, 10)) {
                $severity = if ($event.Id -in @(8, 10)) { "High" } else { "Medium" }
                Send-Alert -severity $severity -description "Suspicious Sysmon Event $($event.Id): $($event.Message)"
            }
        }
        Start-Sleep -Seconds 30
    } catch {
        Start-Sleep -Seconds 60
    }
}
'@

$forwarderScript | Out-File -FilePath "C:\Sysmon\sysmon-forwarder.ps1" -Encoding UTF8

# Run the forwarder
cd C:\Sysmon
.\sysmon-forwarder.ps1
```

### Test Sysmon Integration (Windows VM)

```powershell
# Test connection to backend
Invoke-RestMethod -Uri "http://10.0.2.4:5000/api/health"

# Test sending log
$logData = @{source="Test-VM"; message="Test log from Windows VM"} | ConvertTo-Json
Invoke-RestMethod -Uri "http://10.0.2.4:5000/api/logs" -Method Post -Body $logData -ContentType "application/json"

# Test sending alert
$alertData = @{severity="High"; description="Test alert from Windows VM"} | ConvertTo-Json
Invoke-RestMethod -Uri "http://10.0.2.4:5000/api/alerts" -Method Post -Body $alertData -ContentType "application/json"

# Check if data was received
curl http://10.0.2.4:5000/api/logs
curl http://10.0.2.4:5000/api/alerts
```

## 📡 API Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/api/health` | Health check |
| GET | `/api/logs` | Get all logs |
| POST | `/api/logs` | Add a log entry |
| GET | `/api/alerts` | Get all alerts |
| POST | `/api/alerts` | Create an alert |
| GET | `/api/stats` | Get system statistics |
| GET | `/api/alerts/trends` | Get alert trends for charts |
| GET | `/api/alerts/severity` | Get severity distribution |

### Example API Requests

**Add a log:**
```bash
curl -X POST http://localhost:5000/api/logs \
  -H "Content-Type: application/json" \
  -d '{"source": "Server1", "message": "User logged in"}'
```

**Add an alert:**
```bash
curl -X POST http://localhost:5000/api/alerts \
  -H "Content-Type: application/json" \
  -d '{"severity": "High", "description": "Unauthorized access detected"}'
```

**Get statistics:**
```bash
curl http://localhost:5000/api/stats
# Returns: {"logs": 150, "alerts": 45, "critical": 12}
```

## 🔧 Troubleshooting

### Common Issues and Solutions

| Issue | Solution |
|-------|----------|
| **Backend not starting** | Check if port 5000 is in use: `sudo lsof -i :5000` |
| **Frontend can't connect** | Ensure backend is running and CORS is enabled |
| **Sysmon events not showing** | Run PowerShell as Administrator |
| **Connection timeout** | Check network settings in VirtualBox |
| **API returns 422 error** | Verify JSON format matches the API expectations |

### Logs Location
- **Backend logs**: Console output
- **Database**: `backend/siem.db`
- **Frontend logs**: Browser console (F12)
- **Sysmon logs**: `Microsoft-Windows-Sysmon/Operational` in Event Viewer

### Network Verification

**Ubuntu Host:**
```bash
# Check if backend is listening
netstat -tuln | grep 5000

# Check firewall
sudo ufw status
```

**Windows VM:**
```powershell
# Test connectivity
Test-NetConnection 10.0.2.4 -Port 5000

# Check IP
ipconfig
```

## 📁 Project Structure

```
home-siem-unique/
├── backend/
│   ├── app.py                 # Main FastAPI application
│   ├── requirements.txt       # Python dependencies
│   ├── siem.db               # SQLite database
│   ├── parser/
│   │   └── sysmon_parser.py   # Sysmon log parser
│   └── core/
│       ├── db.py             # Database configuration
│       └── models.py         # Database models
├── frontend/
│   ├── src/
│   │   ├── App.js            # Main React component
│   │   ├── api.js            # API service layer
│   │   ├── index.css         # Global styles
│   │   └── components/
│   │       ├── RiskGauge.js
│   │       ├── Timeline.js
│   │       ├── GeoMap.js
│   │       └── LogTable.js
│   ├── package.json
│   └── public/
│       └── index.html
├── sysmon/
│   └── Sysmon-config.xml      # Sysmon configuration
├── demo/
│   └── demo_post_event.sh     # Demo script
└── README.md
```

## 🤝 Contributing

1. Fork the repository
2. Create a new branch: `git checkout -b feature/your-feature`
3. Make your changes
4. Test thoroughly
5. Submit a pull request

### Development Guidelines
- Follow PEP 8 for Python code
- Use ESLint for JavaScript/React code
- Write meaningful commit messages
- Add comments for complex logic
- Update documentation when adding features

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## 🙏 Acknowledgments

- [FastAPI](https://fastapi.tiangolo.com/)
- [React](https://reactjs.org/)
- [Sysmon](https://docs.microsoft.com/en-us/sysinternals/downloads/sysmon)
- [SQLAlchemy](https://www.sqlalchemy.org/)
- [Recharts](https://recharts.org/)
- [Leaflet](https://leafletjs.com/)

## 🚀 Roadmap

- [ ] Real-time WebSocket updates
- [ ] Email/Webhook notifications
- [ ] Custom alert rules
- [ ] Advanced threat intelligence
- [ ] Docker containerization
- [ ] Kubernetes deployment
- [ ] Multi-tenancy support

---

**Made with ❤️ for the cybersecurity community**

If you find this project useful, please ⭐ star the repository!
