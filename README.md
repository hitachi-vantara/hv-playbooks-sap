# Hitachi Storage Provisioning Pipelines for SAP HANA

**Author:** Hitachi Vantara SAP Engineering Team  
**Date:** November 2025  
**Tested on:** Hitachi Virtual Storage Platform One Block Storage Modules for Red Hat Ansible 4.4.2

## Overview
This repository contains a collection of Ansible playbooks designed to automate the provisioning, management, and decommissioning of storage resources on Hitachi Virtual Storage Platform (VSP) systems, specifically tailored for SAP HANA environments.

The repository includes:
1.  **End-to-End Pipelines:** Fully automated workflows for setting up new SAP HANA environments.
2.  **Individual Task Playbooks:** Standalone playbooks for performing specific, granular tasks like creating a single LDEV or deleting a hostgroup.

---

## 1. End-to-End Pipelines
These are fully automated workflows for setting up new SAP HANA environments from start to finish.

### 1.1. SAP HANA Scale-Out (VSP E/F/G Series - Two Pools)
**Directory:** `sap-hana-scale-out-vsp-efg`  
**Version:** v1.0  
**Description:** Automated storage provisioning for SAP HANA Scale-Out servers using a **Two HDP Storage Pool** method (Data & Log volumes separation).

**Workflow:**
1.  `0_storagesystem_validate.yml`: Validates VSP system support.
2.  `1_load_nodes.yml`: Loads SAP HANA node details from `nodes_info.csv`.
3.  `2_storage_port_create.yml`: Configures VSP storage ports.
4.  `3_hostgroup_add-wwn_create.yml`: Creates hostgroups and registers WWNs.
5.  `4_paritygroup_get_pg_id.yml`: Checks existing Parity Groups and determines IDs.
6.  `5_paritygroup_create.yml`: Creates Parity Groups for storage pools.
7.  `6_storagepool_create.yml`: Creates two distinct storage pools (Data and Log).
8.  `7_vvol_creation_mapping.yml`: Creates required VVols and maps them to hostgroups.

---

### 1.2. SAP HANA Scale-Out (VSP One Block Storage)
**Directory:** `sap-hana-scale-out-vsp-one`  
**Version:** v1.0  
**Description:** Automated storage provisioning for SAP HANA Scale-Out servers using a **Single DDP Storage Pool** method on VSP One Block Storage.

**Workflow:**
1.  `0_storagesystem_validate.yml`: Validates VSP One system support.
2.  `1_disk_drives_fact.yml`: Checks for required free disk drives.
3.  `2_ddp_pool_create.yml`: Creates a Dynamic Provisioning (DDP) Pool.
4.  `3_load_nodes.yml`: Loads SAP HANA node details from `nodes_info.csv`.
5.  `4_vsp_one_storage_port_create.yml`: Configures storage ports.
6.  `5_vsp_one_server_profile_create_so.yml`: Creates server profiles, adds WWNs, and connects ports.
7.  `6_vsp_one_volume_creation_mapping.yml`: Creates volumes in the DDP pool and attaches them to server profiles.

---

### 1.3. SAP HANA Scale-Up (VSP E/F/G Series - Single Pool)
**Directory:** `sap-hana-scale-up-vsp-efg-single-pool`  
**Version:** v1.1  
**Description:** Automated storage provisioning for a SAP HANA Scale-Up server using a **Single HDP Storage Pool**.

**Workflow:**
1.  `0_storagesystem_validate.yml`: Validates VSP system support.
2.  `1_storage_port_create.yml`: Configures storage ports.
3.  `2_hostgroup_create.yml`: Creates hostgroups and registers WWNs.
4.  `3_paritygroup_get_pg_id.yml`: Determines Parity Group IDs.
5.  `4_paritygroup_create.yml`: Creates Parity Group with required RAID type.
6.  `5_storagepool_create.yml`: Creates a single HDP storage pool.
7.  `6_vvol_creation_in_pool.yml`: Creates Virtual Volumes and maps LUNs to the hostgroup.

---

### 1.4. SAP HANA Scale-Up (VSP E/F/G Series - Two Pools)
**Directory:** `sap-hana-scale-up-vsp-efg-two-pools`  
**Version:** v1.1  
**Description:** Automated storage provisioning for a SAP HANA Scale-Up server using **Two HDP Storage Pools** (Data & Log separation).

**Workflow:**
1.  `0_storagesystem_validate.yml`: Validates VSP system support.
2.  `1_storage_port_create.yml`: Configures storage ports.
3.  `2_hostgroup_create.yml`: Creates hostgroups and registers WWNs.
4.  `3_paritygroup_get_pg_id.yml`: Determines Parity Group IDs.
5.  `4_paritygroup_create.yml`: Creates Parity Groups with required RAID type.
6.  `5_storagepool_create.yml`: Creates two separate HDP storage pools.
7.  `6_vvol_creation_in_pool.yml`: Creates Virtual Volumes in respective pools and maps LUNs.

---

## 2. Individual Task Playbooks
**Directory:** `individual_playbooks_for_tasks/`

This folder contains standalone playbooks for performing specific, granular tasks outside of the main provisioning pipelines. They are ideal for one-off operations, troubleshooting, or reporting.

### 2.1. Modification Playbooks
| Playbook File | Summary |
| :--- | :--- |
| `modify_storage_pool_expand.yml` | Expands storage pool capacity by adding a new PVOL from an existing parity group and auto-assigns a sequential name based on existing volume naming convention. |
| `modify_ldev_resize.yml` | Expands the capacity of specified LDEVs to a new size and displays a summary of the resized volumes. |
| `modify_storage_port_config.yml` | Configures storage port settings including security, fabric mode, connection type, and port speed. |

**Configuration:** Use `vsp_direct_variables_modify.yml` to specify targets.

### 2.2. Deletion Playbooks
**⚠️ WARNING:** These playbooks perform destructive actions. Use with caution.
| Playbook File | Summary |
| :--- | :--- |
| `delete_all_existing_hostgroups.yml` | Deletes ALL custom hostgroups from the storage system after unpresenting all associated LDEVs. |
| `delete_hostgroup.yml` | Deletes a specific hostgroup from multiple ports, forcibly removing all mapped LDEVs. |
| `delete_ldevs_by_name.yml` | Finds and forcibly deletes all LDEVs that match a specific name pattern (prefix). |
| `delete_ldevs_id_list.yml` | Forcibly deletes a specific list of LDEV IDs provided in a variable list. |
| `delete_luns_from_host.yml` | Identifies LDEVs by name pattern and detaches (unpresents) them from their connected hostgroups. |
| `delete_paritygroup_id.yml` | Deletes a specific Parity Group by ID. |
| `delete_storage_pool_pg_by_name.yml` | Deletes storage pools matching a name pattern, including all their pool volumes, and subsequently deletes the associated parity groups. |
| `delete_vsp_one_all_servers.yml` | Retrieves all server profiles from VSP One and deletes them by ID. |

**Configuration:** Use `vsp_direct_variables_delete.yml` to specify targets.

### 2.3. Custom Facts Gathering Playbooks
| Playbook File | Summary |
| :--- | :--- |
| `get_ddp_pool_custom_facts.yml` | Retrieves and displays a detailed summary for all Dynamic Provisioning Pools (DDP). |
| `get_ldevs_facts_name_keyword.yml` | Searches for LDEVs whose names start with a specific keyword (prefix) and displays their details. |
| `get_pvol_from_pool.yml` | Retrieves all LDEVs (PVOLs) belonging to storage pools that match a specific name pattern. |
| `get_storage_pools_custom_facts.yml` | Retrieves detailed facts for all storage pools and displays a custom summary. |
| `hostgroup_custom_facts_all_ports.yml` | Retrieves information for ALL hostgroups on the system, displaying a summary of Name, Port, WWNs, and LUNs. |
| `hostgroup_facts_custom_ports.yml` | Retrieves hostgroup information for specific ports defined in variables. |
| `ldev_pool_id_custom_facts.yml` | Retrieves and lists all LDEVs (PVOLs) residing in a specific storage pool identified by its Pool ID. |
| `paritygroup_custom_facts.yml` | Retrieves and displays configuration facts for a single specified Parity Group ID. |
| `pvol-vvol_by_name_custom_facts.yml` | Filters and displays LDEVs (PVOLs/VVOLs) that contain a specific keyword in their name. |
| `storagesystem_custom_facts.yml` | Retrieves general storage system information including Model, Serial Number, and Microcode version. |
| `get_ddp_pool_custom_facts.yml` | Retrieves volume facts for a specific DDP pool by name using the VSP One API. |

**Configuration:** Use `vsp_direct_variables_facts.yml` to specify targets.

---

## Configuration Files

Each pipeline and task folder contains specific configuration files that must be updated before execution:

*   **`vsp_direct_global_variables.yml`**:
    *   Contains global variables specific to the selected pipeline (e.g., Pool Names, RAID levels, LDEV sizing).

*   **`nodes_info.csv`** (Scale-Out Pipelines only):
    *   A CSV file containing details for the SAP HANA nodes.
    *   **Format:** `Hostname,Storage_Port_1,Storage_Port_2,WWPN_1,WWPN_2`

*   **`vsp_direct_variables_*.yml`** (Individual Tasks):
    *   Variable files within the `individual_playbooks_for_tasks` directory used to define targets and parameters for modify, delete, or fact-gathering operations.


## Prerequisites
*   Red Hat Ansible Core - 2.16, 2.17, 2.18
*   Python - 3.9 or higher
*   Hitachi Virtual Storage Platform One Block Storage Modules for Red Hat Ansible 4.3 or later
*   Network connectivity to Hitachi VSP Storage
*   Valid credentials configured in `ansible_vault_storage_var.yml`


## Usage
1.  Update the relevant `.yml` and `.csv` configuration files with your specific environment details.
2.  Run the desired playbook using `ansible-playbook`, providing the vault password if necessary:
    ```
    ansible-playbook sap-hana-scale-out-vsp-efg/main.yml --ask-vault-pass
    ```
    ```
    ansible-playbook individual_playbooks_for_tasks/modify/modify_ldev_resize.yml --ask-vault-pass
    ```
    Option: -J or --ask-vault-password or --ask-vault-pass ask for vault password, use this only if vault file is encrypted.

###################################################################################################

**DISCLAIMER: ** _All materials provided in this repository, including but not limited to Ansible Playbooks and Terraform Configurations, are made available as a courtesy. These materials are intended solely as examples, which may be utilized in whole or in part. Neither the contributors nor the users of this platform assert or are granted any ownership rights over the content shared herein. It is the sole responsibility of the user to evaluate the appropriateness and applicability of the materials for their specific use case.

Use of the material is at the sole risk of the user and the material is provided “AS IS,” without warranty, guarantees, or support of any kind, including, but not limited to, the implied warranties of merchantability, fitness for a particular purpose, and non-infringement. Unless specified in an applicable license, access to this material grants you no right or license, express or implied, statutorily or otherwise, under any patent, trade secret, copyright, or any other intellectual property right of Hitachi Vantara LLC (“HITACHI”). HITACHI reserves the right to change any material in this document, and any information and products on which this material is based, at any time, without notice. HITACHI shall have no responsibility or liability to any person or entity with respect to any damages, losses, or costs arising from the materials contained herein._
