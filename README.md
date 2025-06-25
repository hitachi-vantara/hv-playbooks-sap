**SAP HANA Storage Deplyment through Ansible Pipeline**

Hitachi Vantara developed a comprehensive pipeline encapsulated in **main.yml**, to automate the initial creation of SAP HANA scale-up volumes on the VSP Block storage system. These two pipelines have been created and thoroughly tested for all creation tasks:

**sap-hana-tdi-su-single-pool**: Designed for the latest VSPs

**sap-hana-tdi-su-two-pool**s: Designed for older VSPs

Above pipeline efficiently orchestrates the following tasks:

**Configure Storage Port**: Sets up the necessary storage ports on the Hitachi VSP system using 1_storage_port_create.yml

**Configure Hostgroups:** Establishes host groups and adds WWNs of hosts using 2_hostgroup_create.yml

**Parity Group Creation:** Creates parity groups with the required RAID type for data redundancy with 4_paritygroup_create.yml, after retrieving the parity group ids with 3_paritygroup_get_pg_id.yml

**Single or Two Storage Pool with Pool volumes Creation**: Create a storage pool with approx. 4 or 8 Pool Volumes (PVOLs) with 5_storagepool_create.yml

**VVOLs Creation and Mapping**: Provisions for Virtual Volumes (VVOLs) are made for operating system and SAP HANA components, including shared, log, and data volumes, utilizing the 6_vvol_creation_in_pool.yml configuration file. These volumes are then mapped to the host groups that contain the host WWNs. The creation process adheres to Hitachi Vantara's best practices for SAP HANA, ensuring optimal performance and reliability.

**Variables:** To manage variables effectively, we have utilized two below variables:

vsp_direct_variables_create.yml: A file containing global variables for the storage provisioning process, there would be a separate file for modification and deletion.

ansible_vault_storage_var.yml: An Ansible Vault file to securely store sensitive VSP credentials and connection information.

**Run the ansible playbook command to use pipeline:**

ansible-playbook main.yml -e "vsp_storage=e1090" -J

 -e EXTRA_VARS or --extra-vars EXTRA_VARS        set additional variables for the Storage which will be use

 -J or --ask-vault-password or--ask-vault-pass    ask for vault password


###################################################################################################

**DISCLAIMER: **
_All materials provided in this repository, including but not limited to Ansible Playbooks and Terraform Configurations, are made available as a courtesy. These materials are intended solely as examples, which may be utilized in whole or in part. Neither the contributors nor the users of this platform assert or are granted any ownership rights over the content shared herein. It is the sole responsibility of the user to evaluate the appropriateness and applicability of the materials for their specific use case.
 
Use of the material is at the sole risk of the user and the material is provided “AS IS,” without warranty, guarantees, or support of any kind, including, but not limited to, the implied warranties of merchantability, fitness for a particular purpose, and non-infringement. Unless specified in an applicable license, access to this material grants you no right or license, express or implied, statutorily or otherwise, under any patent, trade secret, copyright, or any other intellectual property right of Hitachi Vantara LLC (“HITACHI”). HITACHI reserves the right to change any material in this document, and any information and products on which this material is based, at any time, without notice. HITACHI shall have no responsibility or liability to any person or entity with respect to any damages, losses, or costs arising from the materials contained herein._
