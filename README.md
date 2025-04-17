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
