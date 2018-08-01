
# Lab Exercise 

>**Please note that the images used in the lab guide are representative and NOT based on any specific pod. Please use the information in the lab guide instead.**

## Getting started ##

Cisco Application Centric Infrastructure (ACI) technology provides the capability to insert Layer 4 through Layer 7 (L4-L7) functions using an approach called a service graph. The servive graph controls network connectivity consisting of VLANs.

Lab will be used to demonstrate L4-L7 service insertion in unmanaged mode to simulate an enterprise network and/or cloud provider’s application delivery offering while allowing the application owner to manage the L4-L7 device using Ansible. 

The topology used for this Lab is as follows:

**IMAGE**

The goal is to provide a central point of control to configure both the Cisco APIC as well as the F5 BIG-IP. Network stitching is achieved by automating the deployment of a service graph on APIC and L4-L7 configuration is automated directly on the BIG-IP

**IMAGE**

**For this lab**
* We will use the F5 BIG-IP VE Virtual ADC to demonstrate this functionality
* The service graph will be deployed in One-ARM mode which implies **one** interface will be consumed on the BIG-IP which handles the client as well as server traffic
* A default route will be configured on the BIG-IP
* SNAT = 'Automap' will be configured on the BIG-IP (return traffic from backend servers are forced to pass back through the BIG-IP)
* The EPG's used are **web-epg**(provider) and **epg-l3out** (consumer). Bridge domain **vip-bd**

**IMAGE**

**Let's begin the lab**

## Automate configuration on APIC and BIG-IP using Cisco NSO

We will provision the APIC and BIG-IP using NSO.

Connect to NSO using the following information:

* **NSO Address**: 172.21.208.249
* **username**: admin
* **password**: C1sc0123

Once you are logged in click on **Device Manager** located on the dashboard

![](images/NSO-dm.png)

Before procedding we are going to sync NSO to have the latest APIC and BIG-IP configuration.  
Select both the devices using the checkbox, click on the **running man icon** and click on **sync-from**

![](images/NSO-dm1.png)

While the device is syncing a yellow tab will show up next to the device.

![](images/NSO-dm2.png)

Once the devices are synced, there will be green tabs with **yes** for **found**, **connected** and **in-sync**

![](images/NSO-dm3.png)

Click on the **Cisco** icon on the top left corner to go back to the dashboard

Click on **Configuration Editor** located on the dashboard.  

![](images/NSO-cm.png)

Click on **ncs:services**

![](images/NSO-cm1.png)

Click on **+** sign next to **aci-bigip:aci-bigip** service

![](images/NSO-cm2.png)

There will will a pop up, this is the name of the service enter **studentxx-demo** where **xx** is your student pod.Example 'student01-demo' and click on **confirm**

![](images/NSO-sm.png)

This will take you back to the services page, click on the service created **studentxx-demo**

![](images/NSO-sm1.png)

Under the section **vlans/**, click the **+** sign

![](images/NSO-sm-vlan.png)

There will be a pop up, enter the name of the VLAN **vlan** and click **confirm**

![](images/NSO-sm-vlan1.png)

Click on the **vlan** created

![](images/NSO-sm-vlan2.png)

* Enter the vlan tag **1234**.  
* Click on **studentxx-demo** on the link ncs:services/aci-bigip:aci-bigip{**studentxx-demo**}/vlans{vlan}/ to go back to the service added

![](images/NSO-sm-vlan3.png)

Under section **/sdn-controller**, from the drop down list choose **cisco-apic**. This information is automatically getting pulled from the NSO. This drop down is a list of all devices that are of type 'Cisco-ACI'. In our lab we have only one device configured on the NSO hence only one device in the drop down menu.

Under section **/sdn-controller/tenant/**, fill in the following
* name (drop down list) - **studentxx** -> your student pod
* application-profile-name (drop down list) - **app**
* epg-provider-name(drop down list) - **web-epg**
* epg-consumer-name (drop down list) - **epg-l3out**
* bd-provider-name (drop down list) - **vip-bd** 
* bd-comsumer-name (drop down list) - **vip-bd**
* contract-name (type it in) - **cntr**

**IMAGE**

Under section **/sdn-controller/tenant/vns-ldev-vip**, fill in the following
* name - **bigip**
* device-type - leave it as **VIRTUAL**
* domain-name (drop down list) - **CLBerlin2016**

**IMAGE**

Under section **/sdn-controller/tenant/vns-abs-graph**, fill in the following
* name - **sgt**
* template-type - leave it as **ADC-ONE-ARM**

Under section **/load-balancer** from the drop down list choose **f5-bigip**

**IMAGE**

Under section **/load-balancer-vlans/**, click on the '+' sign

A pop up window will appear, from the drop down list choose **vlan** and click **confirm**

**IMAGE*

Click on the **vlan** created

**IMAGE**

Under section **/interfaces** click on the **+** sign

**IMAGE**

A pop up window will appear, choose interface **1.1** from the drop down list and click **confirm**

**IMAGE**

Click on the iterface name.

**IMAGE**

Change the tagging value to **untagged** from the dropdown list

**IMAGE**

Click on **studentxx-demo** on the link ncs:services/aci-bigip:aci-bigip{**student-demo**}/load-balancer/vlans{vlan}/interfaces{1.1}/
in the upper left corner

**IMAGE**

Scroll down to section **/load-balancer/self-ip/**, click the **+** sign

**IMAGE**

A pop up window will appear, enter the name of the self-ip **selfip** and click on **confirm**

**IMAGE**

Click on the self-ip created

**IMAGE**

Enter the IP address - {}/24
Choose **vlan** from the drop down list

**IMAGE**

Click on **studentxx-demo** on the link ncs:services/aci-bigip:aci-bigip{**student-demo**}/load-balancer/self-ip{selfip}/
in the upper left corner

Scroll down to section **/load-balancer/virtual-server/**, click the **+** sign

**IMAGE**

A pop up window will appear, enter the name of the virtual server **http_vs** and click on **confirm**

**IMAGE**

Click on the virtual server created

**IMAGE**

Enter the destination-ip - {}
Enter the port number - 80

Under section /profiles , click on the **+** sign

**IMAGE**

A pop up window will appear, enter **http** and click **confirm**

**IMAGE**

Click on **studentxx-demo** on the link ncs:services/aci-bigip:aci-bigip{**student-demo**}/virtual-server/self-ip{http_vs}/
in the upper left corner

**IMAGE**

Scroll down to section **/load-balancer/pool/**, enter the following
* name - **http_pool**
* load-balancing-method - leave it as **round-robin**

**IMAGE**

Under section **/load-balancer/pool/monitor**, click on the **+** sign

**IMAGE**

A pop up window will appear, enter the monitor **http** and click **confirm**

**IMAGE**

Scroll down to section **/load-balancer/pool/members**, click on the **+** sign 

**IMAGE**

A pop up window will appear, enter **node1** and click **confrim**

**IMAGE**

Click on the node created 

**IMAGE**

Enter Node IP address = {}

**IMAGE**

Click on **studentxx-demo** on the link ncs:services/aci-bigip:aci-bigip{**student-demo**}/pool/members{node1}/
in the upper left corner

**IMAGE**

Scroll to the bottom and add another node like you did previosuly.  
* name - **node2**
* IP address - **{}**

**IMAGE**

Now we are going to commit the configuration

Click on **commit manager** at the bottom of the page

You will see four different tabs **changes**, **warnings**, **config** and **native config**. Click on each and familarize yourself with the configuration that is going to be pushed. Make sure there are NO warnings.

**IMAGE**

Once completed, click the **commit** buttom on the top right hand corner

**IMAGE**

When commited successfully you will see Current Transaction - **empty**

**IMAGE**

--------------------------------------------------------------------------------------------------
Click on **Lab-Git-Project**. This is a view only permission.

![](images/Tower-Project1.png)

The SCM URL defines the Github repo from where the playbooks and other content is being pulled from.

![](images/Tower-Project2.png)

Click on **Templates** located in the top level menu: 

Click on the template **Configure-ACI**
* This is a view only template. This template will not be launched.
* There is a project associated with the template (which is the GIT project).
* There is a ansible playbook associated with the template (pulled from GIT - aci_configuration.yaml).

![](images/Tower-Template1.png)
![](images/Tower-Template2.png)

**Playbook contents - aci_configuration.yaml** - To view contents [click here](https://github.com/f5devcentral/f5-aci-integration-automation-ansible/blob/master/Labs/AnsibleWorkflow/playbooks/aci_configuration.yaml)
* Take as input Jinga2 files and covert them to XML files
	* Jinga2 files allow the user to variabalize the content as needed. To view the Jinga2 files used for this playbook [click here](https://github.com/f5devcentral/f5-aci-integration-automation-ansible/tree/master/Labs/AnsibleWorkflow/aci_posts-OneArmDeployment)
* Use the aci_rest Ansible module and post the XML files created in the above step to ACI. Following is what is getting configured on the APIC. To view the playbook contents [click here](https://github.com/f5devcentral/f5-aci-integration-automation-ansible/blob/master/Labs/AnsibleWorkflow/playbooks/aci_configuration.yaml)
	* Logical Device Cluster
	* Contract
	* Service graph template
	* Attach service graph template to contract
		* Device selection policy
		* Assign provided contract to provider EPG
		* Assign consumer contract to consumer EPG

Back on Ansible tower scroll down -> Click on the template **Configure-BIG-IP**
* This is a view only template. This template will not be launched.
* There is a project associated with the template (which is the GIT project).
* There is a ansible playbook associated with the template (pulled from GIT - bigip_configuration.yaml).

![](images/Tower-Template3.png)

**Playbook contents - bigip_configuration.yaml** - To view contents [click here](https://github.com/f5devcentral/f5-aci-integration-automation-ansible/blob/master/Labs/AnsibleWorkflow/playbooks/bigip_configuration.yaml)
* Calls another playbook named **onboarding.yaml** to configure onboarding tasks. To view contents [click here](https://github.com/f5devcentral/f5-aci-integration-automation-ansible/blob/master/Labs/AnsibleWorkflow/common/onboarding.yaml)
	* NTP
	* DNS
	* Hostname
	* SSHD setting
* Configures the network
	* VLAN
	* Self-IP
	* Static route
* Calls another playbook names **http-service.yaml** to configure L7 tasks. To view contents [click here](https://github.com/f5devcentral/f5-aci-integration-automation-ansible/blob/master/Labs/AnsibleWorkflow/common/http_service.yaml)
	* Node members
	* Pool
	* Virtual Server

Go back to Ansible Tower. A workflow has been created in Ansible Tower to chain the execution of the above two playbooks

Scroll down -> Click on the template **Configure-Workflow**. 
This is a workflow template consisting of two playbooks we viewed earlier
1) Configure-ACI
2) Configure-BIG-IP

![](images/Tower-Workflow.png)

Click on the 'Workflow Editor' button to view the workflow configured

![](images/Tower-Workflow1.png)

After viewing 'close' the workflow editor

![](images/Tower-Workflow2.png)

Now we will view the parameters that we are going to pass to the playbook for execution. There are two methods through which we are passing variables to the playbook. One is through the **Extra Variables** text box and the other is through the **Survey**

![](images/Tower-variables.png)

View the paramters in the **Extra Variables** text box. Values that you provide here will be provided as input to the playbooks in the workflow

![](images/Tower-Workflow3.png)


Scroll through and make sure the following paramters are specified:

```
#################
#APIC information
#################
consumerBD_name: "vip-bd" 			#Consumer Bridge domain name
providerBD_name: "vip-bd"			#Provider bridge domain name

appProfile_name: "app"				#Application profile name
consumerEPG_name: "epg-l3out"			#Consumer EPG name
providerEPG_name: "web-epg" 			#Provider EPG name

SGtemplate_name: "sgt"				#Service graph template name
contract_name: "cntr"				#Contract name

logicalDeviceCluster_name: "bigip"		#Logical Device Cluster name

#################
#BIG-IP information
#################

vlan_name: vlan
vlan_id: '1234'
vlan_interface: '1.1'

bigip1_selfip_name: 'SelfIP'
bigip1_selfip_netmask: 255.255.255.0

static_route_name: default
static_route_destination: 0.0.0.0
static_route_netmask: 0.0.0.0

pool_name: http-pool
pool_member1_port: '80'
pool_member2_port: '80'

vip_name: http
vip_port: '80'
snat: automap
pool_name: http-pool
profiles:
 - http
```

Scroll to the bottom and click on the 'Rocket' icon next to the template.

![](images/Tower-LaunchWorkflow.png)

This will launch the playbook. A survey will pop up when the rocket button is clicked.
The survey is an Ansible Tower feature to allow users to provide input to the playbook while executing the playbook. These are variables passed to the playbook along with the input provided in the 'Extra Variables' text box earlier.

```
In the Survey enter the following:
BIG-IP - Do you want to on-board? = 'yes'
BIG-IP - Deploy L7 configuration? = 'yes'
BIG-IP IPAddress = '{TBIGIPIP}'
BIG-IP username = 'admin'
BIG-IP password = 'cisco123'
BIG-IP Virtual IP Address: '{TL2F5VIP}'
BIG-IP Self IP Address: '{TL2F5INTSIP}'
BIG-IP Default Route: '{TL2F5VIPGW}'
BIG-IP Pool Member1: '{TVM2IP}'
BIG-IP Pool Member2: '{TVM3IP}'
APIC IPAddress = '172.21.208.173'
APIC username = 'studentxx'	#Replace xx to your student ID
APIC password = 'ciscolive.2018'
APIC Tenant Name = 'studentxx'	#Replace xx to your student ID
```
Click Launch once the Survey is filled according to the parameters above

![](images/Tower-LaunchSurvey.png)

At this point the playbook is executing. It will first configure the APIC and then the BIG-IP

You will see the status of the playbook is in mode **Running**. The workflow will first execute Configure-ACI playbook. Click on details

![](images/Tower-RunWorflow1.png)

The new window will indicate the progress of running playbook **Configure-ACI**

![](images/Tower-RunWorflow2.png)

After the playbook has executed sucessfully click on **Jobs** on the top left hand corner

![](images/Tower-RunWorflow3.png)

You will see the Configure-Workflow job being executed (blinking green icon next to it), click it. It will take you back to the workflow execution, at this point the **Configure-ACI** playbook would have executed sucessfully and the playbook **Configure-BIG-IP** will be getting executed. Click on details

![](images/Tower-RunWorflow4.png)

![](images/Tower-RunWorflow5.png)

Once the playbook has executed sucessfully again click on **Jobs**. You will see five jobs got executed as part of the workflow. From the bottom

* Git project SCM update (before a playbook is run the GIT repo is updated to make sure the latest code is available)
* Configure-ACI
* Git project SCM update
* Configure-BIG-IP
* Workflow execution

The JOB ID in the screen shot does not need to match what you see

![](images/Tower-RunWorflow6.png)

**This concludes the section on using Ansible playbooks to configure APIC and BIG-IP**

## Verifying the Deployment

### Verify APIC configuration
Let's login into the APIC with the following username and password from the web browser

* APIC : http://172.21.208.173
* Username: {TSTUDENT}
* Password: ciscolive.2018

On the APIC GUI click on **Tenants**. In the Tenant Search text box enter your student ID. Example: student01. This will open up your tenant on the left hand side of the APIC GUI pane

![](images/APIC-Tenant.png)

In the left hand pane under your tenant to view the logical device cluster deployed click on 
**Services->L4-L7->Devices->bigip**

![](images/APIC-LDC.png)

In the logical device cluster the following has been configured
* Name: **bigip**  
* Service Type: **ADC**  
* Device Type: **Virtual**  
* VMM Domain: **VMware/CLBerlin2016** 
* View: **Single Node**
* Context Aware: **Single**  
* Function Type: **GoTo**  
* Device 1
	* VM Nme: **{TBIGIPVM}** #Name of the BIG-IP Virtual Edition
	* vCenter name: **DMZ_VC**
	* Interface: 1_1 (One arm mode so only one interface on BIG-IP specified for client and server traffic)
* Logical interface
	* Consumer is mapped to Device1 interface 1_1
	* Provider is mapped to Device1 interface 1_1

In the left hand pane under your tenant to view the service graph template click on 
**Services->L4-L7->Service Graph Template->sgt**

![](images/APIC-SGT.png)

The service graph template has been configured
* One-Arm mode
* Associated to logical device cluster **bigip** 

To deploy the service graph a few steps are needed 
* Assign service graph template to contract
* Create device selection policy
* Attach contracts to the correct EPG's

In the left hand pane under your tenant to view the contract click on  
**Contracts->Standard->cntr->all**

![](images/APIC-Contract.png)

The service graph template **sgt** has been assigned to the contract

In the left hand pane under your tenant to view the device selection policy click on   
**Services->L4-L7->Device Selection Policy->cntr-sgt-ADC**

![](images/APIC-DSP.png)

The logical device context instructs Cisco Application Centric Infrastructure (ACI) about which load balancer device to use to render a graph. The device **bigip** is assigned for rendering the graph

In the left hand pane under your tenant to view the provided contract assigned to the EPG's click on   
**Application Profiles->app->Application EPGs->web-epg->Contracts**

![](images/APIC-Prov-Contract.png)

Contract **cntr** is assigned as a Provided contract to EPG **web-epg**

In the left hand pane under your tenant to view the consumer contract assigned to the EPG's click on  
**Networking->External Routed Networks->studentxx-l3out->Networks->epg-l3out** (where studentxx represents your student ID)

![](images/APIC-Cons-Contract.png)

Contract **cntr** is assigned as a Consumer contract to EPG **epg-l3out**

In the left hand pane under your tenant to view the deployed graph click on  
**Services->L4-L7-Deployed Graph Instances**

![](images/APIC-Deployed-Graph.png)

The graph is in state **applied** which indicates it was deployed correctly

### Verify BIG-IP configuration

Let’s log into the F5 BIG-IP **{TBIGIPIP}** with the following username and password from the web browser 

* BIG-IP: **[https://{TBIGIPIP}](https://{TBIGIPIP})**  
* Username: **admin**  
* Password: **cisco123**  

On the left Navigation menu, click the **Network -> VLAN** and you should be able to see the vlan assigned to the interface 1_1

![](images/BIGIP-vlan.png)

On the left Navigation menu, click the **Network -> Self IPs** and you should be able to see the Self-IP added and assigend to VLAN above

![](images/BIGIP-Selfip.png)

On the left Navigation menu, Click **Local Traffic -> Nodes** and you should see the brief information of the real server pool information

![](images/BIGIP-nodes.png)

Click **Local Traffic -> Pools** , click on the Pool

![](images/BIGIP-pool.png)

Click the hyperlink under **Name** and you should be directed to the Pool **Properties** page. Now click the **Members** tab and you should see the real servers (pool members) we configured when we were deploying the service graph.

![](images/BIGIP-poolmembers.png)

Click the **Local Traffic -> Virtual Servers** and you should be able to see the brief Virtual IP information. You can see that the VIP is currently listening on HTTP port 80.

![](images/BIGIP-vs.png)

In the **Virtual Server List**, click the **Name** in the hyperlink and you will see the **Property** of the Virtual Server with more detailed information. The configured the parameters will appear here. 

Click the **Resources** tab and you should see the both **Default** and **Fallback** persistence profiles are set to **None**. Also the **Default Pool** has been set.

![](images/BIGIP-vs1.png)

You can now verify the Virtual Server (or VIP) by using your browser and entering the VIP into the address window:  

URL: **[http://{TL2F5VIP}](http://{TL2F5VIP})**  

You should see the page with the hostname of your VMs similar to the following:   

Press the enter button (do not use the refresh button of your browser) at the IP address **{TL2F5VIP}** at the web browser, you should see a different page and the VIP is load balanced by the ADC.

We have verified connectivity to the web server via the ADC VIP.

**This concludes the section for the Lab**

## Automate cleanup on BIG-IP and APIC using Ansible

Connect to the Ansible tower using the following information:
* **Ansible Tower Address**: 172.21.208.250
* **username**: {TSTUDENT}
* **password**: cisco123

Click on **Templates** located in the top level menu: 

![](images/Tower-Cleanup-Template1.png)

Click on the template **Cleanup-BIG-IP**
* This is a view only template. This template will not be launched.
* There is a project associated with the template (which is the GIT project).
* There is a ansible playbook associated with the template (pulled from GIT - bigip_configuration_delete.yaml).

![](images/Tower-Cleanup-Template2.png)

**Playbook contents - bigip_configuration_delete.yaml** - To view contents [click here](https://github.com/f5devcentral/f5-aci-integration-automation-ansible/blob/master/Labs/AnsibleWorkflow/playbooks/bigip_configuration_delete.yaml)
* Calls another playbook named **http_service_cleanup.yaml** to delete L7 configuration. To view contents [click here](https://github.com/f5devcentral/f5-aci-integration-automation-ansible/blob/master/Labs/AnsibleWorkflow/common/http_service_cleanup.yaml)
* Deletes the L7 configuration
	* Node members
	* Pool
	* Virtual Server
* Delete the network configuration
	* VLAN
	* Self-IP
	* Static route

Go back to Ansible Tower, Click on the template **Cleanup-ACI**
* This is a view only template. This template will not be launched.
* There is a project associated with the template (which is the GIT project).
* There is a ansible playbook associated with the template (pulled from GIT - aci_configuration_delete.yaml).

![](images/Tower-Cleanup-Template3.png)

**Playbook contents - aci_configuration_delete.yaml** - To view contents [click here](https://github.com/f5devcentral/f5-aci-integration-automation-ansible/blob/master/Labs/AnsibleWorkflow/playbooks/aci_configuration_delete.yaml)
* Take as input Jinga2 files and covert them to XML files
	* Jinga2 files allow the user to variabalize the content as needed. To view the Jinga2 files used for this playbook [click here](https://github.com/f5devcentral/f5-aci-integration-automation-ansible/tree/master/Labs/AnsibleWorkflow/aci_posts_delete)
* Use the aci_rest Ansible module and post the XML files created in the above step to ACI. Following is what is getting deleted on the APIC. To view the playbook contents [click here](https://github.com/f5devcentral/f5-aci-integration-automation-ansible/blob/master/Labs/AnsibleWorkflow/playbooks/aci_configuration_delete.yaml)
	* Contract
	* Dettach service graph template to contract
		* Device selection policy
		* Unassign provided contract to provider EPG
		* Unassign consumer contract to consumer EPG
	* Service graph template
	* Logical Device Cluster

Go back to Ansible Tower, a workflow has been created in Ansible Tower to chain the execution of the above two playbooks

Click on the template **Cleanup-Workflow**. 
This is a workflow template consisting of two playbooks we viewed earlier
1) Cleanup-BIG-IP
2) Cleanup-ACI

![](images/Tower-Cleanup-Workflow.png)

Click on the 'Workflow Editor' button to view the workflow configured

![](images/Tower-Cleanup-Workfloweditor.png)

After viewing 'close' the workflow editor

![](images/Tower-Cleanup-Workfloweditor1.png)

View the parameters in the 'Extra Variables' text box. Values that you provide here will be provided as input to the playbooks in the workflow. We will use the same paramters as used in the configuration workflow. 

Scroll to the bottom and click on the 'Rocket' icon next to the template.

![](images/Tower-Cleanup-Launchworkflow.png)


This will launch the playbook. A survey will pop up when the rocket button is clicked.
The survey is an Ansible Tower feature to allow users to provide input to the playbook while executing the playbook. These are variables passed to the playbook along with the input provided in the 'Extra Variables' text box.

```
In the Survey enter the following:
BIG-IP IPAddress = '{TBIGIPIP}'
BIG-IP username = 'admin'
BIG-IP password = 'cisco123'
BIG-IP Virtual IP Address: '{TL2F5VIP}'
BIG-IP Self IP Address: '{TL2F5INTSIP}'
BIG-IP Default Route: '{TL2F5VIPGW}'
BIG-IP Pool Member1: '{TVM2IP}'
BIG-IP Pool Member2: '{TVM3IP}'
APIC IPAddress = '172.21.208.173'
APIC username = 'studentxx'	#Replace xx to your student ID
APIC password = 'ciscolive.2018'
APIC Tenant Name = 'studentxx'	#Replace xx to your student ID
```
Click Launch once the Survey is filled according to the parameters above

![](images/Tower-Cleanup-Launchsurvey.png)

At this point the playbook is executing. It will first cleanup the BIG-IP and then the APIC

You will see the status of the playbook is in mode **Pending**. The workflow will first execute Cleanup-BIG-IP playbook. Click on **details**

![](images/Tower-Cleanup-Workflowexecution1.png)

The new window will indicate the progress of running playbook **Cleanup-BIG-IP**

![](images/Tower-Cleanup-Workflowexecution2.png)

After the playbook has executed sucessfully click on **Jobs** on the top left hand corner

You will see the Cleanup-Workflow job being executed (blinking green icon next to it), click on it. It will take you back to the workflow execution, at this point the **Cleanup-BIG-IP** playbook would have executed sucessfully and the playbook **Cleanup-ACI** will be getting executed. Click on details

![](images/Tower-Cleanup-Workflowexecution3.png)

![](images/Tower-Cleanup-Workflowexecution4.png)

Once the playbook has executed sucessfully again click on **Jobs**. You will see five jobs got executed as part of the workflow. From the bottom

* Git project SCM update (before a playbook is run the GIT repo is updated to make sure the latest code is available)
* Configure-ACI
* Git project SCM update
* Configure-BIG-IP
* Workflow execution

The **JOB ID** in the screen shot does not need to match what you see

![](images/Tower-Cleanup-Workflowexecution5.png)

**This concludes the section on using Ansible playbooks to cleanup the APIC and BIG-IP**

## Verifying the cleanup of the deployment

### Verify BIG-IP configuration cleanup

Let’s log into the F5 BIG-IP **{TBIGIPIP}** with the following username and password from the web browser 

* BIG-IP: **[https://{TBIGIPIP}](https://{TBIGIPIP})**  
* Username: **admin**  
* Password: **cisco123**  

Click on the following in the Navigation menu and make sure there is no configuration
* Click the **Network -> VLAN** 
* Click the **Network -> Self IPs** 
* Click **Local Traffic -> Nodes** 
* Click **Local Traffic -> Pools** 
* Click the **Local Traffic -> Virtual Servers** 

### Verify APIC configuration cleanup
Let's login into the APIC with the following username and password from the web browser

* APIC : http://172.21.208.173
* Username: {TSTUDENT}
* Password: ciscolive.2018

On the APIC GUI click on **Tenants**. In the Tenant Search text box enter your student ID. Example: student01. This will open up your tenant on the left hand side of the APIC GUI pane. Navigate to the following and make sure there is no configuration
* Click on **Services->L4-L7->Devices**
* Click on **Services->L4-L7->Service Graph Template**
* Click on **Contracts->Standard**
* Click on **Services->L4-L7->Device Selection Policy**
* Click on **Application Profiles->app->Application EPGs->web-epg->Contracts** 
* Click on **Networking->External Routed Networks->studentxx-l3out->Networks->epg-l3out** (where studentxx represents your student ID)
* Click on **Services->L4-L7-Deployed Graph Instances**

**This concludes the section of the Lab**

>**Congratulations! This session of the lab is completed, please proceed to the next lab session**
