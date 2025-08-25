# Ludus LitterBox

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

A Ludus role for deploying [LitterBox](https://github.com/BlackSnufkin/LitterBox) - a comprehensive malware analysis sandbox - on Windows systems within Ludus lab environments.

## Overview

LitterBox provides a controlled sandbox environment designed for security professionals to analyze malware and test security tools. This Ludus role automates the deployment and configuration of LitterBox on Windows VMs, providing:

- **Static and Dynamic Analysis**: Comprehensive malware behavior analysis
- **Multiple Analysis Engines**: YARA, PE analysis, memory inspection, and more
- **Isolated Testing Environment**: Safe execution and analysis of suspicious files
- **Web-Based Interface**: Easy-to-use UI running on port 1337
- **API Integration**: GrumpyCats client library for automation

## Security Notice

**CRITICAL**: This tool is designed for isolated lab environments only!
- Deploy only in isolated, non-production systems
- Contains functionality to disable Windows Defender
- Handles potentially malicious files
- Never expose to production networks

## Requirements

### Ludus Host Requirements
- Ludus version 1.3.0 or higher
- Ansible 2.10 or higher
- Required Ansible Collections:
  - `ansible.windows` >= 2.0.0
  - `community.windows` >= 2.0.0
  - `chocolatey.chocolatey` >= 1.5.0

### Target VM Requirements
- **OS**: Windows 10/11, Server 2019/2022 (SERVER 2022 RECOMMENED FOR AV DISABLE)
- **RAM**: Minimum 4GB (8GB recommended)
- **Storage**: 10GB free disk space
- **Privileges**: Administrator access required
- **Network**: Internet access for initial setup

## Installation

### Quick Deploy to Ludus Range

```bash
# Install required collections on Ludus server
ansible-galaxy collection install ansible.windows community.windows chocolatey.chocolatey

# Add the role to your Ludus server
ludus range roles add ludus_litterbox_role

# Deploy to specific Windows VMs
ludus range deploy -t "litterbox" --limit "WS01,WS02"

# Or deploy to all Windows VMs
ludus range deploy -t "litterbox" --limit windows
```

### Manual Installation

```bash
# Clone the repository
git clone https://github.com/professor-moody/ludus_litterbox_role
cd ludus_litterbox_role

# Copy to your Ansible roles directory
cp -r . /etc/ansible/roles/ludus_litterbox_role/
```

## 🔧 Configuration

### Key Variables (defaults/main.yml)

```yaml
# Core Installation Settings
ludus_litterbox_install: true                              # Enable/disable installation
ludus_litterbox_install_dir: "C:\\Tools\\LitterBox"       # Installation directory
ludus_litterbox_python_version: "3.11.9"                  # Python version to install
ludus_litterbox_repo_url: "https://github.com/BlackSnufkin/LitterBox.git"

# Network Configuration
ludus_litterbox_host: "127.0.0.1"                         # Bind address (use 0.0.0.0 for network access)
ludus_litterbox_port: 1337                                # Web interface port

# Security Settings
ludus_litterbox_disable_defender: true                    # Disable Windows Defender (LAB USE ONLY!)
ludus_litterbox_defender_exclusions: true                 # Add AV exclusions
ludus_litterbox_require_admin: true                       # Require admin privileges

# UI/Access Settings
ludus_litterbox_firewall_rule: true                       # Create firewall exception
ludus_litterbox_desktop_shortcut: true                    # Create desktop shortcut

# Analysis Configuration
ludus_litterbox_analysis_timeout: 300                     # Analysis timeout (seconds)
ludus_litterbox_max_file_size: 104857600                 # Max file size (100MB)
ludus_litterbox_allowed_extensions:                       # Supported file types
  - exe, dll, sys, scr, com
  - bat, ps1, vbs, js
  - doc, docx, xls, xlsx, ppt, pptx, pdf, lnk

# Feature Flags
ludus_litterbox_enable_static: true                       # Static analysis
ludus_litterbox_enable_dynamic: true                      # Dynamic analysis
ludus_litterbox_enable_holygrail: true                   # BYOVD detection
ludus_litterbox_enable_doppelganger: true                # Process similarity
ludus_litterbox_enable_yara: true                        # YARA scanning

# System Settings
ludus_litterbox_install_chocolatey: true                  # Install Chocolatey
ludus_litterbox_install_python: true                      # Install Python if missing
ludus_litterbox_debug: false                              # Enable debug logging
```

## Example Playbooks

### Custom Configuration for Network Access

```yaml
---
- hosts: litterbox_servers
  vars:
    ludus_litterbox_host: "0.0.0.0"              # Listen on all interfaces
    ludus_litterbox_port: 8080                   # Custom port
    ludus_litterbox_install_dir: "D:\\Security\\LitterBox"
    ludus_litterbox_debug: true                  # Enable debug logging
    ludus_litterbox_analysis_timeout: 600        # 10 minute timeout
  roles:
    - role: ludus_litterbox
```

### Minimal Installation (Keep Defender Active)

```yaml
---
- hosts: analysis_workstations
  vars:
    ludus_litterbox_disable_defender: false      # Keep Defender enabled
    ludus_litterbox_defender_exclusions: false   # No AV exclusions
    ludus_litterbox_enable_dynamic: false        # Static analysis only
  roles:
    - role: ludus_litterbox
```

## 🖥️ Post-Installation Usage

### Accessing LitterBox

After successful deployment, access LitterBox through:

1. **Web Interface**: 
   - Local: `http://127.0.0.1:1337`
   - Network: `http://<VM_IP>:1337` (if configured for network access)

2. **Desktop Shortcut**: 
   - Double-click "LitterBox" on the desktop

3. **Command Line**:
   ```powershell
   # Using the start script
   C:\Tools\LitterBox\start_litterbox.bat
   
   # Manual start
   cd C:\Tools\LitterBox\LitterBox
   .\venv\Scripts\python.exe litterbox.py
   ```

### API Access with GrumpyCats

The GrumpyCats client library is automatically installed:

```python
# Example API usage
from grumpycat import GrumpyCat

client = GrumpyCat("http://127.0.0.1:1337")
result = client.analyze_file("suspicious.exe")
```

## Analysis Capabilities

### Static Analysis Features
- **YARA Scanning**: Signature-based detection
- **PE Analysis**: Import/Export tables, sections, resources
- **Document Analysis**: Macro extraction from Office files
- **String Extraction**: Unicode and ASCII strings
- **Entropy Analysis**: Packed/encrypted content detection
- **LNK Parsing**: Shortcut file analysis

### Dynamic Analysis Features
- **Process Monitoring**: Behavior tracking
- **Memory Inspection**: Runtime memory analysis
- **Code Injection Detection**: Hollowing and injection techniques
- **Sleep Pattern Analysis**: Evasion detection
- **ETW Telemetry**: Windows event tracing

### Specialized Scanners
- **HolyGrail**: Bring Your Own Vulnerable Driver (BYOVD) detection
- **Doppelganger**: Process impersonation analysis
- **FuzzyHash**: Similarity matching

## Maintenance

### Update LitterBox

```powershell
# Update to latest version
cd C:\Tools\LitterBox\LitterBox
git pull
.\venv\Scripts\pip.exe install -r requirements.txt --upgrade
```

### Re-run Ansible Role

```bash
# Force update on specific VM
ludus range deploy -t "litterbox" --limit "TARGET_VM" --extra-vars "ludus_litterbox_force_update=true"
```

### Clean Installation

```powershell
# Remove existing installation
Remove-Item -Path "C:\Tools\LitterBox" -Recurse -Force

# Re-deploy
ludus range deploy -t "litterbox" --limit "TARGET_VM"
```

## 🐛 Troubleshooting

### Common Issues and Solutions

| Issue | Solution |
|-------|----------|
| **Python Installation Fails** | Manually install Python 3.11+ from python.org, ensure PATH is set |
| **Git Clone Errors** | Check internet connectivity, verify proxy settings if applicable |
| **Port Already in Use** | Change `ludus_litterbox_port` variable or stop conflicting service |
| **Module Import Errors** | Re-run pip install: `.\venv\Scripts\pip.exe install -r requirements.txt` |
| **Firewall Blocking Access** | Verify firewall rule created, check Windows Firewall settings |
| **Insufficient Permissions** | Run Ansible with elevated privileges, check UAC settings |

### Enable Debug Mode

For detailed logging:

```yaml
ludus_litterbox_debug: true
ludus_litterbox_log_level: "DEBUG"
```

Check logs at: `C:\Tools\LitterBox\LitterBox\logs\`

### Verify Installation

```powershell
# Check Python
python --version

# Check git
git --version

# Check LitterBox directory
Test-Path "C:\Tools\LitterBox\LitterBox"

# Check virtual environment
Test-Path "C:\Tools\LitterBox\LitterBox\venv"

# Test Python packages
C:\Tools\LitterBox\LitterBox\venv\Scripts\python.exe -c "import flask; print('Flask OK')"
```

## 📁 Directory Structure

After installation, the following structure is created:

```
C:\Tools\LitterBox\
├── LitterBox\              # Main application directory
│   ├── venv\              # Python virtual environment
│   ├── config\            # Configuration files
│   ├── uploads\           # Uploaded samples
│   ├── results\           # Analysis results
│   ├── temp\              # Temporary files
│   ├── logs\              # Application logs
│   ├── database\          # Local database
│   ├── tools\             # Analysis tools
│   ├── Scanners\          # Scanner modules
│   ├── Utils\             # Utility scripts
│   └── GrumpyCats\        # API client library
└── start_litterbox.bat    # Startup script
```

## 🏷️ Tags

The role supports the following Ansible tags for selective execution:

- `litterbox` - Complete LitterBox installation
- `install` - Core installation tasks
- `chocolatey` - Chocolatey package manager
- `python` - Python installation
- `dependencies` - Required dependencies
- `tools` - Analysis tools setup
- `directories` - Directory creation
- `defender` - Windows Defender configuration (use with caution!)
- `dangerous` - High-risk operations

## 🤝 Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

1. Fork the repository
2. Create your feature branch (`git checkout -b feature/AmazingFeature`)
3. Commit your changes (`git commit -m 'Add some AmazingFeature'`)
4. Push to the branch (`git push origin feature/AmazingFeature`)
5. Open a Pull Request

## 📚 Additional Resources

- [LitterBox GitHub Repository](https://github.com/BlackSnufkin/LitterBox)
- [GrumpyCats API Client](https://github.com/BlackSnufkin/GrumpyCats)
- [Ludus Documentation](https://docs.ludus.cloud)
- [Ansible Windows Documentation](https://docs.ansible.com/ansible/latest/collections/ansible/windows/)
- [YARA Documentation](https://yara.readthedocs.io/)

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## ✨ Acknowledgments

- [BlackSnufkin](https://github.com/BlackSnufkin) - Original LitterBox author
- [Professor Moody](https://github.com/professor-moody) - Ludus role maintainer
- [Bad Sector Labs](https://github.com/badsectorlabs) - Ludus platform creators
- The security research community for continuous improvements

## ⚖️ Legal Disclaimer

**IMPORTANT**: This tool is intended for authorized security testing and research purposes only. Users are solely responsible for complying with all applicable laws and regulations in their jurisdiction. 

The authors and contributors:
- Assume no liability for misuse or damage caused by this software
- Do not condone or support illegal activities
- Recommend using this tool only in isolated lab environments
- Advise obtaining proper authorization before testing

By using this software, you acknowledge that you understand and agree to these terms.

---

**Role Version**: 1.0.0  
**LitterBox Compatibility**: Latest  
**Last Updated**: 2024  
**Maintained by**: Professor Moody
