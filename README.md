# CSF-LDF-WHM

## Overview

### What is CSF?

**ConfigServer Security & Firewall (CSF)** is a comprehensive stateful packet inspection (SPI) firewall, login/intrusion detection, and security application for Linux servers. CSF provides:

- Advanced firewall configuration with stateful packet inspection
- Login failure detection and automatic IP blocking
- Port flood protection
- Process tracking and monitoring
- Connection limiting and rate limiting
- Integration with popular control panels (WHM/cPanel, DirectAdmin, etc.)
- Email alerts for security events
- Extensive customization options

### What is LFD?

**Login Failure Daemon (LFD)** is a daemon process that runs alongside CSF and performs the following functions:

- Monitors log files for authentication failures
- Automatically blocks IPs that exceed login failure thresholds
- Detects and blocks port scanning attempts
- Monitors server load and resource usage
- Sends email alerts for security events
- Provides real-time protection against brute force attacks
- Tracks and blocks distributed attacks

Together, CSF and LFD provide a robust security solution for servers running WHM/cPanel, protecting against unauthorized access, brute force attacks, and various security threats.

## Prerequisites

Before installing CSF and LFD, ensure your system meets the following requirements:

### System Requirements

- **Operating System**: Linux-based OS (CentOS, RHEL, CloudLinux, Ubuntu, Debian, etc.)
- **Control Panel**: WHM/cPanel (this guide is specific to WHM/cPanel installations)
- **Root Access**: Root or sudo privileges required for installation
- **Perl**: Perl 5.8 or higher (usually pre-installed)
- **iptables**: Kernel support for iptables (standard on most Linux distributions)

### Network Requirements

- SSH access to the server
- Stable internet connection for downloading installation files
- Ability to modify firewall rules (ensure you have alternative access if SSH is blocked)

### Recommended Before Installation

- Take a backup of your current firewall configuration
- Document your current iptables rules (if any)
- Ensure you have console access (IPMI, KVM, or physical access) in case of network lockout
- Note down your current IP address to whitelist it during configuration

## Features and Capabilities

CSF/LFD provides extensive security features:

- **Firewall Management**: Easy-to-use interface for managing firewall rules
- **Intrusion Detection**: Real-time detection of suspicious activities
- **Login Monitoring**: Track failed login attempts across multiple services (SSH, FTP, SMTP, POP3, IMAP, etc.)
- **Country Blocking**: Block traffic from specific countries using GeoIP
- **Port Knocking**: Advanced security through port knocking sequences
- **Connection Tracking**: Monitor and limit connections per IP
- **Distributed Attack Protection**: Detect and block distributed login attacks
- **Process Monitoring**: Alert on suspicious processes
- **Directory Watching**: Monitor directories for changes
- **Exploit Detection**: Detect common server exploits
- **UI Integration**: Seamless integration with WHM interface

## Installation Guide

### INSTALL CODE

This guide provides step-by-step instructions for installing CSF and LFD on a WHM/cPanel server. The installation process will also remove APF (Advanced Policy Firewall) and BFD (Brute Force Detection) if present, as they are similar applications that would conflict with CSF/LFD over iptables control.

Execute the following commands as root:

```bash
# Navigate to the temporary directory
cd /tmp

# Remove any existing CSF archive to ensure a clean installation
rm -fv csf.tgz

# Download the latest CSF package from the official ConfigServer repository
wget https://download.configserver.com/csf.tgz

# Extract the downloaded archive
tar -xzf csf.tgz

# Navigate into the extracted CSF directory
cd csf

# Run the installation script
sh install.sh

# Remove APF and BFD if they exist to prevent conflicts
sh /etc/csf/remove_apf_bfd.sh
```

### Installation Command Explanations

1. **`cd /tmp`**: Changes to the temporary directory where installation files will be downloaded. Using /tmp ensures cleanup on reboot if needed.

2. **`rm -fv csf.tgz`**: Removes any existing CSF archive to prevent extraction conflicts. The `-f` flag forces removal without prompting, and `-v` provides verbose output.

3. **`wget https://download.configserver.com/csf.tgz`**: Downloads the latest stable version of CSF from the official ConfigServer repository.

4. **`tar -xzf csf.tgz`**: Extracts the downloaded tarball. The flags mean:
   - `-x`: Extract files
   - `-z`: Decompress using gzip
   - `-f`: Specify the filename

5. **`sh install.sh`**: Executes the CSF installation script, which:
   - Installs CSF and LFD files to `/etc/csf/`
   - Sets up init scripts
   - Configures basic firewall rules
   - Integrates with WHM/cPanel

6. **`sh /etc/csf/remove_apf_bfd.sh`**: Removes APF (Advanced Policy Firewall) and BFD (Brute Force Detection) if installed, preventing conflicts with CSF/LFD.

## Post-Installation Steps

After successful installation, follow these steps to configure CSF for production use:

### 1. Access CSF in WHM

1. Log in to WHM (Web Host Manager) as root
2. Navigate to **Plugins** → **ConfigServer Security & Firewall**
3. You should see the CSF interface with various configuration options

### 2. Disable Testing Mode

**Important**: CSF starts in testing mode by default, which automatically allows all traffic if there's a configuration issue.

1. Click the **"Configure Firewall"** button
2. Locate the `TESTING` setting (usually near the top)
3. Change `TESTING = "1"` to `TESTING = "0"`
4. Scroll to the bottom and click **"Change"**

### 3. Configure for VPS/Virtual Environments

If you're running CSF on a VPS or virtual private server:

1. In the same configuration screen, locate `MONOLITHIC_KERNEL`
2. Set `MONOLITHIC_KERNEL = "1"`
3. This setting is required for VPS environments that don't use modular kernels
4. Click **"Change"** to save

### 4. Restart CSF and LFD

1. Return to the main ConfigServer Security & Firewall page
2. Click **"Restart csf+lfd"**
3. Wait for the confirmation message that services have restarted successfully

### 5. Whitelist Trusted IPs

To prevent accidental lockout from trusted IP addresses:

1. On the main ConfigServer Security & Firewall page, locate the **"Quick Allow"** section
2. Enter your IP address or IP range
3. Add a comment describing the IP (e.g., "Office IP" or "Admin Home")
4. Click **"Quick Allow"**
5. Repeat for any other trusted IPs

### 6. Verify Installation

Run the following test to ensure CSF is working correctly:

```bash
# Test if CSF can properly configure iptables
perl /usr/local/csf/bin/csftest.pl
```

Expected output should indicate that CSF is functioning properly.

### 7. Review Default Configuration

Review and adjust these important settings based on your needs:

- **TCP_IN/TCP_OUT**: Ports allowed for incoming/outgoing TCP connections
- **UDP_IN/UDP_OUT**: Ports allowed for incoming/outgoing UDP connections
- **LF_TRIGGER**: Number of login failures before IP is blocked
- **LF_TRIGGER_PERM**: Permanent blocks after this many temporary blocks
- **CT_LIMIT**: Connection limit per IP address

## Configuration Recommendations

### Essential Security Settings

```bash
# Edit the CSF configuration file
vi /etc/csf/csf.conf
```

**Recommended settings for enhanced security:**

1. **Login Failure Detection**:
   ```
   LF_TRIGGER = "5"          # Block after 5 failed login attempts
   LF_TRIGGER_PERM = "3"     # Permanent block after 3 temporary blocks
   LF_SELECT = "1"           # Enable login failure detection
   ```

2. **Connection Limiting**:
   ```
   CONNLIMIT = "80;50"       # Limit 80 connections per IP to port 80
   PORTFLOOD = "80;tcp;20;5" # Block IPs making 20+ connections to port 80 within 5 seconds
   ```

3. **SYN Flood Protection**:
   ```
   SYNFLOOD = "1"            # Enable SYN flood protection
   SYNFLOOD_RATE = "100/s"   # Maximum SYN rate
   SYNFLOOD_BURST = "150"    # Maximum SYN burst
   ```

4. **Port Scan Detection**:
   ```
   PS_INTERVAL = "300"       # Port scan detection interval (seconds)
   PS_LIMIT = "10"           # Block after 10 port scans
   PS_PERMANENT = "1"        # Permanently block port scanners
   ```

5. **Email Alerts**:
   ```
   LF_EMAIL_ALERT = "1"      # Enable email alerts
   LF_ALERT_TO = "admin@yourdomain.com"  # Alert recipient
   ```

### Common Port Configurations

For typical web hosting environments:

```
# Incoming TCP ports
TCP_IN = "20,21,22,25,53,80,110,143,443,465,587,993,995,2077,2078,2082,2083,2086,2087,2095,2096"

# Outgoing TCP ports (usually allow all)
TCP_OUT = "1:65535"

# Incoming UDP ports
UDP_IN = "20,21,53"

# Outgoing UDP ports
UDP_OUT = "1:65535"
```

### Allow Cloudflare IPs (if using Cloudflare)

If your site uses Cloudflare, add their IPs to the allow list:

```bash
# Add to /etc/csf/csf.allow or use WHM interface
# Visit https://www.cloudflare.com/ips/ for current IP ranges
```

## Security Best Practices

### 1. Regular Updates

Keep CSF and LFD updated to the latest version:

```bash
# Update CSF to the latest version
csf -u
```

### 2. Monitor Email Alerts

- Configure email alerts to monitor security events
- Regularly review block notifications
- Investigate unusual patterns of failed login attempts

### 3. Whitelist Critical IPs

- Always whitelist your own IP addresses
- Add office/admin IPs to csf.allow
- Document whitelisted IPs with comments

### 4. Regular Backups

Backup your CSF configuration before making changes:

```bash
# Backup CSF configuration
cp -r /etc/csf /root/csf-backup-$(date +%Y%m%d)
```

### 5. Test Configuration Changes

After making configuration changes, test them:

```bash
# Restart CSF with configuration check
csf -r
```

### 6. Enable Country Blocking (Optional)

Block traffic from countries you don't serve:

```bash
# Edit /etc/csf/csf.conf
CC_DENY = "CN,RU,KP"  # Block China, Russia, North Korea (example)
CC_ALLOW_FILTER = "1"  # Enable country filtering
```

### 7. Implement Port Knocking (Advanced)

For enhanced SSH security, consider enabling port knocking:

```bash
# Edit /etc/csf/csf.conf
PORTKNOCKING = "22;KNOCK1,KNOCK2,KNOCK3"
```

### 8. Review Logs Regularly

Monitor CSF logs for security events:

```bash
# View recent blocks
tail -f /var/log/lfd.log

# View firewall deny log
tail -f /var/log/messages | grep CSF
```

### 9. Configure PHP Security

Add to your php.ini file:

```ini
disable_functions = show_source, system, shell_exec, passthru, exec, popen, proc_open
```

### 10. Disable Unnecessary Services

- Review running services and disable those not needed
- Close unused ports
- Use csf.conf to restrict port access

## Troubleshooting

### Common Issue 1: Missing iptables Kernel Module

**Error Message**:
```
iptables LKM ip_tables missing so this firewall cannot function unless you enable MONOLITHIC_KERNEL in /etc/csf/csf.conf
```

OR you receive emails saying:
```
lfd failed....A restart was attempted automagically
```

**Solution Option 1 (via WHM)**:

1. Log in to WHM
2. Navigate to **Plugins** → **ConfigServer Security & Firewall**
3. Click the **"Configure Firewall"** button
4. Scroll down to find `MONOLITHIC_KERNEL`
5. Set the value to `1` to enable it
6. Click the **"Change"** button at the bottom
7. Click **"Restart csf+lfd"** on the next page

**Solution Option 2 (via SSH)**:

```bash
# Edit the CSF configuration file
vi /etc/csf/csf.conf

# Change from:
MONOLITHIC_KERNEL = "0"

# To:
MONOLITHIC_KERNEL = "1"

# Save the file and restart CSF
csf -r
```

### Common Issue 2: Locked Out After Configuration

**Solution**:

If you're locked out due to CSF configuration:

1. Access via console (IPMI/KVM)
2. Stop CSF:
   ```bash
   csf -x
   ```
3. Add your IP to allow list:
   ```bash
   echo "YOUR_IP_ADDRESS # My IP" >> /etc/csf/csf.allow
   ```
4. Restart CSF:
   ```bash
   csf -s
   ```

### Common Issue 3: CSF Not Starting

**Diagnostic Steps**:

```bash
# Check CSF status
csf -v

# Test CSF functionality
perl /usr/local/csf/bin/csftest.pl

# Check for errors in logs
tail -50 /var/log/lfd.log

# Verify iptables is working
iptables -L -n
```

### Common Issue 4: High Server Load

If LFD is causing high server load:

```bash
# Edit /etc/csf/csf.conf
# Adjust these settings:
PT_LIMIT = "0"              # Disable process tracking if not needed
PS_INTERVAL = "0"           # Disable port scan detection if causing issues
```

### Common Issue 5: Email Alerts Not Working

**Solution**:

```bash
# Verify email settings in /etc/csf/csf.conf
LF_EMAIL_ALERT = "1"
LF_ALERT_TO = "valid@email.com"

# Test email functionality
echo "Test" | mail -s "CSF Test" your@email.com
```

### Common Issue 6: Unable to Access Certain Services

**Solution**:

Check if the service port is allowed in CSF:

```bash
# Check current port configuration
csf -g PORT_NUMBER

# Allow a specific port
csf -a PORT_NUMBER

# Or edit /etc/csf/csf.conf and add to TCP_IN or UDP_IN
```

## Additional Tools and Commands

### Check Server Security

Use CSF's built-in security checker:

```bash
# Run security check
perl /usr/local/csf/bin/csecheck.pl
```

Or via WHM: Navigate to ConfigServer Security & Firewall → **"Check Server Security"**

This tool provides recommendations for improving server security.

### Uninstall CSF

If you need to uninstall CSF completely:

```bash
# Stop CSF and LFD
csf -x

# Run the uninstall script
sh /etc/csf/uninstall.sh
```

### Useful CSF Commands

```bash
# Start CSF and LFD
csf -s

# Stop CSF and LFD (removes firewall rules)
csf -x

# Restart CSF and LFD
csf -r

# Reload CSF (faster than restart)
csf -q

# Check if an IP is blocked
csf -g IP_ADDRESS

# Temporarily allow an IP
csf -ta IP_ADDRESS

# Temporarily deny an IP
csf -td IP_ADDRESS

# Permanently allow an IP
csf -a IP_ADDRESS "Comment"

# Permanently deny an IP
csf -d IP_ADDRESS "Comment"

# Remove an IP from allow/deny lists
csf -ar IP_ADDRESS  # Remove from allow
csf -dr IP_ADDRESS  # Remove from deny

# View all current blocks
csf -l

# Update CSF to latest version
csf -u
```

## Additional Resources and Documentation

### Official Documentation

- **CSF Official Website**: [https://www.configserver.com/cp/csf.html](https://www.configserver.com/cp/csf.html)
- **CSF Documentation**: [https://download.configserver.com/csf/readme.txt](https://download.configserver.com/csf/readme.txt)
- **ConfigServer Forum**: [https://forum.configserver.com/](https://forum.configserver.com/)

### WHM/cPanel Resources

- **cPanel Documentation**: [https://docs.cpanel.net/](https://docs.cpanel.net/)
- **WHM Documentation**: [https://docs.cpanel.net/whm/](https://docs.cpanel.net/whm/)

### Security Resources

- **OWASP Top 10**: [https://owasp.org/www-project-top-ten/](https://owasp.org/www-project-top-ten/)
- **CIS Benchmarks**: [https://www.cisecurity.org/cis-benchmarks/](https://www.cisecurity.org/cis-benchmarks/)
- **Linux Security Best Practices**: [https://www.linux.com/topic/security/](https://www.linux.com/topic/security/)

### Community Support

- **ConfigServer Support Forum**: [https://forum.configserver.com/](https://forum.configserver.com/)
- **cPanel Forums**: [https://forums.cpanel.net/](https://forums.cpanel.net/)
- **ServerFault**: [https://serverfault.com/](https://serverfault.com/) (Search for CSF-related questions)

### Video Tutorials

- Search for "CSF WHM installation" on YouTube for video guides
- cPanel's official YouTube channel has WHM security tutorials

### Related Tools

- **Imunify360**: Commercial security solution alternative
- **Fail2Ban**: Alternative intrusion prevention system
- **ModSecurity**: Web application firewall (complements CSF)

## Support and Contributions

For issues, questions, or contributions related to this guide:

- Check the official CSF documentation first
- Visit the ConfigServer support forum
- Contact your hosting provider for server-specific assistance

---

**Note**: Always test configuration changes on a staging environment before applying them to production servers. Keep backups of your configurations and ensure you have alternative access methods (console/IPMI) before making firewall changes.

**Disclaimer**: This guide is provided for informational purposes. Always ensure you understand the implications of security configurations before applying them to production systems. The authors are not responsible for any issues arising from the use of this guide.
