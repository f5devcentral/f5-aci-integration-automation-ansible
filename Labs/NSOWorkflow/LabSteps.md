
# Lab Exercise 

>**Please note that the images used in the lab guide are representative and NOT based on any specific pod. Please use the information in the lab guide instead.**

## Getting started ##

Cisco Application Centric Infrastructure (ACI) technology provides the capability to insert Layer 4 through Layer 7 (L4-L7) functions using an approach called a service graph. The servive graph controls network connectivity consisting of VLANs.

Lab will be used to demonstrate L4-L7 service insertion in unmanaged mode to simulate an enterprise network and/or cloud provider’s application delivery offering while allowing the application owner to manage the L4-L7 device using Ansible. 

The topology used for this Lab is as follows:

![](images/Ansible-topology.png)

The goal is to provide a central point of control to configure both the Cisco APIC as well as the F5 BIG-IP. Network stitching is achieved by automating the deployment of a service graph on APIC and L4-L7 configuration is automated directly on the BIG-IP

![](images/NSO-logicaldiagram.png)

**For this lab**
* We will use the F5 BIG-IP VE Virtual ADC to demonstrate this functionality
* The service graph will be deployed in One-ARM mode which implies **one** interface will be consumed on the BIG-IP which handles the client as well as server traffic
* A default route will be configured on the BIG-IP
* SNAT = 'Automap' will be configured on the BIG-IP (return traffic from backend servers are forced to pass back through the BIG-IP)
* The EPG's used are **web-epg**(provider) and **epg-l3out** (consumer). Bridge domain **vip-bd**

![](images/Ansible-topology1.png)

>**Let's begin the lab**

## Automate configuration on APIC and BIG-IP using Cisco NSO

We will provision the APIC and BIG-IP using NSO.

Connect to NSO using the following information:

* **NSO Address**: http://172.21.208.249:8080
* **username**: admin
* **password**: C1sc0123

Once you are logged in click on **Device Manager** located on the dashboard

![](images/NSO-dm.png)

Before proceeding we are going to sync NSO to have the latest APIC and BIG-IP configuration.  
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

1) Enter the vlan tag **1234**.  
2) Click on **studentxx-demo** on the link ncs:services/aci-bigip:aci-bigip{**studentxx-demo**}/vlans{vlan}/ to go back to the service added

![](images/NSO-sm-vlan3.png)

Under section **/sdn-controller**, from the drop down list choose **cisco-apic**. This information is automatically getting pulled from the NSO. This drop down is a list of all devices that are of type 'Cisco-ACI'. In our lab we have only one device configured on the NSO hence only one device in the drop down menu.

![](images/NSO-sm-apic.png)

Under section **/sdn-controller/tenant/**, fill in the following
* name (drop down list) - **studentxx** -> xx to represent your student pod
* application-profile-name (drop down list) - **app**
* epg-provider-name(drop down list) - **web-epg**
* epg-consumer-name (drop down list) - **epg-l3out**
* bd-provider-name (drop down list) - **vip-bd** 
* bd-comsumer-name (drop down list) - **vip-bd**
* contract-name (type it in) - **cntr**

![](images/NSO-sm-apic1.png)

Under section **/sdn-controller/tenant/vns-ldev-vip**, fill in the following
* name - **bigip**
* device-type - leave it as **VIRTUAL**
* domain-name (drop down list) - **CLBerlin2016**
* vm-name - **BigIP-xx** -> xx to represent your student Pod

![](images/NSO-sm-apic2.png)

Under section **/sdn-controller/tenant/vns-abs-graph**, fill in the following
* name - **sgt**
* template-type - leave it as **ADC-ONE-ARM**

![](images/NSO-sm-apic3.png)

Under section **/load-balancer** from the drop down list choose **f5-bigip**

![](images/NSO-sm-lb.png)

Under section **/load-balancer-vlans/**, click on the '+' sign

![](images/NSO-sm-lb-vlan.png)

A pop up window will appear, from the drop down list choose **vlan** and click **confirm**

![](images/NSO-sm-lb-vlan1.png)

Click on the **vlan** created

![](images/NSO-sm-lb-vlan2.png)

Under section **/interfaces** click on the **+** sign

![](images/NSO-sm-lb-vlan5.png)

A pop up window will appear, choose interface **1.1** from the drop down list and click **confirm**

![](images/NSO-sm-lb-vlan6.png)

Click on the interface name **1.1**.

![](images/NSO-sm-lb-vlan7.png)

1) Change the tagging value to **untagged** from the dropdown list
2) Click on **studentxx-demo** on the link ncs:services/aci-bigip:aci-bigip{**student-demo**}/load-balancer/vlans{vlan}/interfaces{1.1}/ in the upper left corner

![](images/NSO-sm-lb-vlan8.png)

Scroll down to section **/load-balancer/self-ip/**, click the **+** sign

![](images/NSO-sm-lb-selfip.png)

A pop up window will appear, enter the name of the self-ip **selfip** and click on **confirm**

![](images/NSO-sm-lb-selfip1.png)

Click on the self-ip created

![](images/NSO-sm-lb-selfip2.png)

1) Enter the IP address - {TL2F5INTSIP}/24
2) Choose **vlan** from the drop down list
3) Click on **studentxx-demo** on the link ncs:services/aci-bigip:aci-bigip{**student-demo**}/load-balancer/self-ip{selfip}/
in the upper left corner

![](images/NSO-sm-lb-selfip3.png)

Scroll down to section **/load-balancer/route/**, click the **+** sign

![](images/NSO-sm-lb-route.png)

A pop up window will appear, enter the name of the virtual server **default** and click on **confirm**

![](images/NSO-sm-lb-route1.png)

Click on the route created

![](images/NSO-sm-lb-route2.png)

1) Enter the gw-address - {TL2F5VIPGW}
2) Enter the destination-network - 0.0.0.0/0
3) Click on **studentxx-demo** on the link ncs:services/aci-bigip:aci-bigip{**student-demo**}/route{default}/
in the upper left corner

![](images/NSO-sm-lb-route3.png)

Scroll down to section **/load-balancer/virtual-server/**, click the **+** sign

![](images/NSO-sm-lb-vip.png)

A pop up window will appear, enter the name of the virtual server **http_vs** and click on **confirm**

![](images/NSO-sm-lb-vip1.png)

Click on the virtual server created

![](images/NSO-sm-lb-vip2.png)

1) Enter the destination-ip - {TL2F5VIP}
2) Enter the port number - 80
3) Under section /profiles , click on the **+** sign

![](images/NSO-sm-lb-vip3.png)

A pop up window will appear, enter **http** and click **confirm**

![](images/NSO-sm-lb-vip4.png)

Click on **studentxx-demo** on the link ncs:services/aci-bigip:aci-bigip{**student-demo**}/virtual-server/self-ip{http_vs}/
in the upper left corner

![](images/NSO-sm-lb-vip5.png)

Scroll down to section **/load-balancer/pool/**, enter the following
1) name - **http_pool**
2) load-balancing-method - leave it as **round-robin**
3) Under section **/load-balancer/pool/monitor**, click on the **+** sign

![](images/NSO-sm-lb-pool.png)

A pop up window will appear, enter the monitor **http** and click **confirm**

![](images/NSO-sm-lb-pool1.png)

Scroll down to section **/load-balancer/pool/members**, click on the **+** sign 

![](images/NSO-sm-lb-poolmember.png)

A pop up window will appear, enter **node1** and click **confrim**

![](images/NSO-sm-lb-poolmember1.png)

Click on the node created 
1) Enter Node IP address = {TVM2IP}
2) Click on **studentxx-demo** on the link ncs:services/aci-bigip:aci-bigip{**student-demo**}/pool/members{node1}/
in the upper left corner

![](images/NSO-sm-lb-poolmember2.png)

Scroll to the bottom and add another node like you did previosuly.  
* name - **node2**
* IP address - **{TVM3IP}**

After adding node 2, you should see two nodes under **/load-balancer/pool/members**

![](images/NSO-sm-lb-poolmember3.png)

**Now we are going to commit the configuration**

Click on the **cisco** logo on the top left corner. Click **commit manager** on the dashboard

![](images/NSO-sm-commit.png)

You will see four different tabs **changes**, **warnings**, **config** and **native config**. Click on each and familarize yourself with the configuration that is going to be pushed. Make sure there are NO warnings.

Once checked, click the **commit** buttom on the top right hand corner

![](images/NSO-sm-commit1.png)

When commited successfully you will see Current Transaction - **empty**

![](images/NSO-sm-commit2.png)

>**This concludes the section on using NSO to configure APIC and BIG-IP**

--------------------------------------------------------------------------------------------------
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

![](images/NSO-bigip-vlan.png)

On the left Navigation menu, click the **Network -> Self IPs** and you should be able to see the Self-IP added and assigend to VLAN above

![](images/NSO-bigip-selfip.png)

On the left Navigation menu, Click **Local Traffic -> Nodes** and you should see the brief information of the real server pool information

![](images/NSO-bigip-nodes.png)

Click **Local Traffic -> Pools** , click on the Pool

![](images/NSO-bigip-pool.png)

Click the hyperlink under **Name** and you should be directed to the Pool **Properties** page. Now click the **Members** tab and you should see the real servers (pool members) we configured when we were deploying the service graph.

![](images/NSO-bigip-poolmembers.png)

Click the **Local Traffic -> Virtual Servers** and you should be able to see the brief Virtual IP information. You can see that the VIP is currently listening on HTTP port 80.

![](images/NSO-bigip-vs.png)

In the **Virtual Server List**, click the **Name** in the hyperlink and you will see the **Property** of the Virtual Server with more detailed information. The configured the parameters will appear here. 

Click the **Resources** tab and you should see the both **Default** and **Fallback** persistence profiles are set to **None**. Also the **Default Pool** has been set.

![](images/NSO-bigip-vs1.png)

You can now verify the Virtual Server (or VIP) by using your browser and entering the VIP into the address window:  

URL: **[http://{TL2F5VIP}](http://{TL2F5VIP})**  

You should see the page with the hostname of your VMs similar to the following:   

Press the enter button (do not use the refresh button of your browser) at the IP address **{TL2F5VIP}** at the web browser, you should see a different page and the VIP is load balanced by the ADC.

We have verified connectivity to the web server via the ADC VIP.

>**This concludes the section for the Lab**

--------------------------------------------------------------------------------------------------

## Automate cleanup on BIG-IP and APIC using NSO

Connect to NSO using the following information:

* **NSO Address**: http://172.21.208.249:8080
* **username**: admin
* **password**: C1sc0123

Click on the **Cisco** icon on the top left corner to go back to the dashboard

Click on **Service Manager** located on the dashboard.  

![](images/NSO-cleanup-commit.png)

* Drop down list **Select service point** in the upper left corner and choose **aci-bigip**. The service model we commited earlier will show up **studentxx-demo**. 
* Check box next to **studentxx-demo**
* Click on the **-** sign

![](images/NSO-cleanup-commit1.png)

Click on the **Cisco** icon on the top left corner to go back to the dashboard

Click on **Commit Manager** located on the dashboard.  

![](images/NSO-cleanup-commit2.png)

Click on **native config** and look how the content is being clean up

![](images/NSO-cleanup-commit3.png)

Click on **commit** to start the cleanup process

![](images/NSO-cleanup-commit4.png)

Cleanup should be successful with the current transsaction showing as **Empty**

**This concludes the section on using NSO to cleanup the APIC and BIG-IP**

--------------------------------------------------------------------------------------------------

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
* Click on **Networking->External Routed Networks->studentxx-l3out->Networks->epg-l3out** (where studentxx represents your student ID) - no consumed contract
* Click on **Services->L4-L7-Deployed Graph Instances**

>**This concludes the section of the Lab**

>**Congratulations! This session of the lab is completed, please proceed to the next lab session**

-----------------
# Bonus Lab

Do try this lab if you have experience using Putty and are comfortable editing files in a linux environment

Use API to NSO to deploy the service model instead of using the NSO GUI. This gives you one touch point to configure both the ACI and the BIG-IP

Steps:
* Open notepad in your windows enviroment (Click on the windows icon on the bottm left corner and search for notepad)
* Copy the following payload and make edits as menitoned below
  *  Under aci-bigip:aci-bigip,  "name": "**student**-demo" **Change the value to reflect your demo pod (example: student02-demo)**
  *  Under tenant , "name": "student**01**" **Change the value to reflect your demo pod (example: student02)**
  *  Under tenant->vns-ldev-vip, "vm-name": "BigIP-**01**" **Change the value to reflect your demo pod (example: BigIP-02)**

```
{
  "aci-bigip:aci-bigip": [
    {
      "name": "student-demo",
      "vlans": [
        {
          "name": "vlan-demo",
          "tag": 1234
        }
      ],
      "sdn-controller": {
        "device-name": "cisco-apic",
        "tenant": {
          "name": "student01",
          "application-profile-name": "app",
          "epg-provider-name": "web-epg",
          "epg-consumer-name": "epg-l3out",
          "bd-provider-name": "vip-bd",
          "bd-consumer-name": "vip-bd",
          "contract-name": "cntr",
          "vns-ldev-vip": {
            "name": "bigip",
            "domain-name": "CLBerlin2016",
            "vm-name": "BigIP-01"
          },
          "vns-abs-graph": {
            "name": "sgt"
          }
        }
      },
      "load-balancer": {
        "device-name": "f5-bigip",
        "vlans": [
          {
            "name": "vlan-demo",
            "interfaces": [
              {
                "name": "1.1",
                "tagging": "untagged"
              }
            ]
          }
        ],
        "self-ip": [
          {
            "name": "selfip",
            "ip-address": "{TL2F5INTSIP}\/24",
            "vlan": "vlan-demo"
          }
        ],
        "route": [
          {
            "name": "default",
            "gw-address": "{TL2F5VIPGW}",
            "destination-network": "0.0.0.0\/00"
          }
        ],
        "virtual-server": [
          {
            "name": "http_vs",
            "destination-ip": "{TL2F5VIP}",
            "destination-port": 80,
            "profiles": [
              "http"
            ]
          }
        ],
        "pool": {
          "name": "http_pool",
          "monitor": [
            "http"
          ],
          "members": [
            {
              "name": "node1",
              "ip-address": "{TVM2IP}"
            },
            {
              "name": "node2",
              "ip-address": "{TVM3IP}"
            }
          ]
        }
      }
    }
  ]
}
```

* Open putty on your desktop. SSH to the BIG-IP
  * Login - {TBIGIPIP}
  * Username: root
  * Password: cisco123

* Run command vi sm.json (this will open up a new file)
* Copy the edited contents from notepad to the sm.json file (This will be the payload to send to the NSO service model)
  * Press **i** once you **vi** the file to be able to add content to the file
* Save the file, command to use **:wq!**
* This will take you back to the CLI
* Run command
  * curl -k -v --header "Content-type:application/vnd.yang.data+json" --user admin:C1sc0123 -X POST -d @sm.json http://172.21.208.249:8080/api/running/services
  
Once executed you can go to the NSO as well as the APIC and BIG-IP to verify that the service model has been deployed
