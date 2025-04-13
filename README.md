# Ansible Apache Deployment Project

**(Generated on Sunday, April 13, 2025 at 10:07 AM IST)**

This Ansible project automates the deployment and basic configuration of the Apache web server (`apache2` on Debian/Ubuntu, `httpd` on RHEL/CentOS) on a group of Linux servers.

## Features

* Installs the appropriate Apache package based on the OS family.
* Creates a custom web root directory (default: `/srv/www/mywebapp`).
* Deploys a sample `index.html` to the web root.
* Configures Apache to listen on a custom port defined by the `apache_listen_port` variable (e.g., 3030).
* Removes the default `Listen 80` directive from the main Apache configuration.
* Configures the default Apache VirtualHost to use the custom port and web root directory.
* Includes necessary `<Directory>` directives with `Require all granted` for Apache 2.4+ to ensure content is accessible.
* Enables the default site configuration (Debian/Ubuntu specific).
* Ensures the Apache service is enabled on boot and currently running.
* Opens the custom `apache_listen_port` in the firewall (`ufw` for Debian/Ubuntu, `firewalld` for RHEL/CentOS).
* Uses an Ansible Role (`apache`) for modularity.
* Uses group variables for easy configuration.
* Designed to be idempotent - can be run multiple times safely.

## Requirements

* **Control Node:** Ansible installed.
* **Managed Nodes:**
    * Linux distribution from Debian family (Ubuntu, Debian) or RedHat family (CentOS, RHEL, Rocky, Fedora).
    * SSH server enabled.
    * Python 2.7 or Python 3.x installed (usually present by default).
    * A user account that Ansible can connect as (via `ansible_user`).
    * Passwordless `sudo` configured for the `ansible_user` to allow privilege escalation (`become: yes`).
* **Network:** Control node must have SSH access to all managed nodes on port 22 (or configured SSH port). Managed nodes need internet access to download packages.
* **SSH Keys:** Key-based SSH authentication from the control node to managed nodes is highly recommended.
* **Ansible Collections (for Firewall):** If not already installed, you might need:
    ```bash
    ansible-galaxy collection install community.general ansible.posix
    ```

## Project Structure

```text
ansible-project/
├── inventories/
│   └── production/
│       ├── group_vars/
│       │   └── webservers.yml       # Define variables (e.g., apache_listen_port)
│       └── hosts.ini               # Define hosts and connection details
├── roles/
│   └── apache/                     # Apache configuration role
│       ├── tasks/
│       │   └── main.yml            # Main tasks for Apache setup
│       ├── handlers/
│       │   └── main.yml            # Handlers (e.g., restart apache)
│       ├── templates/
│       │   └── apache-vhost.conf.j2 # Template for the vhost config
│       ├── files/
│       │   └── index.html          # Sample web page file
│       └── vars/
│           └── main.yml            # Role vars (package names, paths)
├── site.yml                        # Main playbook orchestrating roles & firewall
└── README.md                       # This file
## Configuration

1.  **Inventory (`inventories/production/hosts.ini`):**
    * Define your managed nodes under a group (e.g., `[webservers]`).
    * Specify connection details for each host, especially if usernames or IPs differ:
        ```ini
        [webservers]
        control-node      ansible_host=IP_OR_HOSTNAME_1    ansible_user=your_user_1
        node1             ansible_host=IP_OR_HOSTNAME_2    ansible_user=your_user_2
        node2             ansible_host=IP_OR_HOSTNAME_3    ansible_user=your_user_3
        ```

2.  **Group Variables (`inventories/production/group_vars/webservers.yml`):**
    * Create this file if it doesn't exist.
    * Define key variables here. **Crucially, set your desired Apache port.**
        ```yaml
        ---
        # Variables for the webservers group
        apache_listen_port: 3030  # <--- CHANGE TO YOUR DESIRED PORT
        # web_root_dir: /srv/www/mywebapp # Optional: Override default if needed
        ```

## Usage

Ensure you are in the `ansible-project` directory on your control node.

1.  **Syntax Check:** Verify the playbook syntax before running.
    ```bash
    ansible-playbook -i inventories/production/hosts.ini site.yml --syntax-check
    ```

2.  **Dry Run (Check Mode):** See what changes Ansible *would* make without actually changing anything.
    ```bash
    ansible-playbook -i inventories/production/hosts.ini site.yml --check
    ```

3.  **Execute Playbook:** Apply the configuration to your servers.
    ```bash
    ansible-playbook -i inventories/production/hosts.ini site.yml
    ```
    *(Note: If you haven't configured passwordless sudo, you might need the `-K` or `--ask-become-pass` flag, but passwordless sudo is recommended).*

## Verification

After a successful playbook run:

1.  **Check Service Status (on a managed node):**
    * Debian/Ubuntu: `sudo systemctl status apache2.service`
    * RHEL/CentOS: `sudo systemctl status httpd.service`
    * It should show "active (running)".

2.  **Check Web Page Access (from control node or your machine):**
    * Replace `<managed_node_ip>` with the actual IP and `<port>` with the value of `apache_listen_port`.
    * `curl http://<managed_node_ip>:<port>`
    * Example: `curl http://10.103.20.81:3030`
    * You should see the HTML output of the `index.html` file ("Hello from Ansible!"). You can also test in a web browser.

---
