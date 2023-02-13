# project-7

Devops Tooling Website Solution

The tools we want our team to be able to use are well known and widely used by multiple DevOps teams, so we will introduce a single DevOps Tooling Solution that will consist of:

1: Jenkins – free and open source automation server used to build CI/CD pipelines.
2: Kubernetes – an open-source container-orchestration system for automating computer application deployment, scaling, and management.
3: Jfrog Artifactory – Universal Repository Manager supporting all major packaging formats, build tools and CI servers. Artifactory.
4: Rancher – an open source software platform that enables organizations to run and manage Docker and Kubernetes in production.
5: Grafana – a multi-platform open source analytics and interactive visualization web application.
6: Prometheus – An open-source monitoring system with a dimensional data model, flexible query language, efficient time series database and modern alerting approach.
7: Kibana – Kibana is a free and open user interface that lets you visualize your Elasticsearch data and navigate the Elastic Stack.
Note: Do not feel overwhelmed by all the tools and technologies listed above, we will gradually get ourselves familiar with them in upcoming projects!

In this project we will implement a solution that consists of following components:

1; Infrastructure: AWS
2: Webserver Linux: Red Hat Enterprise Linux 8
3: Database Server: Ubuntu 20.04 + MySQL
4: Storage Server: Red Hat Enterprise Linux 8 + NFS Server
5: Programming Language: PHP
6: Code Repository: GitHub

On the diagram below you can see a common pattern where several stateless Web Servers share a common database and also access the same files using Network File Sytem (NFS) as a shared file storage. Even though the NFS server might be located on a completely separate hardware – for Web Servers it look like a local file system from where they can serve the same files.


<img width="924" alt="Screenshot 2023-02-10 at 01 07 54" src="https://user-images.githubusercontent.com/118350020/217967428-0387efdb-1592-4426-b79b-a7ef9df14a97.png">

STEP 1 – PREPARE NFS SERVER
Spin up a new EC2 instance with RHEL Linux 8 Operating System.
Based on our LVM experience from Project 6, Configure LVM on the Server.

connect your NFS server with SSH as shown in the below diagram

<img width="781" alt="Screenshot 2023-02-09 at 03 43 42" src="https://user-images.githubusercontent.com/118350020/217972189-d591301b-1158-4617-b104-ec18879df7b4.png">




The first we want to do is to list all the blocks that are attached to this instance
using the below command as shown in the below diagram
lsblk

<img width="782" alt="Screenshot 2023-02-09 at 03 48 14" src="https://user-images.githubusercontent.com/118350020/217971417-43af59f7-fd6a-458c-ad4f-b136ebfb1795.png">

the next step to take is to use gdisk utility to create a single partition on each of the 3 disks
so we are going to use the below commands to get that done

sudo /dev/xvdf

<img width="781" alt="Screenshot 2023-02-09 at 04 01 28" src="https://user-images.githubusercontent.com/118350020/217973023-9af75b97-0cdc-43d9-9060-d273d4b28e29.png">

sudo /dev/xvdg

<img width="781" alt="Screenshot 2023-02-09 at 04 04 51" src="https://user-images.githubusercontent.com/118350020/217973812-a72d3603-abad-4d30-a3ae-4d60acce9f92.png">

sudo /dev/xvdh

<img width="781" alt="Screenshot 2023-02-09 at 04 07 45" src="https://user-images.githubusercontent.com/118350020/217973981-eaa35c82-d1ad-4283-a582-a69247be971a.png">

the next thing we are going to do now, is Install lvm2 package using the below commmand
sudo yum install lvm2 -y

<img width="778" alt="Screenshot 2023-02-09 at 04 12 32" src="https://user-images.githubusercontent.com/118350020/217974483-4f1a66dd-babb-4574-8e11-01fdb9f6159c.png">
<img width="780" alt="Screenshot 2023-02-09 at 04 13 16" src="https://user-images.githubusercontent.com/118350020/217974544-e745f4ab-cc77-447c-b54b-6d0edd4c55e7.png">

So now, let us run this command below to check for available partitions.
sudo lvmdiskscan  

<img width="773" alt="Screenshot 2023-02-09 at 04 18 47" src="https://user-images.githubusercontent.com/118350020/217974943-32ab7aa0-2ad4-400d-9b56-50b7e35d5a40.png">

The next step is to create a physical volume for each disks
And to create that, we are going use pvcreate utility to mark each 
of 3 disks as physical volumes (PVs) to be used by LVM

so we are going to run the below commands one after each other as shown in the diagram below

sudo pvcreate /dev/xvdf1
sudo pvcreate /dev/xvdg1
sudo pvcreate /dev/xvdh1

<img width="782" alt="Screenshot 2023-02-09 at 04 41 15" src="https://user-images.githubusercontent.com/118350020/217975760-ef707807-b54e-41ce-88fd-bc950ca93dd7.png">

now lets verify that our Physical volume has been created successfully by
running the command below and as shown in the diagram below as well

sudo pvs
<img width="783" alt="Screenshot 2023-02-09 at 04 44 47" src="https://user-images.githubusercontent.com/118350020/217976810-808b1c56-9d3c-4b02-ad78-c9b5d43d8206.png">
wow that works fine.

So let us use vgcreate utility to add all 3 PVs to a volume group (VG) and  Name the VG webdata-vg
we are going to run this command below now

sudo vgcreate webdata-vg /dev/xvdh1 /dev/xvdg1 /dev/xvdf1

<img width="783" alt="Screenshot 2023-02-09 at 04 50 54" src="https://user-images.githubusercontent.com/118350020/217978041-607beabd-891a-42f7-be1d-0bf7168d3332.png">

let us run sudo vgs and see what we just did as shown in the below diagram

<img width="780" alt="Screenshot 2023-02-09 at 04 53 29" src="https://user-images.githubusercontent.com/118350020/217979911-7e98f777-9079-4334-9247-41c226fdbd41.png">
thats good.

so right now, we are going to 3 Logical Volumes. lv-opt lv-apps, and lv-logs
so we are going to run this 3 commands below one after each other, as it will 
be shown in the diagram below

the next thing is to create 3 Logical Volumes. lv-opt lv-apps, and lv-logs
using the commands below and it will be run ,one after each other

sudo lvcreate -n 1v-apps -L 9G webdata-vg
sudo lvcreate -n 1v-logs -L 9G webdata-vg
sudo lvcreate -n 1v-opt -L 9G webdata-vg

<img width="782" alt="Screenshot 2023-02-09 at 05 03 37" src="https://user-images.githubusercontent.com/118350020/217983743-20093022-0973-4c6d-b957-2f5eb9e5c003.png">

So let us verify that our Logical Volume has been created successfully 
by running the command below and as shown in the diagram below as well

sudo lvs

<img width="782" alt="Screenshot 2023-02-09 at 05 06 16" src="https://user-images.githubusercontent.com/118350020/217985488-b9fe9d9e-a956-4cbc-ae42-56bc8e4d9776.png">

so now let us Verify the entire setup by running the command below

sudo vgdisplay -v #view complete setup - VG, PV, and LV

<img width="1440" alt="Screenshot 2023-02-09 at 05 39 13" src="https://user-images.githubusercontent.com/118350020/217986524-56f1004c-5518-414c-808f-a70e357e3804.png">

<img width="1440" alt="Screenshot 2023-02-09 at 05 39 19" src="https://user-images.githubusercontent.com/118350020/217986685-61a2c3a5-97e3-4043-bca8-566b6958268e.png">

The next step is to format disks as xfs not as ext4 
so we are going to run the below commands one after each other

sudo mkfs -t xfs /dev/webdata-vg/lv-apps

sudo mkfs -t xfs /dev/webdata-vg/lv-logs

sudo mkfs -t xfs /dec/webadat-vg/lv-opt

<img width="1440" alt="Screenshot 2023-02-09 at 06 12 49" src="https://user-images.githubusercontent.com/118350020/217987767-49d998f1-75cc-4c34-8bb4-241981b9e80c.png">

so next step is to create a mount point for lv-opt lv-apps, and lv-logs
so i will use the below commands for that

sudo mkdir /mnt/apps
sudo mkdir /mnt/logs
sudo mkdir /mnt/opt

as shown shown in the diagram below

<img width="779" alt="Screenshot 2023-02-09 at 06 21 41" src="https://user-images.githubusercontent.com/118350020/218323212-9a19b02c-1a5d-40fa-959b-125c0fff82d7.png">

so the next step is to mount the point
i will run the command below

sudo mount /dev/webdata-vg/lv-apps /mnt/apps
sudo mount /dev/webdata-vg/lv-logs /mnt/logs
sudo mount /dev/webdata-vg/lv-opt /mnt/opt

as shown in diagram below

<img width="779" alt="Screenshot 2023-02-09 at 06 34 02" src="https://user-images.githubusercontent.com/118350020/218323605-ffcb6b35-55a2-4a11-b980-19b5a1ca34a9.png">

The next step is to Install NFS server, and configure it to start on reboot and make sure it is up and running

first is to update our server by running this command
sudo yum -y update

why the update is going on the NFS ,lets try and connect of DB server and install mysql server on it
So am going to run this command now to install mysql on my DB Server
sudo apt install mysql-server -y 
but we need to first run an update on the DB server first,using this command
sudo apt update -y

as shown in the diagram below

<img width="766" alt="Screenshot 2023-02-13 at 02 41 22" src="https://user-images.githubusercontent.com/118350020/218352204-edb1f977-0858-4f9d-9047-0a15ae5bb419.png">

now let us install mysql server now
sudo apt install mysql-server -y 
as shown in the below diagram

<img width="763" alt="Screenshot 2023-02-13 at 02 47 53" src="https://user-images.githubusercontent.com/118350020/218352935-1e2d6624-64af-4e14-8ead-ec2cc00c57fc.png">

now let us run this command in the DB server
sudo mysql

<img width="766" alt="Screenshot 2023-02-13 at 03 30 24" src="https://user-images.githubusercontent.com/118350020/218357318-6412e5f6-0cd5-4015-82bb-34d757559175.png">

now we are going to Create a database and name it tooling and 
Create a database user and name it webaccess
as shown in the below diagram from our DB Server

<img width="761" alt="Screenshot 2023-02-13 at 04 01 27" src="https://user-images.githubusercontent.com/118350020/218360941-c6323cd0-aa39-4857-9df4-78203e34ded4.png">

so now let us go back to our NFS snd let us run the command below
sudo yum install nfs-utils -y as show in the diagram below

<img width="763" alt="Screenshot 2023-02-13 at 04 06 27" src="https://user-images.githubusercontent.com/118350020/218361523-5f37a023-9a56-4733-99e4-a7c03e6f7d21.png">

so we are going to start the nfs server, enable it and also check the status by running the below commands
sudo systemctl start nfs-server.service

sudo systemctl enable nfs-server.service

sudo systemctl status nfs-server.service



As shown in the below diagram, its active
<img width="764" alt="Screenshot 2023-02-13 at 04 15 04" src="https://user-images.githubusercontent.com/118350020/218362513-63664325-208a-433f-b859-af510efaca5a.png">

So let us set up permission that will allow our Web servers to read, write and execute files on NFS
we are going to run all this commands below, one after another
sudo chown -R nobody: /mnt/apps
sudo chown -R nobody: /mnt/logs
sudo chown -R nobody: /mnt/opt

sudo chmod -R 777 /mnt/apps
sudo chmod -R 777 /mnt/logs
sudo chmod -R 777 /mnt/opt

sudo systemctl restart nfs-server.service

as shown in the below diagram

<img width="757" alt="Screenshot 2023-02-13 at 04 24 01" src="https://user-images.githubusercontent.com/118350020/218363460-87eb79c1-a103-44e8-a890-8c9da63ccc34.png">

now lets us look at it status
sudo systemctl status nfs-server.service



<img width="765" alt="Screenshot 2023-02-13 at 04 27 35" src="https://user-images.githubusercontent.com/118350020/218363868-f519540e-fade-46ef-9138-1906fd92fbb1.png">
as shown in the above diagram, its still active


so now, let us Configure access to NFS for clients within the same subnet (example of Subnet CIDR – 172.31.32.0/20 ):
so we are to run this command below
sudo vi /etc/exports

as shown in the below diagram

<img width="762" alt="Screenshot 2023-02-13 at 04 36 12" src="https://user-images.githubusercontent.com/118350020/218364881-b7d7ef0f-bd43-4ead-87c2-6c5bb12439c0.png">


/mnt/apps 172.31.0.0/20(rw,sync,no_all_squash,no_root_squash)
/mnt/logs 172.31.0.0/20(rw,sync,no_all_squash,no_root_squash)
/mnt/opt 172.31.0.0/20(rw,sync,no_all_squash,no_root_squash)

so we are going to paste all this inside as shown in the below diagram

<img width="762" alt="Screenshot 2023-02-13 at 04 43 36" src="https://user-images.githubusercontent.com/118350020/218365799-10cf2638-36d7-40d7-92b5-d06887773578.png">

 so now let us run the below command
 sudo exportfs -arv
 
 <img width="761" alt="Screenshot 2023-02-13 at 04 47 29" src="https://user-images.githubusercontent.com/118350020/218366055-148b4f73-2f1b-49e3-a5a4-9ce6981cea66.png">


as shown above, it as exported

so now, let us Check which port is used by NFS and open it using Security Groups (add new Inbound Rule)
let us run the command below

rpcinfo -p | grep nfs

<img width="761" alt="Screenshot 2023-02-13 at 04 51 22" src="https://user-images.githubusercontent.com/118350020/218366528-a882b5da-2f23-47bc-a8e8-21af07cc1253.png">

Important note: In order for NFS server to be accessible from your client, you must also open following ports: TCP 111, UDP 111, UDP 2049
as shown in the below diagram

<img width="853" alt="Screenshot 2023-02-13 at 04 54 21" src="https://user-images.githubusercontent.com/118350020/218366830-0ee2ec06-0305-4d60-9a8d-ec3ecf9abfdb.png">
 <img width="1328" alt="Screenshot 2023-02-13 at 05 07 35" src="https://user-images.githubusercontent.com/118350020/218368298-c5d13bb6-ffce-41ec-857f-b74bc91e58f1.png">


