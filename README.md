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

The next step is to go and configure our webserver

We need to make sure that our Web Servers can serve the same content from shared storage solutions, 
in our case – NFS Server and MySQL database.

So now let uus connect to our webserver1 as shown in the below diagram

<img width="907" alt="Screenshot 2023-02-21 at 17 47 31" src="https://user-images.githubusercontent.com/118350020/220408117-587c33e0-44fe-4a96-8a6f-db3e508c8eb8.png">

we already know that one DB can be accessed for reads and writes by multiple clients. For storing shared files that our Web Servers will use – we will utilize NFS and mount previously created Logical Volume lv-apps to the folder where Apache stores files to be served to the users (/var/www).

This approach will make our Web Servers stateless, which means we will be able to add new ones or remove them whenever we need, and the integrity of the data (in the database and on NFS) will be preserved.

During the next steps we will do following:

Configure NFS client (this step must be done on all three servers
Deploy a Tooling application to our Web Servers into a shared NFS folder
Configure the Web Servers to work with a single MySQL database

Launch a new EC2 instance with RHEL 8 Operating System which is our webserver 1

so we are going to

Install NFS client on it, using the below command

1: sudo yum install nfs-utils nfs4-acl-tools -y as shown in the below diagram


<img width="912" alt="Screenshot 2023-02-21 at 20 13 13" src="https://user-images.githubusercontent.com/118350020/220437736-bcbb680d-f9c0-4619-ab0f-6953bc119333.png">

so now we are going to Mount /var/www/ and target the NFS server’s export for apps
using the below command

2: sudo mkdir /var/www
3: sudo mount -t nfs -o rw,nosuid 172.31.14.17:/mnt/apps /var/www


<img width="914" alt="Screenshot 2023-02-21 at 20 21 18" src="https://user-images.githubusercontent.com/118350020/220439030-07d3e777-a281-4194-bfcc-28a6459447b3.png">

as we can see from the diagram above, there is no error, so it runs smoothly,
so now let us run this command below

 df -h 

<img width="911" alt="Screenshot 2023-02-21 at 20 24 19" src="https://user-images.githubusercontent.com/118350020/220439594-db2352d4-1e38-414a-ba49-b43731ef63e1.png">

as you can see from the diagram above, the highlited part is our NFS server

let us create a folder on it here and see, if it will show on our NFS server
am going to run this command now

 sudo touch /var/www//test.md

<img width="906" alt="Screenshot 2023-02-21 at 20 32 15" src="https://user-images.githubusercontent.com/118350020/220441130-321aec88-bf02-4bb5-aa28-ec011ba208da.png">

as you can see from above diagram, the folder as being created, now let us check our NFS server to confirmed that
i will ruun this command on it now.
ls /mnt/apps

<img width="910" alt="Screenshot 2023-02-21 at 20 35 39" src="https://user-images.githubusercontent.com/118350020/220441766-1d6dec34-5b9d-4986-a2a4-8ebd36012493.png">

as you can see from the above diagram, it works perfectly.

next now is to Make sure that the changes will persist on Web Server after reboot:
so we are going to run the below command on our webserver1 right now

4: sudo vi /etc/fstab and 
add the following line below inside it.

172.31.14.17:/mnt/apps /var/www nfs defaults 0 0
 
 as shown in the below diagram
 
 <img width="907" alt="Screenshot 2023-02-21 at 20 43 39" src="https://user-images.githubusercontent.com/118350020/220443226-9799d40c-0290-40f4-865f-44ee3ad53975.png">

<img width="906" alt="Screenshot 2023-02-21 at 20 47 58" src="https://user-images.githubusercontent.com/118350020/220444004-00bd4073-7b9e-4d4b-bcc3-9bd97b506299.png">

the next thing to do now is to

5: Install Remi’s repository, Apache and PHP by running the below command one after each other

sudo yum install httpd -y

sudo dnf install https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm

sudo dnf install dnf-utils http://rpms.remirepo.net/enterprise/remi-release-8.rpm

sudo dnf module reset php

sudo dnf module enable php:remi-7.4

sudo dnf install php php-opcache php-gd php-curl php-mysqlnd

sudo systemctl start php-fpm

sudo systemctl enable php-fpm

sudo setsebool -P httpd_execmem 1

as shown in the below diagrams

<img width="903" alt="Screenshot 2023-02-21 at 21 18 39" src="https://user-images.githubusercontent.com/118350020/220449851-8ebd7b8f-082e-401b-82c5-f290a1ba038e.png">

<img width="913" alt="Screenshot 2023-02-21 at 21 21 56" src="https://user-images.githubusercontent.com/118350020/220450340-ae9b479f-4996-4cbd-84be-50b8ebd9fe87.png">

<img width="909" alt="Screenshot 2023-02-21 at 21 24 32" src="https://user-images.githubusercontent.com/118350020/220450743-2e5b9030-8c2d-49ca-a0a9-636669a39093.png">

<img width="914" alt="Screenshot 2023-02-21 at 21 26 18" src="https://user-images.githubusercontent.com/118350020/220450927-b05fb946-89ad-406e-874f-41dc32836167.png">

<img width="914" alt="Screenshot 2023-02-21 at 21 26 18" src="https://user-images.githubusercontent.com/118350020/220451097-d7e57068-33e4-4151-83d1-c0a99775d0f6.png">

<img width="916" alt="Screenshot 2023-02-21 at 21 28 16" src="https://user-images.githubusercontent.com/118350020/220451307-74f71ac7-ecea-414c-aba2-bdb5d07398e5.png">

<img width="910" alt="Screenshot 2023-02-21 at 21 30 04" src="https://user-images.githubusercontent.com/118350020/220451629-46d73fdc-7f44-44fc-b59f-0de2334a2af4.png">

So we are going to Repeat steps 1-5 for another 2 Web Servers.
After that is done.

we are now going to Verify that Apache files and directories are available on the Web Server in /var/www and also on the NFS server in /mnt/apps. 
If you see the same files – it means NFS is mounted correctly. 
You can try to create a new file touch test.md

from one server and check if the same file is accessible from other Web Servers.

now let  us run this command below
ls /var/www

As you can see from the diagram above, cgi bin html was created while installing the apache

we can as well confirmed that in our NFS server by running this command below
ls /mnt/apps

<img width="907" alt="Screenshot 2023-02-21 at 22 38 57" src="https://user-images.githubusercontent.com/118350020/220464716-0dbb28ad-f55e-41a6-9433-e64226d01b33.png">

as we can see from the above diagram, cgi bin was also created inside NFS server as well

so now, we  are going to Locate the log folder for Apache on the Web Server and mount it to NFS server’s export for logs.
And Repeat step 4 to make sure the mount point will persist after reboot

Now let us go back to our webserver1 to 3 and run the command below
ls /var/log


<img width="912" alt="Screenshot 2023-02-21 at 22 47 49" src="https://user-images.githubusercontent.com/118350020/220466674-6ac69467-7fc0-4e11-ab13-9f45bce75581.png">
<img width="909" alt="Screenshot 2023-02-21 at 23 19 37" src="https://user-images.githubusercontent.com/118350020/220471622-231298ec-3ce5-47ee-9348-b0e2c36d7633.png">

as you can see, there is nothing in that log folder, so now am going to run this command below but this time around is going to log not apps

sudo mount -t nfs -o rw,nosuid 172.31.14.17:/mnt/logs /var/log//httpd

As shown in the below diagram

<img width="909" alt="Screenshot 2023-02-21 at 23 27 03" src="https://user-images.githubusercontent.com/118350020/220473046-d38e2854-48ac-4ccb-b97e-a5d0a0b2bc85.png">

so now am going to run the below command all the 3 webserver
sudo vi /etc/fstab and input this  content inside 172.31.14.17:/mnt/logs /var/log//httpd nfs defaults 0 0
as shown in the below diagram

<img width="911" alt="Screenshot 2023-02-21 at 23 34 49" src="https://user-images.githubusercontent.com/118350020/220474408-835991bb-ae06-4917-9099-dc31ff2d0e40.png">

<img width="906" alt="Screenshot 2023-02-21 at 23 35 31" src="https://user-images.githubusercontent.com/118350020/220474460-24ea350a-01e1-434e-b5dc-04525854c4d1.png">


So now, let us Fork the tooling source code from Darey.io Github Account to your Github account.

but firslty we need to install git on our 3 webserver by running this command below
sudo yum install git -y
as you can see from the below diagram

<img width="906" alt="Screenshot 2023-02-22 at 00 01 04" src="https://user-images.githubusercontent.com/118350020/220478474-1d4f7a17-e07f-4d03-b38a-cb2325664704.png">
<img width="915" alt="Screenshot 2023-02-22 at 00 01 54" src="https://user-images.githubusercontent.com/118350020/220478531-65f58962-f1db-41a0-9d75-82b1badbee97.png">

next is to run this command below on all the 3 webserver
git init 
As shown in the diagram below

<img width="907" alt="Screenshot 2023-02-22 at 00 10 44" src="https://user-images.githubusercontent.com/118350020/220479996-659d4e7e-76ee-4078-8928-8ab1b4d2ec84.png">

so now we are going to run the below command on all the 3 webserver

git clone https://github.com/darey-io/tooling.git

as shown in the below diagram

<img width="911" alt="Screenshot 2023-02-22 at 00 16 04" src="https://user-images.githubusercontent.com/118350020/220480451-693bd31a-c31f-4bb3-ad46-4e39ff2af32a.png">

so now let us run ls command as shown in the below diagram

<img width="910" alt="Screenshot 2023-02-22 at 00 39 20" src="https://user-images.githubusercontent.com/118350020/220483513-9023fc59-293c-46c7-a7cb-9a95300b9999.png">


now let us run ls tooling as shown in the below diagram

<img width="908" alt="Screenshot 2023-02-22 at 00 41 26" src="https://user-images.githubusercontent.com/118350020/220483819-457ceba5-3a82-45e7-ac94-b752345bb51d.png">

so let us run this command cd tooling and also ls, as seen in the below diagram

<img width="912" alt="Screenshot 2023-02-22 at 00 44 06" src="https://user-images.githubusercontent.com/118350020/220484245-990b4ae2-dbdc-4d2e-aa7d-ddb28afe9b2a.png">

Next step is to Deploy the tooling website’s code to the Webserver. 
And Ensure that the html folder from the repository is deployed to /var/www/html

so we are going to run this command below on all the 3 webserver

ls /var/www
As shown in the below diagram, we have the html inside the tooling

<img width="908" alt="Screenshot 2023-02-22 at 00 49 36" src="https://user-images.githubusercontent.com/118350020/220485074-e7163948-1da0-4099-8344-2d66a5d5497b.png">

so now, let us run this command below on all the 3 webserver

sudo cp -R html/. /var/www/html as shown in the below diagram

<img width="912" alt="Screenshot 2023-02-22 at 00 55 54" src="https://user-images.githubusercontent.com/118350020/220485736-c113b126-fe7a-49d7-af24-b356fa1653da.png">

so am going to ruun this command below
ls /var/www/html 

As shown in the below diagram

<img width="910" alt="Screenshot 2023-02-22 at 01 20 31" src="https://user-images.githubusercontent.com/118350020/220488788-81412de7-865a-4a9e-9566-95717ac0fde0.png">

now let us go into our  on EC2 instant to open TCP port 80 on all the Web Server

let us run this command

cd ..

now let us run the command below on all the 3 webserver

sudo setenforce 0

as shown in the diagram below

<img width="908" alt="Screenshot 2023-02-22 at 01 42 11" src="https://user-images.githubusercontent.com/118350020/220491304-e77e0cca-eb04-4f04-b360-9a4b9c139e03.png">
 
 Next step is to run the command below and go into the file as it will be shown in the diagram below
 
 sudo vi /etc/sysconfig/selinux
 
 
 <img width="906" alt="Screenshot 2023-02-22 at 01 47 40" src="https://user-images.githubusercontent.com/118350020/220492111-bc17f916-6750-4ea3-ad10-85f64098767a.png">

so the highlighted part will be changed to disabled as shown in the below diagram

<img width="907" alt="Screenshot 2023-02-22 at 01 51 04" src="https://user-images.githubusercontent.com/118350020/220492419-f7391950-a509-403f-9040-dbc946c2e8ff.png">

so now let us start our httpd by running the below command
sudo systemctl start httpd
sudo systemctl status httpd

as shown in the below diagram, its active

<img width="912" alt="Screenshot 2023-02-22 at 01 56 46" src="https://user-images.githubusercontent.com/118350020/220493254-e04b4486-edb4-434f-ad8e-2ddb8981d53c.png">

now let us copy the public IP on the webserver and paste it on our web search to see the outcome as shown in the below diagram

<img width="1091" alt="Screenshot 2023-02-22 at 02 09 28" src="https://user-images.githubusercontent.com/118350020/220494881-58385ff1-60ac-42fb-aa09-75aee502f6c5.png">


So next step is to Update the website’s configuration to connect to the database (in /var/www/html/functions.php file).
so we are running this command below

sudo vi /var/www/html/functions.php  as shown in the below diagram

<img width="915" alt="Screenshot 2023-02-22 at 02 25 47" src="https://user-images.githubusercontent.com/118350020/220497031-29ce02cb-5807-4db4-87b9-4a33440e4fd0.png">
 
 so the highlighted part will be edited by inputting the below content
 
 ('172.31.15.65', 'webaccess', 'allo', 'tooling');
 
 as shown in the below diagram
 
 <img width="908" alt="Screenshot 2023-02-22 at 02 28 38" src="https://user-images.githubusercontent.com/118350020/220497362-ed719c41-cde5-439b-8bc0-d64e6ead7f5e.png">

so now, we are going to Apply tooling-db.sql script.
but first, we need to install mysql on all the 3 webserver by running the below command
you need to change your directory to tooling, using the below command

cd tooling/ so now, you can install your mysql

sudo yum install mysql -y 

as shown in the below diagram

<img width="911" alt="Screenshot 2023-02-22 at 02 45 14" src="https://user-images.githubusercontent.com/118350020/220499605-f71ec4e4-9667-4f8b-9349-60c1646fa98a.png">

on our DBserver, we need to ruun this command below
sudo vi /etc/mysql/mysql.conf.d/mysqld.cnf

as shown in the below diagram 


<img width="906" alt="Screenshot 2023-02-22 at 03 30 30" src="https://user-images.githubusercontent.com/118350020/220505816-cc12d2bf-abe4-44d8-899f-bd81cd6eebb1.png">

so we have to changed this bind-address            = 127.0.0.1 
                           mysqlx-bind-address     = 127.0.0.1
                           
                           to
                           
                           bind-address            = 0.0.0.0
                           mysqlx-bind-address     = 0.0.0.0

as shown in the diagram below

<img width="909" alt="Screenshot 2023-02-22 at 03 35 55" src="https://user-images.githubusercontent.com/118350020/220506683-5494b02c-75e6-4e76-b027-7d0a45079545.png">

so we need to restart our mysql on the DB server using the below command
sudo systemctl restart mysql
sudo systemctl status mysql

<img width="918" alt="Screenshot 2023-02-22 at 03 39 37" src="https://user-images.githubusercontent.com/118350020/220507260-0f64e04a-3432-4139-bda2-d9f0f67e197e.png">

so now let us go back to our webserver to run this below command
mysql -h 172.31.15.65 -u webaccess -p tooling < tooling-db.sql

<img width="905" alt="Screenshot 2023-02-22 at 03 57 15" src="https://user-images.githubusercontent.com/118350020/220510096-bb65be24-b371-4230-bc14-8bf7bfd7409f.png">
  so now lets go back to our DB server and run the below commnad
  sudo mysql
  
  as shown in the below. diagram
  
  <img width="1440" alt="Screenshot 2023-02-22 at 04 00 55" src="https://user-images.githubusercontent.com/118350020/220510598-7873decd-2a30-4af6-8e25-98dc3c0fbc56.png">
  
so let us this command below

show databases;

<img width="911" alt="Screenshot 2023-02-22 at 04 04 30" src="https://user-images.githubusercontent.com/118350020/220510909-8e364d7c-a1f7-4bb8-a140-27b350a61d2e.png">

use tooling;
show tables;
select * from users;
as shown in the below diagram

<img width="908" alt="Screenshot 2023-02-22 at 04 12 22" src="https://user-images.githubusercontent.com/118350020/220511963-e63a417d-1a66-44e0-bb13-d008a8f33fff.png">
 
 if we run this command below on our webserve, we are still going to see all the content in our mysql DB server 
 
 sudo vi tooling-db.sql 
 as youu can see from the below diagram
 
 <img width="912" alt="Screenshot 2023-02-22 at 04 18 15" src="https://user-images.githubusercontent.com/118350020/220512776-ba3e2424-37f4-4a1d-84be-b6fd837ae29b.png">

now let uus run the below command on our webserver to find our test page

ls /etc/httpd/conf.d/welcome.conf

as shown in the diagram below

<img width="912" alt="Screenshot 2023-02-22 at 04 39 08" src="https://user-images.githubusercontent.com/118350020/220515533-3906fb4f-8e77-4171-a57e-f8bf9817c3e0.png">

if i should run this below command on our webserver, it will show the below diagram as well

vi /etc/httpd/conf.d/welcome.conf

<img width="979" alt="Screenshot 2023-02-22 at 04 48 10" src="https://user-images.githubusercontent.com/118350020/220516910-0d1634de-22b3-46fe-b6ea-c503c61f99d6.png">

now let uus rename this vi /etc/httpd/conf.d/welcome.conf to this 

sudo mv /etc/httpd/conf.d/welcome.conf /etc/httpd/conf.d/welcome.backup
sudo systemctl restart httpd
sudo systemctl status httpd

<img width="980" alt="Screenshot 2023-02-22 at 04 58 58" src="https://user-images.githubusercontent.com/118350020/220518417-aaf2888a-df83-4fce-8c11-7afd87323bb9.png">


 now let us launch back out web
 
 <img width="1091" alt="Screenshot 2023-02-22 at 02 09 28" src="https://user-images.githubusercontent.com/118350020/220519537-77cb8654-2cb2-40bc-825b-fbb29eac7ab0.png">

now let me login using my credential

<img width="1440" alt="Screenshot 2023-02-22 at 05 07 06" src="https://user-images.githubusercontent.com/118350020/220519624-9a56ada0-9825-49c1-9760-39a635a65efa.png">

<img width="1440" alt="Screenshot 2023-02-22 at 05 07 25" src="https://user-images.githubusercontent.com/118350020/220519672-8609fbc1-fc80-4ac6-b286-5f383ea6ca21.png">


end of project 7
 





 



