# 📊 Infrastructure Monitoring Stack - Ansible Automation

[![Ansible](https://img.shields.io/badge/Ansible-EE0000?style=flat&logo=ansible&logoColor=white)](https://www.ansible.com/)
[![Prometheus](https://img.shields.io/badge/Prometheus-E6522C?style=flat&logo=prometheus&logoColor=white)](https://prometheus.io/)
[![Grafana](https://img.shields.io/badge/Grafana-F46800?style=flat&logo=grafana&logoColor=white)](https://grafana.com/)
[![License](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)

> **Automated deployment of a complete monitoring infrastructure using Ansible for CentOS/RHEL environments**

This project provides a production-ready, automated deployment solution for setting up a comprehensive metrics collection and visualization stack across multiple servers using Ansible automation.

---

## 📸 Screenshots

### Monitoring Dashboard
![Monitoring Dashboard](images/dashboard.png)
*Real-time system metrics visualization in Grafana*

### Architecture Diagram
![Architecture](images/architecture.png)
*Complete infrastructure overview*

---

## 🎯 Overview

This project automates the complete lifecycle deployment of a modern monitoring infrastructure, eliminating manual configuration and ensuring consistent deployments across environments. Built with infrastructure-as-code principles, it enables rapid setup of production-grade monitoring capabilities.

### Key Features

- 🚀 **One-Command Deployment** – Complete stack setup with a single Ansible playbook execution
- 🐳 **Containerized Services** – All components run in Podman containers for isolation and portability
- 📈 **Real-Time Monitoring** – Continuous metrics collection with 30-second scrape intervals
- 🚨 **Intelligent Alerting** – Pre-configured alert rules for CPU, memory, disk, and availability
- 🎨 **Visualization Ready** – Grafana dashboards for immediate insights
- 🔄 **Idempotent Operations** – Safe to run multiple times without side effects
- 📦 **Role-Based Design** – Modular Ansible roles for maintainability and reusability

---

## 🏗️ Architecture

The infrastructure consists of three logical components:

```
┌─────────────────────┐
│  Control Machine    │
│   (Ansible Host)    │
└──────────┬──────────┘
           │
           │ SSH + Ansible
           │
           ├─────────────────────────────────┐
           │                                 │
           ▼                                 ▼
┌─────────────────────┐          ┌─────────────────────┐
│    Metrics Hub      │          │   Target Servers    │
│                     │◄─────────┤  (Node Exporter)    │
│  • Prometheus:9090  │  Scrape  │                     │
│  • Grafana:3000     │  Metrics │  • Server 1:9100    │
│  • Alertmanager:9093│          │  • Server 2:9100    │
└─────────────────────┘          │  • Server N:9100    │
                                 └─────────────────────┘
```

### Components

| Component | Purpose | Port | Technology |
|-----------|---------|------|------------|
| **Prometheus** | Time-series database and metrics aggregator | 9090 | Docker/Podman |
| **Grafana** | Data visualization and dashboard platform | 3000 | Docker/Podman |
| **Alertmanager** | Alert handling and notification routing | 9093 | Docker/Podman |
| **Node Exporter** | System metrics exporter (CPU, RAM, disk, network) | 9100 | Docker/Podman |

---

## 🛠️ Technologies

- **Ansible** – Infrastructure automation and configuration management
- **Prometheus** – Metrics collection and time-series database
- **Grafana** – Dashboard and visualization platform
- **Node Exporter** – Hardware and OS metrics exporter
- **Alertmanager** – Alert deduplication, grouping, and routing
- **Podman/Docker** – Container runtime for service isolation
- **YAML** – Configuration and playbook definitions

---

## 📁 Project Structure

```
.
├── README.md                        # This file
├── ansible.cfg                      # Ansible configuration
├── inventory                        # Server inventory with credentials
├── playbook.yml                     # Main playbook orchestrating roles
└── roles/
    ├── metrics_hub/                 # Monitoring hub role
    │   ├── files/
    │   │   ├── prometheus.yml       # Prometheus configuration
    │   │   ├── alert_rules.yml      # Alert rule definitions
    │   │   └── alertmanager.yml     # Alertmanager configuration
    │   └── tasks/
    │       └── main.yml             # Hub deployment tasks
    └── system_exporter/             # Node exporter role
        └── tasks/
            └── main.yml             # Exporter deployment tasks
```

---

## 🚀 Getting Started

### Prerequisites

Ensure you have the following installed and configured:

- **Control Machine:**
  - Ansible 2.9+ installed
  - Python 3.6+
  - Network connectivity to target servers

- **Target Servers (CentOS/RHEL 7+):**
  - Podman or Docker installed
  - SSH access with sudo privileges
  - Ports 9090, 9093, 3000, and 9100 available
  - Python 3 installed

### Installation

1. **Clone the repository**
   ```bash
   git clone <repository-url>
   cd new-project
   ```

2. **Configure your inventory**
   
   Edit the `inventory` file with your server details:
   ```ini
   [metrics_hub]
   <your-hub-ip> ansible_user=<username> ansible_password=<password> ansible_become_password=<password>

   [target_servers]
   <server1-ip> ansible_user=<username> ansible_password=<password> ansible_become_password=<password>
   <server2-ip> ansible_user=<username> ansible_password=<password> ansible_become_password=<password>
   ```

   > ⚠️ **Security Note:** For production environments, use Ansible Vault to encrypt sensitive credentials or SSH key-based authentication.

3. **Update Prometheus targets**
   
   Edit `roles/metrics_hub/files/prometheus.yml` and add your target servers:
   ```yaml
   scrape_configs:
     - job_name: 'system_metrics'
       static_configs:
         - targets: 
           - '<server1-ip>:9100'
           - '<server2-ip>:9100'
   ```

4. **Deploy the stack**
   ```bash
   ansible-playbook -i inventory playbook.yml
   ```

5. **Verify deployment**
   
   Check that all containers are running on the metrics hub:
   ```bash
   ssh <metrics-hub-ip>
   podman ps
   ```

---

## 📊 Access & Usage

Once deployed, access the following interfaces:

| Service | URL | Default Credentials |
|---------|-----|---------------------|
| **Grafana** | `http://<metrics-hub-ip>:3000` | Username: `admin`<br>Password: `admin123` |
| **Prometheus** | `http://<metrics-hub-ip>:9090` | No authentication |
| **Alertmanager** | `http://<metrics-hub-ip>:9093` | No authentication |

### Setting Up Grafana Dashboards

1. Log into Grafana
2. Navigate to **Configuration** → **Data Sources**
3. Add Prometheus data source: `http://localhost:9090`
4. Import pre-built Node Exporter dashboard (ID: 1860)
5. View real-time metrics for all monitored servers

---

## 🔔 Alert Rules

The project includes pre-configured alert rules monitoring:

| Alert | Condition | Threshold | Duration |
|-------|-----------|-----------|----------|
| **Instance Down** | Service unreachable | `up == 0` | 2 minutes |
| **High CPU Usage** | CPU utilization | > 80% | 3 minutes |
| **High Memory Usage** | Memory utilization | > 85% | 3 minutes |
| **Low Disk Space** | Available disk space | < 15% | 5 minutes |

Alerts are managed by Alertmanager and can be configured to send notifications via webhook, email, Slack, or other integrations by modifying `roles/metrics_hub/files/alertmanager.yml`.

---

## 🔧 Configuration

### Customizing Scrape Intervals

Edit `roles/metrics_hub/files/prometheus.yml`:
```yaml
global:
  scrape_interval: 30s  # Change to desired interval
```

### Modifying Alert Thresholds

Edit `roles/metrics_hub/files/alert_rules.yml`:
```yaml
- alert: HighCPUUsage
  expr: 100 - (avg by(instance) (irate(node_cpu_seconds_total{mode="idle"}[5m])) * 100) > 80  # Adjust threshold
```

### Changing Grafana Password

Modify the environment variable in `roles/metrics_hub/tasks/main.yml`:
```yaml
env:
  GF_SECURITY_ADMIN_PASSWORD: "your-secure-password"
```

---

## 🐛 Troubleshooting

### Services Not Starting

Check container logs:
```bash
podman logs prometheus-metrics
podman logs grafana-viz
podman logs alertmanager-main
```

### Port Already in Use

The playbook includes automatic cleanup of conflicting processes. If issues persist, manually check:
```bash
sudo lsof -i :9090  # Check Prometheus port
sudo lsof -i :3000  # Check Grafana port
```

### Targets Not Appearing in Prometheus

1. Verify Node Exporter is running on target servers: `podman ps`
2. Check network connectivity: `curl http://<target-ip>:9100/metrics`
3. Verify Prometheus configuration includes correct target IPs

---

## 🔐 Security Considerations

- **Change default passwords** before deploying to production
- **Use Ansible Vault** to encrypt the inventory file: `ansible-vault encrypt inventory`
- **Configure firewall rules** to restrict access to monitoring ports
- **Enable HTTPS** for Grafana in production environments
- **Implement authentication** for Prometheus and Alertmanager
- **Use SSH keys** instead of password-based authentication

---

## 🤝 Contributing

Contributions are welcome! Please feel free to submit a Pull Request. For major changes, please open an issue first to discuss what you would like to change.

---

## 📝 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

---

## 👤 Author

**Your Name**
- GitHub: [@yourusername](https://github.com/yourusername)
- LinkedIn: [Your Profile](https://linkedin.com/in/yourprofile)

---

## 🙏 Acknowledgments

- [Prometheus Documentation](https://prometheus.io/docs/)
- [Grafana Documentation](https://grafana.com/docs/)
- [Ansible Best Practices](https://docs.ansible.com/ansible/latest/user_guide/playbooks_best_practices.html)
- [Node Exporter](https://github.com/prometheus/node_exporter)

---

## 📈 Roadmap

- [ ] Add support for additional exporters (MySQL, PostgreSQL, etc.)
- [ ] Implement Grafana dashboard provisioning
- [ ] Add SSL/TLS configuration for all services
- [ ] Create automated backup and restore procedures
- [ ] Add support for multi-cloud deployments
- [ ] Integrate with external notification systems (Slack, PagerDuty)

---

**⭐ If you find this project useful, please consider giving it a star!**
