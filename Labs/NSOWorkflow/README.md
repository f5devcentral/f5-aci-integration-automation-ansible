
# Service Model - BIG-IP and ACI configuration

>**Please note that the service model is for a particular deployment, customization will be needed before use**

## Pre-requisites ##

* Cisco ACI and BIG-IP NED's are installed on the NSO
* Authentication groups have been added for both the Cisco ACI and the F5 BIG-IP
* Cisco ACI device has been added and authenticated with the NSO
* F5 BIG-IP device has been added and authenticated with the NSO
* Both the devices are in sync with the NSO
* Service model package has been installed

For details on how to configure the NSO, refer to
https://www.cisco.com/c/en/us/support/cloud-systems-management/network-services-orchestrator-4-7/model.html

Once the setup is in place, you can login to the NSO and run the service model to conifgure ACI and BIG-IP. The [lab guide](https://github.com/f5devcentral/f5-aci-integration-automation-ansible/blob/master/Labs/NSOWorkflow/LabSteps.md) has details on how to configure the service model.

