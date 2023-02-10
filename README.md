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

