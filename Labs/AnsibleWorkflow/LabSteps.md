# Lab Exercise 

>**Please note that the images used in the lab guide are representative and NOT based on any specific pod. Please use the information in the lab guide instead.**

Lab will be used to demonstrate L4-L7 service insertion in unmanaged mode to simulate an enterprise network and/or cloud provider’s application delivery offering while allowing the application owner to manage the L4-L7 device using their prefered tool. 

We will use the F5 BIG-IP VE Virtual ADC to demonstrate this functionality.

##Create the Unmanaged L4-L7 Device using Ansible

We will provision the APIC using Ansible for a couple of reasons. Ansible is an open source automation platform that can help with configuration management, application deployment, and task automation. It can also do IT orchestration, where you have to run tasks in sequence and create a chain of events which must happen on several different servers or devices.

Red Hat Ansible Tower provides a single point of control your IT infrastructure with a visual dashboard, role-based access control, job scheduling, integrated notifications and graphical inventory management. 

Connect to the Ansible tower using the following information:

* **Ansible Tower Address**: 172.21.208.250
* **username**: {TSTUDENT}
* **password**: cisco123

Once you are logged in click on **Projects** located in the top level menu
Click on 'Lab-Git-Project'. This is a view only permission.
The SCM URL defines the Github repo from where the playbooks and other content is being pulled from.

Click on **Templates** located in the top level menu: 

![ansible tower](../img/ansibletowertemplate.png)

Click on the template **Configure-ACI-podxx**
* This is a view only template. This template will not be launched.
* There is a project associated with the template (which is the GIT project).
* There is a ansible playbook associated with the template (pulled from GIT - aci_configuration.yaml).

Click on the template **Configure-BIG-IP-podxx**
* This is a view only template. This template will not be launched.
* There is a project associated with the template (which is the GIT project).
* There is a ansible playbook associated with the template (pulled from GIT - bigip_configuration.yaml).

A workflow has been created in Ansible Tower to chain the execution of the above two playbooks

Click on the template **ACI-F5 Workflow-podxx**. 
This is a workflow template consisting of two playbooks we viewed earlier
1) Configure-ACI
2) Configure-BIG-IP

Click on the 'Workflow Editor' button to view the workflow configured
After viewing 'close' the workflow editor

View the paramters in the 'Extra Variables' text box. Values that you provide here will be provided as input to the playbooks in the workflow

Edit the paramters to the following:

```
#################
#APIC information
#################
tenant_name: "studentxx"
consumerBD_name: "vip-bd" 
providerBD_name: "vip-bd"

appProfile_name: "app"
consumerEPG_name: "epg-l3out"
providerEPG_name: "web-epg" 

SGtemplate_name: "sxx-sgt"
contract_name: "sxx-cntr"

logicalDeviceCluster_name: "sxx-bigip"

#################
#BIG-IP information
#################

vlan_information:
- name: "VLAN"
  id: "1234"		#Vlan ID
  interface: "1.1"  #Interface on BIG-IP to which the VLAN is untagged

bigip_selfip_information:
- name: 'SelfIP'
  address: '69.2.101.10'					#Self-IP address
  netmask: '255.255.255.0'  
  vlan: "{{vlan_information[0]['name']}}"	#VLAN to be assinged to the BIG-IP

static_route:
- name: "default"
  gw_address: "69.2.101.1"	#Default gateway assigned on the BIG-IP
  destination: "0.0.0.0"
  netmask: "0.0.0.0"

pools:
- pool_name: "http-pool"	#Pool name
  pool_members:				#Members belonging to the BIG-IP Pool
   - port: "80"
     host: "69.2.1.100"
   - port: "80"
     host: "69.2.1.101"
	 
vips:
- vip_name: "http"			#Virtual IP Name
  vip_port: "80"
  vip_ip: "69.2.101.11"		#IP address where the client traffic will be directed to
  snat: "automap"			#SNAT set to automap, so that return traffic is forced back to the BIG-IP
							#No changes needed on the network routing on the backend servers (pool members)
  pool_name: "http-pool"	#Pool to be assigned to the VIP
  profiles:					#Profiles to be assigned to the VIP
   - http
```

Scroll to the bottom and click on the 'Rocket' icon next to the template.
This will launch the playbook. A survey will pop up when the rocket button is clicked.
The survey is an Ansible Tower feature to allow users to provide input to the playbook while executing the playbook. These are variables passed to the playbook along with the input provided in the 'Extra Variables' text box earlier.

In the Survey enter the following:
BIG-IP - device type = 'virtual'
BIG-IP - High Availability or Stand Alone = "SA"
BIG-IP - Do you want to on-board? = 'yes'
BIG-IP - Deploy L7 configuration? = 'yes'
BIG-IP IPAddress = '172.21.208.109'
BIG-IP username = 'admin'
BIG-IP password = 'cisco123'
APIC IPAddress = '172.21.208.173'
APIC username = 'studentxx'
APIC password = 'ciscolive.2018'

## Verifying the BIG-IP Deployment

Let’s log into the F5 BIG-IP **{TBIGIPIP}** with the following username and password from the web browser (if the previous session has timed out): 
 
* BIG-IP: **[https://{TBIGIPIP}](https://{TBIGIPIP})**  
* Username: **admin**  
* Password: **cisco123**  

![F5 BigIP](../img/f5bigip.png)

On the **Main** menu click **Local Traffic -> Network Map**. You should be able to see the virtual server is created along with its pool and pool members.

![Network Map](../img/netmap.png)

On the left Navigation menu, click the **Local Traffic -> Virtual Servers** and you should be able to see the brief Virtual IP information. You can see that the VIP is currently listening on HTTP port 80.

![VIP 1](../img/vip-1.png)

In the **Virtual Server List**, click the **Name** in the hyperlink and you will see the **Property** of the Virtual Server with more detailed information. The configured the parameters will appear here. 

![VIP 1](../img/vip-2.png)

Click the **Resources** tab and you should see the both **Default** and **Fallback** persistence profiles are set to **None**.  

![VIP 2](../img/vip-3.png)

Click **Local Traffic -> Pools** and you should see the brief information of the real server pool information:

![Pool](../img/pool.png)

Click the hyperlink under **Name** and you should be directed to the Pool **Properties** page. Now click the **Members** tab and you should see the real servers (pool members) we configured when we were deploying the service graph.

![Pool Member](../img/pool-member.png)

Go to your RDP command line and ping **{TL2F5VIP}** (VIP). This should succeed.

You can now verify the Virtual Server (or VIP) by using your browser and entering the VIP into the address window:  

URL: **[http://{TL2F5VIP}](http://{TL2F5VIP})**  

You should see the page with the hostname of your VMs similar to the following:   
	
![Page 1](../img/page1.png)

Press the enter button (do not use the refresh button of your browser) at the IP address **{TL2F5VIP}** at the web browser, you should see a different page and the VIP is load balanced by the ADC.

![Page 2](../img/page2.png)

Now that we have verified connectivity to the web server via the ADC VIP, let's inspect the web server and see which client IP is in the access log.

Use the Putty SSH client in your RDP and connect to the first web server VM and verify its access log entries. For example, you can SSH into VM, {TSTUDENT}-vm02, via its L2 only network connection by using the following:

IP address: **{TVM2L2}**  
Username: **student**  
Password: **ciscolive.2018**  

After logged in, issue the following command:  

```
tail -f /var/log/httpd/access_log
```

You should see an output similar to the following:  

```
{TL2F5INTSIP} - - [06/Feb/2018:20:11:52 -0500] "GET /" 200 770 "-" "-"
{TL2F5INTSIP} - - [06/Feb/2018:20:11:56 -0500] "GET /" 200 770 "-" "-"
```

This is the HTTP health probe from the Big-IP. Let's use the web browser in your RDP session to browse the web server via the ADC VIP for multiple times. You should see the browser switch between VM2 and VM3.

Now, let's inspect the HTTP access log and you will notice an output similar to the following:  

```
199.253.253.12 - - [07/Feb/2017:22:29:07 -0500] "GET / HTTP/1.1" 200 770 "-" "Mozilla/5.0 (Windows NT 6.3; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/55.0.2883.87 Safari/537.36"
```  

The IP address in the HTTP access log contains the client's IP in the HTTP request.

>**Congratulations! This session of the lab is completed, please proceed to the next lab session via the menu**