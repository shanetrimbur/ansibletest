# Cisco IR1835 Router Automation

This project provides Ansible automation for configuring and hardening Cisco IR1835 Industrial Routers. The IR1835 is designed for industrial and edge computing environments where reliability and security are critical, such as:

- Remote industrial sites
- ICS (Industrial Control Systems) environments  
- Edge network locations with intermittent connectivity
- Industrial IoT deployments

## Project Structure

```
ansibletest/
├── ansible.cfg                 # Ansible configuration settings
├── ansible-navigator.yml       # Navigator execution environment config
├── inventory                   # Device inventory file
├── group_vars/
│   └── cisco.yml              # Shared variables for Cisco devices
├── host_vars/
│   └── ir1835.yml             # Host-specific credentials
└── playbooks/
    ├── get_facts.yml          # Fact gathering playbook
    └── cisco_basic_config.yml # Security configuration playbook
```

## How It Works

### Ansible Overview
Ansible is an agentless automation platform that uses SSH to configure network devices. Key concepts in this project:

- **Inventory**: Defines target devices and groups
- **Playbooks**: YAML files containing configuration tasks
- **Variables**: Stored in group_vars and host_vars
- **Modules**: Pre-built code for specific tasks (e.g., cisco.ios.ios_config)
- **Facts**: Device information gathered by Ansible

### Key Components

#### ansible-navigator.yml
```yaml
ansible-navigator:
  ansible:
    inventory:
      entries:
      - inventory
  mode: stdout
  execution-environment:
    container-engine: podman
    enabled: true
    image: registry.redhat.io/ansible-automation-platform-23/ee-supported-rhel8:latest
```
This configures the execution environment using Red Hat's container image, ensuring consistent automation across different control nodes.

#### inventory
```ini
[cisco]
ir1835  ansible_host=192.168.1.90
```
Defines the target Cisco device with its management IP address.

#### group_vars/cisco.yml
```yaml
ansible_connection: ansible.netcommon.network_cli
ansible_network_os: cisco.ios.ios
```
Specifies connection parameters for all Cisco devices:
- Uses network_cli connection plugin
- Sets IOS as the network operating system

#### host_vars/ir1835.yml
```yaml
ansible_user: ansible
ansible_password: PASSW0RD
```
Contains device-specific credentials (should be encrypted in production).

### Playbooks

#### get_facts.yml
```yaml
- name: Get Facts
  hosts: cisco
  gather_facts: false
  tasks:
    - name: Gather legacy and resource facts
      cisco.ios.ios_facts:
        gather_subset: all
        gather_network_resources: all
```
This playbook:
- Targets all devices in 'cisco' group
- Disables default fact gathering
- Uses cisco.ios.ios_facts module to collect device information

#### cisco_basic_config.yml
```yaml
- name: Get Facts
  hosts: cisco
  gather_facts: false
  vars:
    acl: true
  tasks:
    - name: Configure http timeout policy
      cisco.ios.ios_config:
        lines:
          - ip http timeout-policy idle 300 life 3000 requests 30
          - ip ssh server algorithm mac hmac-sha2-256
          # ... additional security configurations
```
Implements security hardening:
- HTTP timeout policies
- SSH security algorithms
- Login restrictions
- Console settings
- SNMP configuration
- Access Control Lists

### Execution Flow

1. **Preparation**:
   - Ansible loads inventory and variables
   - Establishes SSH connections to devices
   - Validates authentication

2. **Fact Gathering**:
   - Collects device information
   - Stores facts for use in tasks
   - Results saved in JSON artifacts

3. **Configuration**:
   - Applies security configurations
   - Validates changes
   - Generates execution logs

### Artifacts and Logging

The `playbook-artifacts` directory contains JSON files with detailed execution results:
```json
{
    "plays": [
        {
            "play_pattern": "cisco",
            "tasks": [
                {
                    "task": "Gather legacy and resource facts",
                    "result": "Ok",
                    "duration": "21s"
                }
            ]
        }
    ]
}
```
These artifacts provide:
- Execution timestamps
- Task results
- Duration metrics
- Device configurations
- Error messages

## Usage

1. **Setup Environment**:
   ```bash
   # Install required collections
   ansible-galaxy collection install cisco.ios

   # Configure execution environment
   podman pull registry.redhat.io/ansible-automation-platform-23/ee-supported-rhel8:latest
   ```

2. **Configure Credentials**:
   - Update host_vars/ir1835.yml with device credentials
   - Consider using ansible-vault for encryption

3. **Run Playbooks**:
   ```bash
   # Gather facts
   ansible-navigator run get_facts.yml

   # Apply security configuration
   ansible-navigator run cisco_basic_config.yml
   ```

## Security Considerations

- Use ansible-vault for sensitive data
- Implement proper ACL configurations
- Regular security audits
- Monitor configuration changes
- Use secure protocols (SSH vs Telnet)

## Troubleshooting

Common issues and solutions:
- Authentication failures: Check credentials in host_vars
- Connection timeouts: Verify network connectivity and SSH access
- Module errors: Ensure correct Ansible collections are installed
- Execution environment issues: Verify podman/container setup

## Contributing

1. Fork the repository
2. Create a feature branch
3. Submit pull request with detailed description
4. Include relevant playbook artifacts

## License

MIT License

Copyright (c) 2024

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
