***

```markdown
# Hitachi Vantara - SAP HANA Full Stack Deployment Playbooks

This repository contains a comprehensive, automated Ansible pipeline for deploying SAP HANA in Tailored Datacenter Integration (TDI) on Hitachi Advanced Server environments. It is specifically designed to work seamlessly with Hitachi VSP and VSP One Block storage systems. 

The automation handles everything from OS tuning and dynamic SAN LUN detection to SAP software downloads and unattended HANA database installations.

## Table of Contents
- [Overview](#-overview)
- [Architecture & Pipeline](#-architecture--pipeline)
- [Directory Structure](#-directory-structure)
- [Prerequisites & Dependencies](#-prerequisites--dependencies)
- [Configuration Guide](#-configuration-guide)
- [Execution Guide](#-execution-guide)

---

## Overview

Manual SAP HANA deployments involve complex storage formatting, strict OS kernel parameter tuning, and tedious SAP package management. This playbook pipeline fully automates these steps, ensuring consistent, repeatable, and SAP-certified installations. 

Key capabilities include:
* **Dynamic LUN Detection:** Automatically detects raw `dm-multipath` LUNs from Hitachi VSP arrays, classifying them by size into HANA Shared, Data, and Log volumes.
* **SAP Note Compliance:** Automatically applies required RedHat/SUSE OS configurations and kernel parameters based on official SAP Notes (e.g., 2002167, 2772999, 2292690).
* **Secure API Integration:** Prompts for SAP S-User credentials securely to download `SAPCAR` and HANA `SAR` media directly from the SAP Software Center.
* **Unattended Installation:** Executes `hdblcm` silently with secure vault integration for master passwords.

---

## Architecture & Pipeline

The execution is orchestrated by `main_pipeline.yml`, utilizing the `import_role` directive to apply tags dynamically. The pipeline consists of six distinct phases:

| Play / Tag | Included Role | Description |
| :--- | :--- | :--- |
| `01` / `prepare_python` | `sap_hana_python_config` | Bootstraps the RHEL/SLES environment with required Python 3.11 libraries (`beautifulsoup4`, `lxml`, `requests`) for SAP API interactions. |
| `02` / `hana_fs` | `sap_hana_filesystem_config` | Scans raw Hitachi LUNs, creates LVM Volume Groups, sets up striped Logical Volumes, and formats XFS filesystems according to TDI best practices. |
| `03` / `os_preconfig` | `sap_general_preconfigure` | Applies baseline OS settings (hostname, DNS, firewall, SELinux, limits.conf, uuidd). |
| `04` / `hana_preconfig` | `sap_hana_preconfigure` | Tunes the system specifically for HANA (CPU C-states, EPB, THP disable, NUMA balancing, TSX, and `sap-hana` tuned profiles). |
| `05` / `download_media` | `sap_hana_download_media` | Securely authenticates and downloads required SAP software packages to `/hana/shared/software`. |
| `06` / `hana_install` | `sap_hana_install` | Extracts SAR files via SAPCAR and executes the SAP HANA Database lifecycle manager (`hdblcm`). |

---

## Directory Structure

```text
sap-hana-full-stack-install/
├── ansible.cfg                           # Ansible runtime configuration
├── requirements.yml                      # Ansible Galaxy collection dependencies
├── main_pipeline.yml                     # Master execution playbook
├── inventory/
│   ├── hana_nodes.ini                    # Target host definitions
│   └── group_vars/
│       └── hana_nodes/                   
│           ├── vars.yml                  # Plaintext configuration variables
│           └── vault.yml                 # Encrypted secrets (Passwords)
├── playbooks/                            # Standalone playbooks for individual execution
│   ├── 01-sap_hana_python_config.yml
│   ├── 02-sap_hana_filesystem_config.yml
│   └── ...
└── roles/                                # Modular Ansible Roles
    ├── sap_general_preconfigure          # OS baseline tuning (SAP Notes)
    ├── sap_hana_preconfigure             # HANA specific kernel tuning
    ├── sap_hana_filesystem_config        # LVM/XFS storage automation on Hitachi LUNs
    ├── sap_hana_download_media           # SAP Launchpad integration
    ├── sap_hana_install                  # HANA hdblcm execution
    └── sap_hana_python_config            # Python dependencies
```

---

## Prerequisites & Dependencies

### 1. Control Node Requirements
* **Ansible Core:** 2.16 or higher
* **Python:** 3.11 or higher

Before running the pipeline, you **must install the required Ansible Galaxy collections** (which include the official `sap_install` and `sap_launchpad` dependencies).

Run the following command from the project root:
```bash
ansible-galaxy collection install -r requirements.yml
```

### 2. Managed Node Requirements
* **OS:** Red Hat Enterprise Linux (9.x) or SUSE Linux Enterprise Server (15.x).
* **Storage:** LUNs presented from Hitachi VSP arrays with `multipathd` active. **LUNs must be raw and unpartitioned.**
* **Credentials:** SAP S-User ID and Password (for media downloads). Root/Sudo access to target servers.

---

## Configuration Guide

### 1. Update Inventory as per your environment
Define your target HANA nodes in `inventory/hana_nodes.ini`:
```ini
[hana_nodes]
hananode01 ansible_host=192.168.1.100 ansible_user=root
```

### 2. Configure Variables as per your environment
Adjust your environment settings in `inventory/group_vars/hana_nodes/vars.yml`:
```yaml
# Target Storage Sizes (GiB) used for dynamic detection (Note: Modify the size as per your environment)
hana_shared_lun_size_gb: 1024
hana_data_lun_size_gb: 1700
hana_log_lun_size_gb: 128

# SAP HANA Installation details (Note: Modify the SID and Number as per your environment, and create master password in encrypted vault file)
sap_hana_install_sid: "HIT"
sap_hana_install_number: "00"
sap_hana_install_master_password: "{{ vault_hana_sidadm_password }}"

# Packages to download (Note: Update the SAP package name with correct version which required to download & install)
hana_softwarecenter_search_list:
  - 'SAPCAR_1400-70007716.EXE'
  - 'IMDB_SERVER20_088_0-80002031.SAR'
```

### 3. Secure Passwords
Store sensitive passwords in Ansible Vault. 
```bash
ansible-vault edit inventory/group_vars/hana_nodes/vault.yml --vault-password-file vault_pass.txt
OR
ansible-vault edit inventory/group_vars/hana_nodes/vault.yml

Note: You can add vault password in the file vault_pass.txt as plain text format, or provide password on the prompt when ask.
```
*Ensure `vault_hana_sidadm_password` and `ansible_ssh_pass` for SSH credentials are set in the vault password file.*

---

## Execution Guide

### Full End-to-End Installation
Execute the entire pipeline. The playbook will prompt you for your S-User ID and password before execution begins.
```bash
ansible-playbook -i inventory/hana_nodes.ini main_pipeline.yml --vault-password-file vault_pass.txt
OR
ansible-playbook -i inventory/hana_nodes.ini -J main_pipeline.yml -e "ansible_ssh_pass=Password"
```
*(Note: If the SAP media is already downloaded, simply press `Enter` at the S-User prompt to skip the download phase. If you don't want to use password file with --vault-password-file vault_pass.txt option, then can use -J, --ask-vault-password, --ask-vault-pass.).*

### Tagged Execution (Running Specific Stages)
You can utilize tags to run or skip specific portions of the pipeline.

**Run only the Filesystem Configuration:**
```bash
ansible-playbook -i inventory/hana_nodes.ini main_pipeline.yml --tags "hana_fs" --vault-password-file vault_pass.txt
```

**Run only the OS Pre-configuration (General + HANA):**
```bash
ansible-playbook -i inventory/hana_nodes.ini main_pipeline.yml --tags "os_preconfig,hana_preconfig" --vault-password-file vault_pass.txt
```

**Run only the HANA DB Installation:**
```bash
ansible-playbook -i inventory/hana_nodes.ini main_pipeline.yml --tags "hana_install" --vault-password-file vault_pass.txt
```
Skip Specific Stages (--skip-tags)
If a portion of your environment is already configured, you can execute the full pipeline but skip specific roles.

Skip the download and filesystem phases (Run everything else):

```bash
ansible-playbook -i inventory/hana_nodes.ini main_pipeline.yml --skip-tags "download_media,hana_fs" --vault-password-file vault_pass.txt
```

Skip the OS Tuning phases:

```bash
ansible-playbook -i inventory/hana_nodes.ini main_pipeline.yml --skip-tags "os_preconfig,hana_preconfig" --vault-password-file vault_pass.txt
```


### Run Playbooks Individually
If you prefer not to use the master pipeline, you can run the individual playbooks located in the `playbooks/` directory:
```bash
ansible-playbook -i inventory/hana_nodes.ini playbooks/02-sap_hana_filesystem_config.yml --vault-password-file vault_pass.txt
```

---

## Security & Best Practices
* **Log Suppression:** Sensitive module outputs (like SAP API calls containing passwords) are suppressed using `no_log: true`.
* **Validation Override:** The `ansible.cfg` is pre-configured with `validate_role_argument_spec = false` to allow seamless skipping of dynamically tagged imported roles without variable evaluation crashes.
* **Vault Security:** Never commit `vault_pass.txt` to version control. Add it to your `.gitignore`.

---
**Maintainer:** Hitachi Vantara SAP Solution Engineering Team  
**License:** Apache 2.0
```
