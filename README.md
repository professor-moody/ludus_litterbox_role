# Ansible Role: Ludus LitterBox

[![License: GPL-3.0](https://img.shields.io/badge/License-GPL%203.0-blue.svg)](LICENSE)

An Ansible role for deploying [LitterBox](https://github.com/BlackSnufkin/LitterBox) - a secure sandbox environment for malware analysis and red team payload testing - on Windows systems in Ludus environments.

LitterBox provides a controlled sandbox environment designed for security professionals to develop and test payloads. This Ludus role automates the deployment and configuration of LitterBox on Windows VMs, including:

- **Testing evasion techniques** against modern detection methods
- **Validating detection signatures** before field deployment
- **Analyzing malware behavior** in an isolated environment
- **Keeping payloads in-house** without exposing them to external vendors
- **LLM-assisted analysis** through MCP server integration

## Requirements

### Ludus Host Requirements
- Ludus version 1.3.0 or higher
- Ansible 2.10 or higher
- Required Ansible Collections:
  - `ansible.windows` >= 2.0.0
  - `community.windows` >= 2.0.0
  - `chocolatey.chocolatey` >= 1.5.0

### Target VM Requirements
- Windows 10/11, Server 2019/2022
- Minimum 4GB RAM (8GB recommended)
- 10GB free disk space
- Administrator privileges

## Installation

### Install to a Ludus Range

```bash
# Install required collections on the Ludus server (if not already installed)
ansible-galaxy collection install ansible.windows
ansible-galaxy collection install community.windows
ansible-galaxy collection install chocolatey.chocolatey

# Or install from requirements file
ansible-galaxy collection install -r requirements.yml

# Add the role to your Ludus server
ludus ansible roles add professor-moody.ludus_litterbox_role

# Deploy to specific Windows VMs
ludus ansible deploy -t "litterbox_server" --limit "WS01,WS02"

# Or deploy to all Windows VMs
ludus ansible deploy -t "litterbox_server" --limit windows
```

### Manual Installation

```bash
# Clone the repository
git clone https://github.com/professor-moody/ludus_litterbox_role
cd ludus_litterbox_role

# Copy to your Ansible roles directory
cp -r . /etc/ansible/roles/ludus_litterbox_role/
```

## 📝 Role Variables

Available variables are listed below, along with default values (see `defaults/main.yml`):

```yaml
# Enable or disable LitterBox installation
ludus_litterbox_install: true

# Installation directory for LitterBox
ludus_litterbox_install_dir: "C:\\Tools\\LitterBox"

# Python version to install (if not already present)
ludus_litterbox_python_version: "3.11.9"

# Enable or disable Python installation
ludus_litterbox_install_python: true

# LitterBox repository URL
ludus_litterbox_repo_url: "https://github.com/BlackSnufkin/LitterBox.git"

# LitterBox server configuration
ludus_litterbox_host: "127.0.0.1"
ludus_litterbox_port: 1337

# Enable or disable auto-start service
ludus_litterbox_autostart: false

# Enable or disable firewall rule creation
ludus_litterbox_firewall_rule: true

# Enable or disable desktop shortcut creation
ludus_litterbox_desktop_shortcut: true

# Enable debug mode
ludus_litterbox_debug: false
```

## Example Playbook

### Basic Installation

```yaml
---
- hosts: windows_hosts
  roles:
    - role: ludus_litterbox
```

### Custom Configuration

```yaml
---
- hosts: windows_analysis_servers
  vars:
    ludus_litterbox_install_dir: "D:\\SecurityTools\\LitterBox"
    ludus_litterbox_port: 8080
    ludus_litterbox_autostart: true
    ludus_litterbox_debug: true
  roles:
    - role: ludus_litterbox
```

### Network-Accessible Configuration

```yaml
---
- hosts: litterbox_server
  vars:
    ludus_litterbox_host: "0.0.0.0"  # Listen on all interfaces
    ludus_litterbox_port: 1337
    ludus_litterbox_firewall_rule: true
  roles:
    - role: ludus_litterbox
```

## Post-Installation

After installation, LitterBox can be accessed in several ways:

### Web Interface
- Navigate to: `http://127.0.0.1:1337` (or your configured host:port)
- Use the desktop shortcut if created

### Start Methods
1. **Desktop Shortcut**: Double-click "LitterBox" on the desktop
2. **Batch File**: Run `C:\Tools\LitterBox\start_litterbox.bat`
3. **Manual Start**: 
   ```powershell
   cd C:\Tools\LitterBox\LitterBox
   .\venv\Scripts\python.exe litterbox.py
   ```

### API Access
The GrumpyCats client library is automatically downloaded and available for API access:
```python
from grumpycat import GrumpyCat
client = GrumpyCat("http://127.0.0.1:1337")
```

## Security Considerations

⚠️ **IMPORTANT SECURITY WARNINGS:**

1. **DEVELOPMENT USE ONLY**: This platform is designed exclusively for testing environments
2. **ISOLATION REQUIRED**: Execute only in isolated VMs or dedicated testing environments
3. **NO WARRANTY**: Provided without guarantees; use at your own risk
4. **LEGAL COMPLIANCE**: Users are responsible for ensuring all usage complies with applicable laws

### Recommended Security Practices:
- Deploy only on isolated, non-production systems
- Use network segmentation to isolate LitterBox instances
- Regularly snapshot VMs before testing unknown samples
- Monitor system resources during analysis
- Never expose LitterBox to the internet without proper authentication

## 📊 Analysis Capabilities

LitterBox provides comprehensive malware analysis features:

### Static Analysis
- YARA signature detection
- PE file analysis (imports, exports, sections)
- Document macro extraction
- String extraction and entropy analysis
- LNK file parsing

### Dynamic Analysis
- Process behavior monitoring
- Memory region inspection
- Code injection detection
- Sleep pattern analysis
- ETW telemetry collection

### Specialized Analysis
- **HolyGrail**: BYOVD vulnerability detection
- **Doppelganger**: Process similarity analysis
- **FuzzyHash**: Code similarity detection

## 🔄 Updates and Maintenance

### Update LitterBox
```powershell
cd C:\Tools\LitterBox\LitterBox
git pull
.\venv\Scripts\pip.exe install -r requirements.txt --upgrade
```

### Re-run the Role
```bash
ludus ansible deploy -t "litterbox_server" --limit "TARGET_VM"
```

## Troubleshooting

### Common Issues

1. **Python Installation Fails**
   - Manually install Python 3.11+ from python.org
   - Ensure PATH is configured correctly

2. **Git Clone Fails**
   - Check internet connectivity
   - Verify git is installed: `git --version`
   - Try manual clone in PowerShell

3. **Port Already in Use**
   - Change the port in variables: `ludus_litterbox_port: 8080`
   - Check for conflicting services

4. **Service Won't Start**
   - Check Windows Event Viewer for errors
   - Verify Python virtual environment is intact
   - Run manually to see error messages

### Debug Mode
Enable debug output by setting:
```yaml
ludus_litterbox_debug: true
```

## 📚 Additional Resources

- [LitterBox GitHub Repository](https://github.com/BlackSnufkin/LitterBox)
- [GrumpyCats Client Documentation](https://github.com/BlackSnufkin/GrumpyCats)
- [Ludus Documentation](https://docs.ludus.cloud)
- [Ansible Windows Modules](https://docs.ansible.com/ansible/latest/collections/ansible/windows/)

## Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

1. Fork the repository at https://github.com/professor-moody/ludus_litterbox_role
2. Create your feature branch (`git checkout -b feature/AmazingFeature`)
3. Commit your changes (`git commit -m 'Add some AmazingFeature'`)
4. Push to the branch (`git push origin feature/AmazingFeature`)
5. Open a Pull Request

## License

This project is licensed under the GPL-3.0 License - see the [LICENSE](LICENSE) file for details.

## Acknowledgments

- [BlackSnufkin](https://github.com/BlackSnufkin) - Original LitterBox author
- [Professor Moody](https://github.com/professor-moody) - Ludus role maintainer
- [Bad Sector Labs](https://github.com/badsectorlabs) - Ludus platform
- All contributors to the integrated analysis tools

## Disclaimer

This tool is intended for authorized security testing and research purposes only. Users are responsible for complying with all applicable laws and regulations. The authors assume no liability for misuse or damage caused by this software.

---

**Ludus Role Version**: 1.0.0  
**LitterBox Version**: Latest from GitHub  
**Last Updated**: 2024
