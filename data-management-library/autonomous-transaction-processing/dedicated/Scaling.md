<table class="tbl-heading"><tr><td class="td-logo">

![](./images/obe_tag.png)

Sept 1, 2019
</td>
<td class="td-banner">
# Test online scaling capabilities of the autonomous database service 
</td></tr><table>

### Need Help?
Please submit feedback or ask for help using our [LiveLabs Support Forum](https://community.oracle.com/tech/developers/categories/livelabsdiscussions). Please click the **Log In** button and login using your Oracle Account. Click the **Ask A Question** button to the left to start a *New Discussion* or *Ask a Question*.  Please include your workshop name and lab name.  You can also include screenshots and attach files.  Engage directly with the author of the workshop.

If you do not have an Oracle Account, click [here](https://profile.oracle.com/myprofile/account/create-account.jspx) to create one.
## Introduction


**Continuous uptime is a key requirement of most business critical applications. While the cloud provides the notion of unlimited scale, this scalability comes at a cost of downtime with most services. The Oracle autonomous database service provides scalability without downtime as demonstrated in this lab **


## Objectives

As an adminstrator,
- Simulate a production workload using Swingbench load generator
- Scale up the OCPUs in your autonomous database service through cloud console
and observe effect on the workload performance
- Scale down the OCPUs in your autonomous database service through cloud console
and observe effect on the workload performance


## Required Artifacts & Prerequisites

- An Oracle Cloud Infrastructure account

- A pre-provisioned instance of Oracle Developer Client image configured with Swingbench in an application subnet. Refer to [Lab 15](Swingbench.md)

- A pre-provisioned Autonomous Transaction Processing instance. Refer to [Lab 4](./ProvisionADB.md)

- Successful completion of [Lab 5](./1ConfigureDevClient.md) and [Lab 15](./Swingbench.md)

## Steps

### **STEP 1: Log in to the Oracle Cloud Developer image and start the order entry workload**

<span style="color:red">Note: To complete this lab its mandatory you have a developer client image configured with swingbench, an autonomous dedicated database instance with the wallet uploaded to the dev client. Follow instructions in [Lab 5](./1ConfigureDevClient.md) and [Lab 15](./Swingbench.md) </span>



**The remainder of this lab assumes you are connected to the image through VNC Viewer and are operating from the image itself and not your local machine.**



- ssh into your developer client machine and navigate to folder /home/opc/swingbench/bin

- Start order entry workload

You can now generate loads on your database by running the charbench utility.  Use the command below. There are 2 parameters you can change to modify the amount of load and users being generated. ``The ???uc flag specifies the number of users that will be ramped up, in the case below 64. The ???rt flag specifies the total running time which is set to 30 seconds by default.``  You can stop running charbench at any time with **Ctrl C.**

```
/home/opc/swingbench/bin/charbench -c /home/opc/swingbench/configs/SOE_Server_Side_V2.xml \
            -cf ~/Downloads/your_wallet.zip \
            -cs yourdb_tp \
            -u soe \
            -p yourpassword \
            -v users,tpm,tps,vresp \
            -intermin 0 \
            -intermax 0 \
            -min 0 \
            -max 0 \
            -uc 64 
```
Once swingbench starts running your will see results similar to the screen below. The first column is a time stamp, the second column indicates how many users of the total users requested with the **-uc** parameter are active, and of interest is the 3rd column which indicates transactions per second. If you see any intermittent connect or other error messages, ignore those.

![](./images/Performancehub/swingbenchoutput.jpeg)



### **STEP 2: Scale up the OCPUs in your autonomous database service**


Now, lets scale the autonomous database you created in previous labs from 1 OCPU to 2 or more while swingbench continues running. You should see the transactions/sec numbers start changing as your database is scaling. Please note that this is also highly influenced by the connection service your are using with the **-cs** parameter. In the statement above we are using the **_tp** service, but while you are experimenting, stop and start swingbench with a different service, such as **_low**, or **_high** to see what differences you observe.

- To scale the system go to your Cloud Database Console and select **Scale Up/Down**:

![](./images/Scaling/scale.jpeg)

When the scaling windows pops up, enter a new **CPU CORE COUNT** and click **UPDATE**

![](./images/Scaling/scale3.jpeg)

The database status will go into **Scaling**. When done it will turn back to **Available**. Please not that during all this time the swingbench application will continue to run because there is no disruption to the service for scaling. You should also notice that the transactions per second increased. or decreased depending on wether you scale the system up or down in number of CPU's.


![](./images/Scaling/swingout2.jpeg)










<table>
<tr><td class="td-logo">

[![](images/obe_tag.png)](#)</td>
<td class="td-banner">
### Congratulations! You successfully tested online scaling capabilities of your autonomous database service.




</td>
</tr>
<table>
