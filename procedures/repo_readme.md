# IT Support Playbooks

**Real-world troubleshooting guides from 10+ years supporting enterprise environments**

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Maintained: Yes](https://img.shields.io/badge/Maintained-Yes-green.svg)]()

## 📖 About This Repository

This repository contains structured troubleshooting playbooks developed from hands-on experience providing IT support across Fortune 500 companies, government agencies, and private clients. Each playbook follows a consistent format designed to reduce resolution time, improve documentation quality, and serve as training material for IT support teams.

These playbooks are actively maintained and reflect real-world scenarios encountered in enterprise environments with Active Directory, Windows workstations, network infrastructure, and standard business applications.

## 🎯 Who This Is For

- **Help Desk Technicians** seeking structured approaches to common issues
- **IT Support Teams** looking to standardize their troubleshooting procedures
- **New IT Professionals** building foundational support skills
- **IT Managers** developing training materials and knowledge bases
- **Anyone** interested in enterprise IT support best practices

## 📚 Table of Contents

### Active Directory
- [Account Lockout Response](playbooks/active-directory/account-lockout-response.md) - Diagnose and resolve AD account lockouts

### Networking
- [Remote Desktop Troubleshooting](playbooks/networking/remote-desktop-troubleshooting.md) - RDP connectivity issues *(Coming Soon)*
- [Network Connectivity Diagnosis](playbooks/networking/network-connectivity-diagnosis.md) - Basic network troubleshooting *(Coming Soon)*

### Hardware and Peripherals
- [Printer Troubleshooting](playbooks/hardware-and-peripherals/printer-troubleshooting.md) - Installation and common printer issues *(Coming Soon)*

### User Management
- [New User Onboarding](playbooks/user-management/new-user-onboarding.md) - Complete new user setup process *(Coming Soon)*
- [User Offboarding](playbooks/user-management/user-offboarding.md) - Secure account deactivation *(Coming Soon)*

### Workstation Support
- [New Workstation Setup](playbooks/workstation-support/new-workstation-setup.md) - Deployment checklist and procedures *(Coming Soon)*
- [Performance Issues](playbooks/workstation-support/performance-issues.md) - Diagnose slow computer problems *(Coming Soon)*

## 🚀 How to Use These Playbooks

### For Quick Reference
1. Navigate to the relevant category folder
2. Open the playbook markdown file
3. Follow the step-by-step resolution process
4. Use the verification checklist before closing the ticket

### For Training
1. Review the **Overview** section to understand scope and severity
2. Study the **Common Symptoms** to recognize issues quickly
3. Practice the **Resolution Steps** in a lab environment
4. Use the **Documentation Template** to build consistent ticket notes

### For Team Implementation
1. Clone or fork this repository to your organization's internal system
2. Customize playbooks with your specific tools, naming conventions, and escalation paths
3. Add your internal KB article links in the **Related Resources** section
4. Maintain version control as your environment changes

### Contributing Your Own Playbooks
Use the [playbook template](templates/playbook-template.md) to create new troubleshooting guides. The template includes all standard sections and ensures consistency across all playbooks.

## 📋 Playbook Structure

Each playbook follows this consistent format:

- **Overview** - Quick summary, severity level, and required access
- **Common Symptoms** - How users report the issue
- **Prerequisites** - Required tools and information
- **Resolution Steps** - Detailed troubleshooting procedures with commands
- **Verification** - Checklist to confirm resolution
- **Documentation Template** - Standardized ticket notes
- **Related Resources** - Links to related playbooks and documentation
- **Escalation Criteria** - When to involve higher-tier support
- **Notes and Tips** - Pro tips from real-world experience

## 🛠️ Tools Referenced

These playbooks commonly reference:

- **Active Directory Users and Computers (ADUC)**
- **PowerShell with AD Module**
- **Remote Desktop Protocol (RDP)**
- **Windows Event Viewer**
- **Command Prompt / PowerShell**
- **Network diagnostic tools** (ping, tracert, nslookup, ipconfig)
- **Standard ITSM platforms** (ServiceNow, Jira Service Desk, etc.)

## 🔄 Repository Status

**Current Status:** Active Development  
**Last Updated:** January 2025  
**Total Playbooks:** 1 (7 more in development)

### Roadmap
- ✅ Active Directory Account Lockout
- 🔄 Remote Desktop Troubleshooting (In Progress)
- 📝 Printer Troubleshooting (Planned)
- 📝 Network Connectivity Diagnosis (Planned)
- 📝 New User Onboarding (Planned)
- 📝 Workstation Deployment (Planned)
- 📝 Performance Issues (Planned)

## 📝 Contributing

Contributions, suggestions, and feedback are welcome! If you have:
- Improvements to existing playbooks
- New playbook ideas
- Corrections or updates
- Questions or clarifications needed

Please feel free to open an issue or submit a pull request.

## ⚖️ License

This repository is licensed under the MIT License - see the [LICENSE](LICENSE) file for details. You are free to use, modify, and distribute these playbooks for personal or commercial use.

## 📧 Contact

**Lisa Wicklein**  
Tampa, FL  
[LinkedIn](https://www.linkedin.com/in/lisawicklein/)  
[GitHub](https://github.com/LWicklein)

## 🙏 Acknowledgments

These playbooks are based on experience supporting diverse environments including:
- Fortune 500 corporations (General Dynamics, Johnson & Johnson, Merck-Medco)
- Government agencies (EEOC, Congressional Budget Office)
- Small business and individual clients
- Contract positions across enterprise organizations

---

**Note:** These playbooks represent general best practices and should be adapted to your organization's specific policies, tools, and security requirements. Always follow your organization's documented procedures and escalation paths.