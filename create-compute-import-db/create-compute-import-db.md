# Create Bastion Server and Import MySQL Data

![mysql heatwave](./images/mysql-heatwave-logo.jpg "mysql heatwave")

## Introduction

When working in the cloud, there are often times when your servers and services are not exposed to the public internet. MySQL HeatWave on OCI is an example of a service that is only accessible through private networks. Since the service is fully managed, we keep it siloed away from the internet to help protect your data from potential attacks and vulnerabilities. It’s a good practice to limit resource exposure as much as possible, but at some point, you’ll likely want to connect to those resources. That’s where Compute Instance, also known as a Bastion host, enters the picture. This Compute Instance Bastion Host is a resource that sits between the private resource and the endpoint which requires access to the private network and can act as a “jump box” to allow you to log in to the private resource through protocols like SSH. This bastion host requires a Virtual Cloud Network and Compute Instance to connect with the MySQL DB Systems.


_Estimated Lab Time:_ 20 minutes

### Objectives

In this lab, you will be guided through the following tasks:

- Create SSH Key on OCI Cloud
- Create Bastien Compute Instance
- Install MySQL Shell on the Compute Instance
- Connect to MySQL Database System
- Import and test External MySQL Data

### Prerequisites

- An Oracle Trial or Paid Cloud Account
- Some Experience with MySQL Shell
- Must Complete Lab 1 - be **Active**

## Task 1: Create SSH Key on OCI Cloud Shell

The Cloud Shell machine is a small virtual machine running a Bash shell which you access through the Oracle Cloud Console (Homepage). You will start the Cloud Shell and generate a SSH Key to use  for the Bastion  session.

1. To start the Oracle Cloud shell, go to your Cloud console and click the cloud shell icon at the top right of the page. This will open the Cloud Shell in the browser, the first time it takes some time to generate it.

    cloudshell-main

    ![cloud shell main](./images/cloud-shell.png  "cloud shell main " )

    ![cloud shell button](./images/cloud-shell-setup.png  "cloud shell button " )

    ![open cloud shell](./images/cloud-shell-open.png "open cloud shell" )

    _Note: You can use the icons in the upper right corner of the Cloud Shell window to minimize, maximize, restart, and close your Cloud Shell session._

2. Once the cloud shell has started, create the SSH Key using the following command:

    ```bash
    <copy>ssh-keygen -t rsa</copy>
    ```

    Press enter for each question.

    Here is what it should look like.  

    ![ssh key](./images/ssh-key-show.png "ssh key show")

3. The public  and  private SSH keys  are stored in ~/.ssh/id_rsa.pub.

4. Examine the two files that you just created.

    ```bash
    <copy>cd .ssh</copy>
    ```

    ```bash
    <copy>ls</copy>
    ```

    ![ssh key list ](./images/shh-key-list.png "shh key list")

    Note in the output there are two files, a *private key:* `id_rsa` and a *public key:* `id_rsa.pub`. Keep the private key safe and don't share its content with anyone. The public key will be needed for various activities and can be uploaded to certain systems as well as copied and pasted to facilitate secure communications in the cloud.

## Task 2: Create Compute instance

You will need a compute Instance to connect to your brand new MySQL database.

1. Before creating the Compute instance open a notepad 

2. Do the followings steps to copy the public SSH key to the  notepad

    Open the Cloud shell
    ![open cloud shell large](./images/cloud-shell-open-large.png "open cloud shell large ")

    Enter the following command  

    ```bash
    <copy>cat ~/.ssh/id_rsa.pub</copy>
    ```

    ![ssh key display](./images/ssh-key-display.png "ssh key display ") 

3. Copy the id_rsa.pub content the notepad

    Your notepad should look like this
    ![show ssh key](./images/notepad-rsa-key.png "show ssh key")  

4. Minimize cloud shell

    ![minimize cloud shell](./images/ssh-key-display-minimize.png "minimize cloud shell")  

5. To launch a Linux Compute instance, go to 
    Navigation Menu
    Compute
    Instances
    ![navigation compute](./images/navigation-compute.png "navigation compute")

6. On Instances in **(root)** Compartment, click  **Create Instance**
    ![compute menu create instance](./images/compute-menu-create-instance.png "ccompute menu create instance ")

7. On Create Compute Instance 

    Enter Name

    ```bash
    <copy>HEATWAVE-Client</copy>
    ```

8. Make sure **(root)** compartment is selected 

9. On Placement, keep the selected Availability Domain

10. On Security, keep the default

    - Shielded instance: Disabled
    - Confidential computing:Disabled

      ![compute create security](./images/compute-create-security.png "compute create security ") 

11. On Image  keep the selected Image, Oracle Linux 8 and click Edit

      ![compute create image](./images/compute-create-image.png "compute create image ")  

12. Click Change Shape

      ![compute create change shape](./images/compute-create-change-shape.png "compute create change shape")  

13. Select Instance Shape: VM.Standard.E2.2

      ![compute create select shape](./images/compute-create-select-shape.png "compute create select shape")  

14. On Networking, click Edit

      ![compute create networking](./images/compute-create-networking.png "compute create networking ")  

15. Make sure **HEATWAVE-VCN**  and  and  **public subnet-HEATWAVE-VCN** are selected. Keep Public IPV4 address **Assign..** default

      ![compute create networking](./images/compute-create-networking-select.png "compute create networking ")

16. On Add SSH keys, paste the public key from the notepad.
  
    ![compute create add ssh key](./images/compute-create-add-ssh-key.png "compute create add ssh key ")

17. Keep Boot Volume default and Click **Create** button to finish creating your Compute Instance.

    ![compute create boot volue](./images/compute-create-boot-volume.png "compute create boot volume")

18. The New Virtual Machine will be ready to use after a few minutes. The state will be shown as 'Provisioning' during the creation
    ![compute provisioning](./images/compute-provisioning.png "compute provisioning ")

19. The state 'Running' indicates that the Virtual Machine is ready to use.

    ![compute active](./images/compute-active.png "compute active")

## Task 3: Connect to Bastien Compute and Install MySQL Shell

1. Copy the public IP address of the active Compute Instance to your notepad

    - Go to Navigation Menu
        - Compute
        - Instances
        - Copy **Public IP**
    ![navigation compute with instance](./images/navigation-compute-with-instance.png "navigation compute with instance ")

2. Go to Cloud shell to SSH into the new Compute Instance

    Enter the username **opc** and the Public **IP Address**.

    Note: The **HEATWAVE-Client**  shows the  Public IP Address as mentioned on TASK 3: Step 1

    (Example: **ssh -i ~/.ssh/id_rsa opc@132.145.170...**) 

    ```bash
    <copy>ssh -i ~/.ssh/id_rsa opc@<your_compute_instance_ip></copy>
    ```

    For the **Are you sure you want to continue connecting (yes/no)?**
    - answer **yes**

    ![connect signin](./images/connect-first-signin.png "connect signin ")

3. You will need a MySQL client tool to connect to your new MySQL HeatWave System from the Batien.

    Install MySQL Shell with the following command (enter y for each question)

    **[opc@…]$**

    ```bash
    <copy>sudo yum install mysql-shell -y</copy>
    ```

    ![mysql shell install](./images/mysql-install-shell.png "mysql shell install ")

## Task 4: Download Airportdb Database

The installation procedure involves downloading the airportdb database to cloud shell and importing the data from cloud shell into the MySQL DB System using the MySQL Shell Dump Loading utility. For information about this utility

To install the airportdb database:

1. Download the airportdb sample database and unpack it. The airportdb sample database is provided for download as a compressed tar or Zip archive. The download is approximately 640 MBs in size.

    a. Get sample file

    ```bash
    <copy>wget https://downloads.mysql.com/docs/airport-db.zip</copy>
    ```

    b. Unzip sample file

    ```bash
    <copy>unzip airport-db.zip</copy>
    ```

2. Download the BTS_RAW_DATA
   
    ```bash
    <copy>https://objectstorage.us-ashburn-1.oraclecloud.com/p/cWn1xdJTll89vPw2Yj4qCe-vTB03M0pGB3AexoBE7efKvcsC3JW0FAcNRETupzds/n/idi1o0a010nx/b/Bucket-CA/o/AutoML/BTS_RAW.csv</copy>
    ```

## Task 5: Import external airportdb schema data into HeatWave DB

The data loading process involves using MySQL Shell and Object storage to create the airportdb schema and import its data using the MySQL Shell Dump Loading utility. For information about this utility, see Dump Loading Utility: [https://dev.mysql.com/doc/mysql-shell/8.0/en/mysql-shell-utilities-load-dump.htmly](https://dev.mysql.com/doc/mysql-shell/8.0/en/mysql-shell-utilities-load-dump.html)

1. You will use this Compute instance to connect to HEATWAVE-DB  using the MySQL Shell client tool and the HeatWave system endpoint (private IP Address) .

   The endpoint (IP Address) can be found in your notepad or go to the **HEATWAVE-HW** DB System page

    Click on the OCI Menu > Databases > DB Systems ... and click on your **HEATWAVE-DB** database. When the **HEATWAVE-DB** Page displays, select the **Connections** tab.
      ![active DB](./images/mysql-detail-active.png "active DB")

    Copy the MySQL Endpoint (Private IP Address) to use in the next step.

    ![heatwave endpoint](./images/mysql-detail-endpoint.png "heatwave endpoint")

2. Use the following command to connect to MySQL using the MySQL Shell client tool. Be sure to add the **HEATWAVE-DB**  private IP address at the end of the command. Also enter the admin user and the db password created on Lab 1

    (Example  **mysqlsh -uadmin -p -h <private IP address>**)

    **[opc@...]$**

    ```bash
    <copy>mysqlsh -uadmin -p -h 10.0.1... </copy>
    ```

    ![mysql shell first connect](./images/mysql-shell-first-connect.png "mysql shell first connect ")

3. Load the airportdb database into the MySQL DB System using the MySQL Shell Dump Loading Utility.

    ```bash
    <copy>\js</copy>
    ```

    ```bash
    <copy>util.loadDump("airportdb", {threads: 16, deferTableIndexes: "all", ignoreVersion: true, loadIndexes:false})</copy>
    ```

    ![mysql load data](./images/mysql-load-data.png "mysql load data ")

4. View  the airportdb total records per table

    ```bash
    <copy>\sql</copy>
    ```

    ```bash
    <copy>SELECT table_name, table_rows FROM INFORMATION_SCHEMA.TABLES WHERE TABLE_SCHEMA = 'airportdb';</copy>
    ```

    ![airportdb total records](./images/airportdb-list.png "airportdb total records") 

5. Exit MySQL Shell

    ```bash
      <copy>\q</copy>
    ```

You may now **proceed to the next lab**

## Acknowledgements

- **Author** - Perside Foster, MySQL Principal Solution Engineering
- **Contributors** - Mandy Pang, MySQL Principal Product Manager,  Nick Mader, MySQL Global Channel Enablement & Strategy Manager
- **Last Updated By/Date** - Cristian Aguilar, MySQL Staff Solution Engineering, August 2024
