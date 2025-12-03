# Hitachi Storage Provisioning Pipelines for SAP HANA

**Author:** Hitachi Vantara SAP Engineering Team  
**Date:** November 2025  
**Tested on:** Hitachi Virtual Storage Platform One Block Storage Modules for Red Hat Ansible 4.4.2

## Overview
This repository contains automated Ansible pipelines for provisioning and managing Hitachi VSP storage for SAP HANA environments. The pipelines cover both **Scale-Out** and **Scale-Up** deployment models across different VSP storage families (VSP E/F/G Series and VSP One).

## Available Pipelines

### 1. SAP HANA Scale-Out (VSP E/F/G Series)
**Directory:** `sap-hana-scale-out-vsp-efg`  
**Version:** v1.0  
**Description:** Automated storage provisioning for SAP HANA Scale-Out servers using a **Two HDP Storage Pool** method (Data & Log separation).

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

### 2. SAP HANA Scale-Out (VSP One)
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

### 3. SAP HANA Scale-Up (VSP E/F/G Series - Single Pool)
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

### 4. SAP HANA Scale-Up (VSP E/F/G Series - Two Pools)
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

## Configuration Files

Each pipeline directory contains specific configuration files that must be updated before execution:

*   **`vsp_direct_global_variables.yml`**:
    *   Contains global variables specific to the selected pipeline (e.g., Pool Names, RAID levels, LDEV sizing).
    *   *Note: Each pipeline has its own version of this file.*

*   **`nodes_info.csv`** (Scale-Out Pipelines only):
    *   A CSV file containing details for the SAP HANA nodes.
    *   **Format:** `Hostname,Storage_Port_1,Storage_Port_2,WWPN_1,WWPN_2`

## Prerequisites
*   Red Hat Ansible Core - 2.16, 2.17, 2.18
*   Python - 3.9 or higher
*   Hitachi Virtual Storage Platform One Block Storage Modules for Red Hat Ansible 4.3 or later
*   Network connectivity to Hitachi VSP Storage
*   Valid credentials configured in `ansible_vault_storage_var.yml`


## Usage
Run the ansible playbook command to use pipeline:

**`ansible-playbook -J main.yml`**

-J or --ask-vault-password or --ask-vault-pass ask for vault password, use this only if vault file is encrypted.

###################################################################################################

**DISCLAIMER: ** _All materials provided in this repository, including but not limited to Ansible Playbooks and Terraform Configurations, are made available as a courtesy. These materials are intended solely as examples, which may be utilized in whole or in part. Neither the contributors nor the users of this platform assert or are granted any ownership rights over the content shared herein. It is the sole responsibility of the user to evaluate the appropriateness and applicability of the materials for their specific use case.

Use of the material is at the sole risk of the user and the material is provided “AS IS,” without warranty, guarantees, or support of any kind, including, but not limited to, the implied warranties of merchantability, fitness for a particular purpose, and non-infringement. Unless specified in an applicable license, access to this material grants you no right or license, express or implied, statutorily or otherwise, under any patent, trade secret, copyright, or any other intellectual property right of Hitachi Vantara LLC (“HITACHI”). HITACHI reserves the right to change any material in this document, and any information and products on which this material is based, at any time, without notice. HITACHI shall have no responsibility or liability to any person or entity with respect to any damages, losses, or costs arising from the materials contained herein._
