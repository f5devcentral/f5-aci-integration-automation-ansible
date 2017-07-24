# f5-aci-integration-automation-ansible
F5 BIG-IP and Cisco ACI integration automation using Ansible

UNMANAGED MODE AUTOMATION (Configure Cisco ACI Network Policy Mode )

Common variable file which will be used by all the playbooks
The commands to be executed on the APIC are stored in the directory - unManagedMode_posts. 
These file are stored as templates which have variables, and these variables will be substituted to actual values from the variable file.
The substitution process is part of the ansible playbooks

Setup:
- vCMP hosts connected to the ACI fabric
- 1 vCMP guest on each vCMP host used in an HA pair

Pre-requisite:
- HA is setup beforehand on the BIG-IP
- Static VLAN pool is define on the APIC (The VLAN’s to be used are known beforehand)

Running the playbooks
Create_vlan.yaml (will be run on BIG-IP vCMP hosts) - ansible-playbook create_vlan.yaml 
- Configure provider and consumer VLAN’s on the BIG-IP vCMP host

Manual step moving VLANS between vCMP host and vCMP guest

Service_configuration.yaml (will be run on BIG-IP vCMP guests and the APIC) - ansible-playbook service_configuration.yaml 
- Create Unmanaged L4-L7 device
- Create Contract
- Create service graph template
- Create device selection policy
- Deploy service graph template
- Assign provided and consumed contract to the End Point Group (EPG) 
- Create BIG-IP network and application parameters

