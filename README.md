# System Administration Ansible Playbooks

Collection of Ansible playbooks for system administration, focusing on server updates, service management, and operational maintenance tasks.

## <¯ Overview

This repository contains operational Ansible playbooks used for maintaining and updating enterprise infrastructure. These playbooks focus on **safe, automated system updates** and **service management** for production environments.

## =Á Contents

### Server Update Playbooks

#### **yum_update_then_reboot_CentOS.yml**
Complete CentOS server update and reboot automation:
- **YUM package updates** - Updates all packages to latest versions
- **Safe reboot process** - Controlled server reboot with verification
- **Serial execution** - Updates servers one at a time to maintain availability
- **Post-reboot verification** - Ensures services come back online properly
- **Downtime minimization** - Designed for production environments

#### **yum_update_nagios_servers**
Targeted updates for Nagios monitoring servers:
- **Monitoring infrastructure updates** - Updates Nagios core and plugins
- **Service preservation** - Ensures monitoring continues during updates
- **Configuration backup** - Backs up Nagios config before updates
- **Verification checks** - Confirms monitoring functionality post-update

#### **yum_update_graylog_servers**
Graylog log management server updates:
- **Log infrastructure updates** - Updates Graylog and MongoDB components
- **Data preservation** - Ensures log data integrity during updates
- **Elasticsearch compatibility** - Maintains cluster health
- **Rollback capability** - Can revert if issues arise

#### **yum_update_elasticsearch_servers**
Elasticsearch search engine updates:
- **Search cluster updates** - Coordinates updates across Elasticsearch nodes
- **Cluster health protection** - Prevents data loss during updates
- **Index preservation** - Ensures data integrity
- **Rolling restart** - Updates nodes one at a time

### Configuration Management

#### **histtimeformat_etc_profile.yml**
Shell history configuration for audit trails:
- **History format standardization** - Adds timestamps to command history
- **Security compliance** - Improves audit capabilities for system administration
- **User consistency** - Ensures all users have proper history format
- **Operational visibility** - Better tracking of administrative actions

## =€ Usage

### Prerequisites
```bash
# Install Ansible
pip install ansible

# Or use package manager
sudo yum install ansible  # CentOS/RHEL
sudo apt-get install ansible  # Ubuntu/Debian
```

### Server Updates

#### Complete CentOS Update and Reboot
```bash
# Interactive execution with confirmation
ansible-playbook -i inventory yum_update_then_reboot_CentOS.yml

# Update specific servers only
ansible-playbook -i inventory yum_update_then_reboot_CentOS.yml --limit "web-*"

# Schedule for specific time window
ansible-playbook -i inventory yum_update_then_reboot_CentOS.yml --extra-vars "maintenance_window=true"
```

#### Infrastructure-Specific Updates
```bash
# Update monitoring infrastructure
ansible-playbook -i inventory yum_update_nagios_servers

# Update logging infrastructure
ansible-playbook -i inventory yum_update_graylog_servers

# Update search cluster
ansible-playbook -i inventory yum_update_elasticsearch_servers
```

### Configuration Management
```bash
# Standardize shell history format
ansible-playbook -i inventory histtimeformat_etc_profile.yml

# Apply to specific environment
ansible-playbook -i inventory histtimeformat_etc_profile.yml --limit production
```

## =' Configuration

### Inventory Setup

**Example inventory file:**
```ini
[centos_servers]
web-server-1.example.com
web-server-2.example.com
app-server-1.example.com

[nagios_servers]
nagios-1.example.com
nagios-2.example.com

[graylog_servers]
graylog-1.example.com
graylog-2.example.com

[elasticsearch_servers]
es-node-1.example.com
es-node-2.example.com
es-node-3.example.com

[all:vars]
ansible_user=ansible_admin
ansible_become=true
ansible_become_method=sudo
```

### Update Safety Features

**Serial Execution:**
```yaml
# Updates one server at a time
serial: 1

# Update in batches of 5
serial: 5

# Update 10% of servers at a time
serial: "10%"
```

**Maintenance Windows:**
```yaml
# Only run during maintenance hours
when: maintenance_window | bool
```

**Pre-update Tasks:**
```yaml
# Backup configurations
- name: Backup current configuration
  copy:
    src: /etc/myapp
    dest: "/backup/myapp-{{ ansible_date_time.iso8601 }}"
```

## =à Advanced Usage

### Rolling Updates
```bash
# Update web servers in controlled groups
ansible-playbook -i inventory yum_update_then_reboot_CentOS.yml \
  --limit "web-*" \
  --extra-vars "serial=2 batch_size=2"
```

### Verification and Rollback
```bash
# Run with check mode (dry-run)
ansible-playbook -i inventory yum_update_then_reboot_CentOS.yml --check

# Diff mode shows what would change
ansible-playbook -i inventory yum_update_then_reboot_CentOS.yml --diff

# Run specific tags only
ansible-playbook -i inventory yum_update_then_reboot_CentOS.yml --tags "pre-update,verify"
```

### Scheduling Automation
```bash
# Cron-based maintenance (example cron entries)
# Weekly CentOS updates - Sundays 2AM
0 2 * * 0 /usr/bin/ansible-playbook /path/to/yum_update_then_reboot_CentOS.yml

# Monthly Nagios updates - First Sunday 3AM
0 3 1-7 * 0 /usr/bin/ansible-playbook /path/to/yum_update_nagios_servers
```

## = Monitoring and Verification

### Update Verification
```bash
# Check server uptime after updates
ansible -i inventory all -m shell -a "uptime"

# Verify service status
ansible -i inventory nagios_servers -m shell -a "systemctl status nagios"

# Check update logs
ansible -i inventory centos_servers -m shell -a "tail -50 /var/log/yum.log"
```

### Health Checks
```bash
# Verify Elasticsearch cluster health
curl -XGET 'http://elasticsearch-server:9200/_cluster/health?pretty'

# Check Graylog functionality
curl -u admin:password 'http://graylog-server:9000/api/system'

# Test Nagios monitoring
curl 'http://nagios-server/nagios/cgi-bin/status.cgi?host=all'
```

##   Important Notes

### Safety Precautions
- **Test environments first** - Always test in non-production before production
- **Backup configurations** - Ensure you have recent backups before updates
- **Maintenance windows** - Schedule updates during low-traffic periods
- **Serial execution** - Use serial updates to maintain availability
- **Monitoring ready** - Ensure monitoring is working before starting updates

### Risk Mitigation
- **Rollback planning** - Know how to revert if updates fail
- **Service verification** - Confirm critical services restart properly
- **Resource monitoring** - Watch system resources during updates
- **Communication** - Notify teams of scheduled maintenance

## =' Troubleshooting

### Common Issues

**Update Fails**
```bash
# Check yum errors
ansible -i inventory centos_servers -m shell -a "tail -100 /var/log/yum.log"

# Clear yum cache and retry
ansible -i inventory centos_servers -m shell -a "yum clean all"
```

**Server Doesn't Come Back**
```bash
# Check server status
ansible -i inventory centos_servers -m ping

# Manual verification
ssh admin@server "uptime; systemctl status"
```

**Service Failures**
```bash
# Check service status
ansible -i inventory all -m shell -a "systemctl status --failed"

# Restart specific services
ansible -i inventory all -m service -a "name=myapp state=restarted"
```

## =Ë Best Practices

### Update Strategy
1. **Plan maintenance windows** - Choose low-traffic periods
2. **Test in staging** - Verify updates work in test environment first
3. **Monitor progress** - Watch for issues during updates
4. **Verify functionality** - Test applications after updates
5. **Document changes** - Keep records of what was updated

### Operational Excellence
- **Use version control** - Track playbook changes
- **Automate verification** - Add post-update validation tasks
- **Communicate status** - Send notifications before/after updates
- **Monitor metrics** - Watch performance indicators
- **Prepare rollback** - Know restoration procedures

## =Ú Related Resources

- **Misc_Ansible_Playbooks**: General Ansible automation playbooks
- **percona_client_install_ansible**: Database installation automation
- **SysAdmin-Shell-Scripts**: Complementary shell administration scripts
- **aws_python_sdk**: AWS infrastructure automation

## > Contributing

Contributions welcome! Please:
1. Test playbooks in non-production first
2. Include safety checks and rollback procedures
3. Add documentation for new variables
4. Follow Ansible best practices

## =Ý License

System administration playbooks - Free to use and modify.

---

**System Administration Ansible Playbooks** - Automating operational maintenance with safe, tested, and reliable server update procedures.