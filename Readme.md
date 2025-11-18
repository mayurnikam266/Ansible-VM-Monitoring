# Ansible VM Monitor

A comprehensive Ansible-based solution for monitoring virtual machines on AWS EC2, collecting system metrics (CPU, Memory, Disk usage), and sending beautifully formatted email reports with real-time health status indicators.

## 🚀 Features

- **Automated VM Discovery**: Dynamic inventory using AWS EC2 plugin
- **Real-time Metrics Collection**: CPU, Memory, and Disk usage monitoring
- **Beautiful HTML Reports**: Animated, responsive email reports with status indicators
- **Multi-OS Support**: Compatible with both Debian/Ubuntu and RedHat/CentOS systems
- **Configurable Alerting**: Email notifications with customizable thresholds
- **Health Status Indicators**: Color-coded status badges (Healthy, Warning, Critical)

## 📋 Prerequisites

### Required Software
- Ansible 2.9+ with the following collections:
  - `ansible.posix`
  - `amazon.aws`
- Python 3.6+
- AWS CLI configured with appropriate credentials

### AWS Requirements
- AWS Account with EC2 access
- IAM user with the following permissions:
  - `ec2:DescribeInstances`
  - `ec2:DescribeInstanceStatus`
- EC2 instances tagged with `Environment: dev`
- SSH access to target EC2 instances

### Email Configuration
- SMTP server access (Gmail, corporate email, etc.)
- Valid email credentials for sending reports

## 🏗️ Project Structure

```
Ansible-VM-Monitor/
├── ansible.cfg                          # Ansible configuration
├── playbook.yaml                        # Main orchestration playbook
├── collect_metrics.yaml                 # VM metrics collection tasks
├── send_report.yaml                     # Email reporting tasks
├── group_vars/
│   └── all.yaml                         # Global variables and email config
├── inventory/
│   └── aws_ec2.yaml                     # AWS EC2 dynamic inventory
└── templates/
    └── report_email_animated.html.j2    # HTML email template
```

## ⚙️ Configuration

### 1. AWS Configuration

Edit `inventory/aws_ec2.yaml`:

```yaml
plugin: amazon.aws.aws_ec2
regions:
  - ap-south-1                    # Change to your AWS region
filters:
  tag:Environment: dev            # Filter by Environment tag
  instance-state-name: running    # Only running instances
compose:
  ansible_host: public_ip_address # Use public IP for connection
keyed_groups:
  - key: tags.Environment
    prefix: env
```

### 2. Email Configuration

Edit `group_vars/all.yaml`:

```yaml
smtp_server: "smtp.gmail.com"     # Your SMTP server
smtp_port: 587                    # SMTP port
email_user: "your-email@gmail.com"    # Sender email
email_pass: "your-app-password"        # Email password/app password
alert_recipient: "admin@company.com"   # Recipient email
```

**Note**: For Gmail, use App Passwords instead of your regular password.

### 3. Ansible Configuration

The `ansible.cfg` file contains optimized settings:

```ini
[defaults]
inventory = ./inventory/aws_ec2.yaml
host_key_checking = False
remote_user = ubuntu              # Change if using different user
gathering = smart

[ssh_connection]
ssh_args = -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null
```

## 🚦 Usage

### Quick Start

1. **Clone the repository**:
   ```bash
   git clone https://github.com/jaiswaladi246/Ansible-VM-Monitor.git
   cd Ansible-VM-Monitor
   ```

2. **Configure AWS credentials**:
   ```bash
   aws configure
   ```

3. **Update configuration files**:
   - Set your AWS region in `inventory/aws_ec2.yaml`
   - Configure email settings in `group_vars/all.yaml`

4. **Test inventory discovery**:
   ```bash
   ansible-inventory --list
   ```

5. **Run the monitoring playbook**:
   ```bash
   ansible-playbook playbook.yaml
   ```

### Manual Execution

Run individual components:

```bash
# Collect metrics only
ansible-playbook collect_metrics.yaml

# Send report only (requires existing metrics)
ansible-playbook send_report.yaml
```

### Automated Scheduling

Add to crontab for automated monitoring:

```bash
# Edit crontab
crontab -e

# Add entry for hourly monitoring
0 * * * * cd /path/to/Ansible-VM-Monitor && ansible-playbook playbook.yaml

# Add entry for daily reports at 9 AM
0 9 * * * cd /path/to/Ansible-VM-Monitor && ansible-playbook playbook.yaml
```

## 📊 Metrics Collected

### System Metrics

| Metric | Description | Collection Method |
|--------|-------------|-------------------|
| **CPU Usage** | Percentage of CPU utilization | `mpstat` command with 1-second sampling |
| **Memory Usage** | Percentage of RAM utilization | `free` command output parsing |
| **Disk Usage** | Percentage of root filesystem usage | `df` command for root partition |

### Health Status Thresholds

| Status | CPU | Memory | Disk | Color |
|--------|-----|--------|------|-------|
| **Healthy** | < 50% | < 50% | < 50% | 🟢 Green |
| **Warning** | 50-80% | 50-80% | 50-80% | 🟡 Yellow |
| **Critical** | > 80% | > 80% | > 80% | 🔴 Red |

## 📧 Email Report Features

### Report Contents

- **Summary Dashboard**: Overview with total VMs and average metrics
- **Detailed Table**: Per-VM metrics with progress bars and status indicators
- **Visual Elements**: 
  - Animated progress bars
  - Color-coded status badges
  - Responsive design for mobile/desktop
  - Professional gradient styling

### Sample Report Structure

```
🚀 VM Health Dashboard
Real-time Infrastructure Monitoring & Analytics

Summary:
- 🖥️ Virtual Machines: 5
- 🔥 Average CPU: 45.2%
- 📊 Average Memory: 67.8%
- 💾 Average Disk: 34.5%

Detailed Metrics:
┌─────────────────┬─────────┬────────┬──────────┐
│ Hostname        │ CPU     │ Memory │ Disk     │
├─────────────────┼─────────┼────────┼──────────┤
│ web-server-01   │ 23.5%   │ 45.2%  │ 78.9%    │
│ api-server-02   │ 67.8%   │ 89.1%  │ 23.4%    │
└─────────────────┴─────────┴────────┴──────────┘
```

## 🔧 Customization

### Modifying Thresholds

Edit the HTML template to change status thresholds:

```jinja2
# In templates/report_email_animated.html.j2
{% if vm.cpu|float < 50 %}        # Change from 50 to your threshold
  <span class="status-badge status-healthy">
{% elif vm.cpu|float < 80 %}      # Change from 80 to your threshold
  <span class="status-badge status-warning">
{% else %}
  <span class="status-badge status-critical">
{% endif %}
```

### Adding New Metrics

1. **Add collection task** in `collect_metrics.yaml`:
   ```yaml
   - name: Get network usage
     shell: "cat /proc/net/dev | grep eth0"
     register: network_usage
   
   - name: Update metrics fact
     set_fact:
       vm_metrics:
         hostname: "{{ inventory_hostname }}"
         cpu: "{{ cpu_usage.stdout | float | round(2) }}"
         mem: "{{ mem_usage.stdout | float | round(2) }}"
         disk: "{{ disk_usage.stdout | float | round(2) }}"
         network: "{{ network_usage.stdout }}"  # New metric
   ```

2. **Update HTML template** to display new metric
3. **Modify report aggregation** in `send_report.yaml`

### Styling Customization

The HTML template uses CSS3 features:
- **Gradient backgrounds**: Modify color stops in CSS gradients
- **Animations**: Adjust `@keyframes` for different effects
- **Responsive breakpoints**: Change media query values
- **Color scheme**: Update CSS custom properties

## 🛠️ Troubleshooting

### Common Issues

#### 1. Inventory Discovery Fails
```
ERROR! Invalid plugin FQDN (amazon.aws.aws_ec2)
```
**Solution**: Install AWS collection:
```bash
ansible-galaxy collection install amazon.aws
```

#### 2. SSH Connection Issues
```
UNREACHABLE! => {"msg": "Failed to connect to the host via ssh"}
```
**Solutions**:
- Verify SSH key is added to ssh-agent
- Check security group allows SSH (port 22)
- Ensure public IP is accessible
- Verify `remote_user` in ansible.cfg matches EC2 instance user

#### 3. Email Sending Fails
```
FAILED! => {"msg": "Authentication failed"}
```
**Solutions**:
- Use App Passwords for Gmail (not regular password)
- Enable "Less secure app access" if using basic auth
- Check firewall allows SMTP traffic (port 587/465)
- Verify SMTP server and port settings

#### 4. Metrics Collection Fails
```
ERROR! failed: [instance] => {"msg": "sysstat not found"}
```
**Solution**: The playbook automatically installs `sysstat`, but if it fails:
```bash
# Manual installation on target servers
sudo apt-get update && sudo apt-get install -y sysstat  # Ubuntu/Debian
sudo yum install -y sysstat                             # CentOS/RHEL
```

#### 5. Permission Denied Errors
```
FAILED! => {"msg": "sudo: no tty present and no askpass program specified"}
```
**Solutions**:
- Ensure SSH user has sudo privileges
- Add user to sudoers: `usermod -aG sudo username`
- Configure passwordless sudo if needed

### Debug Mode

Enable verbose output for troubleshooting:
```bash
ansible-playbook playbook.yaml -v     # Verbose
ansible-playbook playbook.yaml -vv    # More verbose
ansible-playbook playbook.yaml -vvv   # Very verbose with connection debugging
```

### Testing Components

Test individual components:

```bash
# Test inventory
ansible-inventory --list -i inventory/aws_ec2.yaml

# Test connectivity
ansible all -m ping

# Test metric collection
ansible all -m shell -a "mpstat 1 1"

# Test email sending (localhost)
ansible localhost -m mail -a "to=test@example.com subject='Test' body='Test message'"
```

## 📈 Performance Considerations

### Optimization Tips

1. **Parallel Execution**:
   ```yaml
   # In ansible.cfg
   [defaults]
   forks = 20          # Increase parallel execution
   ```

2. **Fact Gathering**:
   ```yaml
   # Disable unnecessary fact gathering
   gather_facts: false  # Add to plays that don't need facts
   ```

3. **SSH Optimization**:
   ```yaml
   # In ansible.cfg
   [ssh_connection]
   ssh_args = -o ControlMaster=auto -o ControlPersist=60s
   pipelining = True
   ```

### Resource Requirements

| Component | CPU | Memory | Network |
|-----------|-----|--------|---------|
| **Ansible Controller** | 1 vCPU | 2GB RAM | 1Mbps |
| **Per Target VM** | ~1% impact | ~10MB | ~1KB/s |
| **Email Report** | Minimal | ~5MB | ~100KB |

## 🔒 Security Best Practices

### SSH Security
- Use SSH key-based authentication (no passwords)
- Implement bastion hosts for private networks
- Regularly rotate SSH keys
- Use SSH agent forwarding carefully

### AWS Security
- Follow principle of least privilege for IAM policies
- Use temporary credentials when possible
- Enable CloudTrail for API auditing
- Regularly review and rotate access keys

### Email Security
- Use App Passwords instead of account passwords
- Enable 2FA on email accounts
- Consider encrypted email solutions for sensitive data
- Validate recipient email addresses

### Ansible Security
- Store sensitive variables in Ansible Vault:
  ```bash
  ansible-vault create group_vars/vault.yaml
  ansible-vault encrypt_string 'password123' --name 'email_pass'
  ```
- Use `no_log: true` for sensitive tasks
- Limit playbook access with proper file permissions

## 📚 Advanced Configuration

### Multi-Environment Setup

Create environment-specific configurations:

```yaml
# group_vars/dev.yaml
smtp_server: "dev-smtp.company.com"
alert_recipient: "dev-team@company.com"

# group_vars/prod.yaml
smtp_server: "prod-smtp.company.com"
alert_recipient: "ops-team@company.com"
```

### Custom Filtering

Filter instances by multiple criteria:

```yaml
# inventory/aws_ec2.yaml
filters:
  tag:Environment: dev
  tag:Team: backend
  instance-state-name: running
  instance-type: [t3.micro, t3.small]
```

### Webhook Integration

Add webhook notifications:

```yaml
- name: Send webhook notification
  uri:
    url: "https://hooks.slack.com/services/YOUR/WEBHOOK/URL"
    method: POST
    body_format: json
    body:
      text: "VM Health Report: {{ collected_metrics | length }} VMs monitored"
```

## 🤝 Contributing

### Development Setup

1. **Fork the repository**
2. **Create a feature branch**:
   ```bash
   git checkout -b feature/new-metric
   ```
3. **Make your changes**
4. **Test thoroughly**:
   ```bash
   ansible-lint playbook.yaml
   ansible-playbook --check playbook.yaml
   ```
5. **Submit a pull request**

### Code Style

- Use 2-space indentation for YAML
- Follow Ansible best practices
- Add comments for complex logic
- Test on multiple OS distributions

## 📄 License

This project is licensed under the MIT License - see the LICENSE file for details.

## 👨‍💻 Author
**Mayur Nikam** 

## 📚 Credits

This project uses learning references/content from **Aditya Jaiswal**  
GitHub Profile: [@jaiswaladi246](https://github.com/jaiswaladi246)


##  Acknowledgments

- Ansible community for excellent documentation
- AWS for comprehensive EC2 API
- Bootstrap and modern CSS frameworks for design inspiration

## 📞 Support

For support and questions:
- Create an issue in the GitHub repository
- Check the troubleshooting section above
- Review Ansible documentation for advanced usage

---

*Last updated: November 18, 2025*
