# Hitachi Storage Provisioning Playbooks for SAP HANA

**Author:** Hitachi Vantara SAP Engineering Team  
**Version:** v1.0  
**Date:** November 2025

## Overview
This repository contains a collection of Ansible playbooks designed to automate the provisioning, management, and decommissioning of storage resources on Hitachi Virtual Storage Platform (VSP) systems, specifically tailored for SAP HANA environments.

The playbooks are organized into three main categories:
1.  **Modification & Expansion:** Tasks for expanding pools, resizing LDEVs, and configuring ports.
2.  **Deletion & Decommissioning:** Tasks for removing resources like LDEVs, hostgroups, and pools.
3.  **Custom Facts Gathering:** Tasks for retrieving detailed storage system information for reporting and audit purposes.

---

## 1. Modification & Expansion Playbooks
These playbooks are used to modify existing storage infrastructure.

| Playbook File | Summary |
| :--- | :--- |
| `modify_storage_pool_expand.yml` | Expands storage pool capacity by adding a new PVOL from an existing parity group and auto-assigns a sequential name based on existing volume naming convention. |
| `modify_ldev_resize.yml` | Expands the capacity of specified LDEVs to a new size and displays a summary of the resized volumes. |
| `modify_storage_port_config.yml` | Configures storage port settings including security, fabric mode, connection type, and port speed. |

**Configuration File:**
*   `vsp_direct_variables_modify.yml` - Central variable file containing configuration parameters for storage pool expansion, LDEV resize, and port configuration.

---

## 2. Deletion & Decommissioning Playbooks
**⚠️ WARNING:** These playbooks perform destructive actions. Use with caution.

| Playbook File | Summary |
| :--- | :--- |
| `delete_all_existing_hostgroups.yml` | Deletes ALL custom hostgroups from the storage system after unpresenting all associated LDEVs. Skips default system hostgroups (ending in `-G00`). |
| `delete_hostgroup.yml` | Deletes a specific hostgroup from multiple ports, forcibly removing all mapped LDEVs and releasing host reserves. |
| `delete_ldevs_by_name.yml` | Finds and forcibly deletes all LDEVs that match a specific name pattern (prefix). Includes unmapping from hostgroups and iSCSI targets before deletion. |
| `delete_ldevs_id_list.yml` | Forcibly deletes a specific list of LDEV IDs provided in a variable list. Handles unmapping from hostgroups automatically. |
| `delete_luns_from_host.yml` | Identifies LDEVs by name pattern and detaches (unpresents) them from their connected hostgroups without deleting the volumes. |
| `delete_paritygroup_id.yml` | Deletes a specific Parity Group by ID. |
| `delete_storage_pool_pg_by_name.yml` | Deletes storage pools matching a name pattern, including all their pool volumes, and subsequently deletes the associated parity groups. |
| `delete_vsp_one_all_servers.yml` | Retrieves all server profiles from VSP One and deletes them by ID while keeping LUN configuration (if specified). |

**Configuration File:**
*   `vsp_direct_variables_delete.yml` - Central variable file containing inputs for all delete operations, including pool names, LDEV lists, and naming patterns.

---

## 3. Custom Facts Gathering Playbooks
These playbooks are safe to run and provide detailed reporting on the storage system state.

| Playbook File | Summary |
| :--- | :--- |
| `get_ddp_pool_custom_facts.yml` | Retrieves and displays detailed summary information for all Dynamic Provisioning Pools (DDP), including capacity, RAID levels, drive details, and encryption status. |
| `get_ldevs_facts_name_keyword.yml` | Searches for LDEVs whose names start with a specific keyword (prefix) and displays their ID, Name, and Size. |
| `get_pvol_from_pool.yml` | Retrieves all LDEVs (PVOLs) belonging to storage pools that match a specific name pattern. Displays a summary of pool capacity and individual volume details. |
| `get_storage_pools_custom_facts.yml` | Retrieves detailed facts for all storage pools and displays a custom summary including Pool ID, Type, LDEV count, Status, and Capacity utilization metrics. |
| `hostgroup_custom_facts_all_ports.yml` | Retrieves information for ALL hostgroups on the system, displaying a formatted summary of Hostgroup Name, Port, WWNs, and mapped LUNs. |
| `hostgroup_facts_custom_ports.yml` | Retrieves hostgroup information for specific ports defined in variables, returning details like mapped WWNs and LDEVs. |
| `ldev_pool_id_custom_facts.yml` | Retrieves and lists all LDEVs (PVOLs) residing in a specific storage pool identified by its Pool ID. |
| `paritygroup_custom_facts.yml` | Retrieves and displays configuration facts for a single specified Parity Group ID. |
| `pvol-vvol_by_name_custom_facts.yml` | Filters and displays LDEVs (PVOLs/VVOLs) that contain a specific keyword in their name, showing ID, Name, Size, Parity Group, and Status. |
| `storagesystem_custom_facts.yml` | Retrieves general storage system information including Model, Serial Number, and Microcode version. |
| `vsp_one_volume_custom_facts.yml` | Retrieves volume facts for a specific DDP pool by name using the VSP One API, displaying a summary of LDEV IDs, names, sizes, and connection counts. |

**Configuration File:**
*   `vsp_direct_variables_facts.yml` - Central variable file for Fact Gathering playbooks, defining search patterns, pool IDs, and port configurations.

---

## Prerequisites
*   Ansible 2.9+
*   `hitachivantara.vspone_block` Ansible Collection
*   Network connectivity to Hitachi VSP Storage (REST API / VSP One API)
*   Valid credentials configured in `ansible_vault_storage_var.yml`

## Usage
1.  Update the relevant `variables_*.yml` file with your specific environment details (Pool IDs, Port names, etc.).
2.  Run the desired playbook using `ansible-playbook`:
    ```
    ansible-playbook modify_storage_pool_expand.yml --ask-vault-pass
    -J or --ask-vault-password or --ask-vault-pass ask for vault password, use this only if vault file is encrypted.
    ```
## Prerequisites
*   Red Hat Ansible Core - 2.16, 2.17, 2.18
*   Python - 3.9 or higher
*   Hitachi Virtual Storage Platform One Block Storage Modules for Red Hat Ansible 4.3 or later
*   Network connectivity to Hitachi VSP Storage
*   Valid credentials configured in `ansible_vault_storage_var.yml`



###################################################################################################

**DISCLAIMER: ** _All materials provided in this repository, including but not limited to Ansible Playbooks and Terraform Configurations, are made available as a courtesy. These materials are intended solely as examples, which may be utilized in whole or in part. Neither the contributors nor the users of this platform assert or are granted any ownership rights over the content shared herein. It is the sole responsibility of the user to evaluate the appropriateness and applicability of the materials for their specific use case.

Use of the material is at the sole risk of the user and the material is provided “AS IS,” without warranty, guarantees, or support of any kind, including, but not limited to, the implied warranties of merchantability, fitness for a particular purpose, and non-infringement. Unless specified in an applicable license, access to this material grants you no right or license, express or implied, statutorily or otherwise, under any patent, trade secret, copyright, or any other intellectual property right of Hitachi Vantara LLC (“HITACHI”). HITACHI reserves the right to change any material in this document, and any information and products on which this material is based, at any time, without notice. HITACHI shall have no responsibility or liability to any person or entity with respect to any damages, losses, or costs arising from the materials contained herein._
