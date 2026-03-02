***
# Hitachi Vantara VSP One BHE Block Storage - Storage Provisioning Playbooks for SAP HANA

This repository contains an automated Ansible pipeline for provisioning Hitachi VSP One BHE Block Storage for a SAP HANA Environment. The pipeline automates the end-to-end creation of HDP Storage Pool, Storage Ports, Server Profiles (Host Groups), and the provisioning and mapping of Data Reduction (DRS) Volumes for SAP HANA Tailored Datacenter Integration (TDI) layouts.

## Important Note Regarding Parity Groups

> **CRITICAL:** Parity group creation is not supported by the Ansible modules for VSP One BHE Block storage in the current version. 
> 
> You **must** create the Parity Groups (PG) manually using the RAIDCOM CLI or VSP360 before running this pipeline. Once PG created, add the correct Parity Group ID to the `vsp_direct_global_variables.yml` file under the `next_parity_group_id` variable.

---

## Table of Contents
- [Overview](#-overview)
- [Pipeline Architecture](#-architecture--pipeline)
- [Prerequisites](#-prerequisites)
- [Configuration Guide](#-configuration-guide)
- [Execution Guide](#-execution-guide)

---

## Overview

This automated workflow provisions storage for multiple SAP HANA nodes dynamically based on a CSV input file. It ensures consistent configuration of Host Mode Options and automated multipath mapping across multiple HBAs per server.

Key capabilities:
* **Dynamic Node Loading:** Reads hostnames, WWNs, and Storage Ports directly from `nodes_info.csv`.
* **Storage Validation:** Connects to the VSP One BHE array and validates the storage model and microcode before proceeding.
* **Automated Storage Pools:** Creates HDP Storage Pool and automatically assigns pool volumes (PVOLs).
* **Server Profile Creation:** Registers server profiles with multiple HBAs and paths dynamically.
* **HANA TDI Volume Provisioning:** Creates OS, HANA Shared, HANA Data (4x), and HANA Log (4x) volumes with Data Reduction (DRS) enabled, and maps them to their respective server profiles in the correct order.

---

## Pipeline Architecture

The execution is orchestrated by `main.yml`, which sequentially imports the following playbooks:

| Playbook | Description |
| :--- | :--- |
| `0_storagesystem_validate.yml` | Connects to the storage array, fetches facts, and validates that the model is a supported VSP One BHE storage. |
| `1_load_nodes.yml` | Parses `nodes_info.csv` into a structured Ansible dictionary list for dynamic looping in later tasks. |
| `2_storagepool_create.yml` | Creates the HDP Storage Pool using the predefined Parity Group ID and assigns LDEVs as Pool Volumes. |
| `3_vsp1_storage_port_create.yml` | Configures the SAN storage ports with the correct speed and fabric connection settings. |
| `4_vsp1_server_profile_create.yml` | Creates Server Profiles with Host Mode `LINUX` and Host Mode Options `68`, registering WWNs and mapping them to specific ports. |
| `5_vsp1_volume_creation_mapping.yml` | Provisions specific volumes for SAP HANA (OS, Shared, 4x Data, 4x Log) and maps them to the respective server profiles. |

---

## Prerequisites

* **Control Node:** Ansible installed with the `hitachi_vantara.vspone_block.vsp` collection.
* **Target Storage:** Hitachi VSP One BHE array reachable via management IP.
* **Parity Groups:** Pre-created Parity Groups on the storage array.
* **Credentials:** Storage administration credentials securely stored in an Ansible Vault file (`playbooks/ansible_vault_vars/ansible_vault_storage_var.yml`), if any other location/file then update in the playbooks.

---

## Configuration Guide

### 1. Update the Nodes Information CSV
Populate the `nodes_info.csv` file with your SAP HANA node details. The format strictly expects 5 columns without a header row:
```csv
hostname, port1, wwn1, port2, wwn2
```
*Example:*
```csv
hananode01,CL1-A,100000109BAA7A1B,CL2-A,100000109BAA7A1C
hananode02,CL3-A,100000109BAA7A2B,CL4-A,100000109BAA7A2C
```

### 2. Configure Global Variables
Edit `vsp_direct_global_variables.yml` to define your environment sizes and parameters. 

**Pay special attention to the Parity Group ID:**
```yaml
# Update the Parity Group ID created via RAIDCOM
next_parity_group_id: "1-1"

# Storage Pool Configuration
storage_pool_name_1: "ANSHAESPOOL8D"
storage_pool_size: "100GB" # Per PVOL size

# SAP HANA LDEV Sizing
os_vol_size: "100GB"
shared_vol_size: "1024GB"
log_vol_size: "128GB"      # Per VVOL size
data_vol_size: "256GB"     # Per VVOL size

# Port Configuration
port_speed: "32"
fabric_connection: true
```

### 3. Secure Storage Credentials
Ensure your vault file contains the connection information required by the `connection_info` dictionary.

```yaml
# playbooks/ansible_vault_vars/ansible_vault_storage_var.yml
storage_address: "192.168.1.50"
vault_storage_username: "admin"
vault_storage_secret: "YourSecurePassword!"
```

---

## Execution Guide

To execute the entire SAP HANA storage provisioning pipeline, run the main playbook from your control node and provide the vault password:

```bash
ansible-playbook main.yml --ask-vault-pass
```
*(Or point to your vault password file using `--vault-password-file vault_pass.txt`)*

### Validating the Run
1. The pipeline will first output the VSP One model, serial number, and microcode version.
2. It will parse the CSV and begin creating pools, configuring ports, and mapping host groups.
3. Upon successful completion, the `5_vsp1_volume_creation_mapping.yml` playbook will output a final debug map showing the exact LDEV IDs mapped to each respective server profile.

---
**Maintainer:** Hitachi Vantara SAP Solution Engineering Team  
**License:** Apache 2.0
**Reference:** https://galaxy.ansible.com/ui/repo/published/hitachivantara/vspone_block/docs/
```
