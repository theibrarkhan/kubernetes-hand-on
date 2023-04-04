# Setup Guide

This guide was written as a support for the different Kubernetes courses that I'm teaching. This guide does not intend to show all possible solutions but covers a few options to use Kubernetes. Other options do exist, but I cannot support them in my courses. For questions:

version 3.1, 23 March 2022: added Ubuntu-only update for Minikube based on Docker. Phasing out Centos as primary platform.

# Setting up Minikube on a Linux virtual machine

Because of ongoing issues with Minikube on MacOS / VMware combination, I have developed a new procedure that works with Docker containers and is only supported on Ubuntu 20.04.

**WARNING: you cannot copy paste from this document, but will have to type all commands.**

1.  Make sure you have either a fully installed Ubuntu 20.04 (virtual) machine.

    1.  8 GB RAM recommended, 4 GB RAM minimal

    2.  2 CPU cores

    3.  20 GB minimal disk space

2.  Install git, vim and bash completion

    1.  On Ubuntu: **sudo apt install git vim**

3.  As ordinary user with sudo privileges, clone the course Git repository

    1.  **git clone https://github.com/sandervanvugt/kubernetes** for Kubernetes in 4 Hours or any other course

    2.  **git clone https://github.com/sandervanvugt/ckad** for the CKAD course

4.  Change into the cloned git repo and run the **minikube-docker-setup.sh** script

    1.  **cd kubernetes**

    2.  **./minikube-docker-setup.sh**

5.  When you see the message that the script is ready, manually run the following command:

    1.  **minikube start --memory=4096 --vm-driver=docker**

6.  Once Minikube has started successfully, you'll see a message that it has started. Type **kubectl get all** to verify the minikube cluster is up and running

# Setting up AiO-Kubernetes

This method is less common but offers the advantage that no nested virtualization is required. Also it gives nice insight in the workings of Kubernetes.

Notice that this procedure only is supported on Ubuntu 20.04 workstation and server. Support for CentOS 7 has been dropped and removed from this guide.

**WARNING: you cannot copy paste from this document, but will have to type all commands.**

1.  Install Ubuntu 20.04 workstation or server, meeting the following minimal requirements

1.  20 GB disk space

1.  4 GB RAM recommended (2 GB minimal)

1.  2 CPU's

1.  No Swap

1.  One ordinary user with sudo privileges must be present. In this document I'll assume the username "student". Change this according to your local setup.

1.  Install some packages

a.**sudo apt install git vim -y**

1.  As ordinary user, clone the course Git repository that is required for the course you are taking

1.  **git clone https://github.com/sandervanvugt/kubernetes** for Kubernetes in 4

Hours

1.  **git clone https://github.com/sandervanvugt/ckad** for CKAD

2.  **git clone https://github.com/sandervanvugt/microservices** for

Microservices

1.  **git clone https://github.com/sandervanvugt/cka** for CKA

1.  Run the setup scripts using root privileges

1.  **cd /ckad** (or **cd /cka**) (or whichever GitHub repository you have cloned)

1.  **sudo ./setup-container.sh**

1.  **sudo ./setup-kubetools-ubuntu.sh**

1.  Still from a root shell, install a Kubernetes master node

a.**sudo kubeadm init --pod-network-cidr=10.10.0.0/16**

1.  Everything from this point is done in a user shell. Set up the kubectl client: a.**cd ~**

1.  **mkdir .kube**

1.  **sudo cp -i /etc/kubernetes/admin.conf .kube/config**

1.  **sudo chown student:student .kube/config**

1.  Set up the Calico networking agent

1.  **kubectl create -f https://docs.projectcalico.org/manifests/tigeraoperator.yaml**

1.  **wget https://docs.projectcalico.org/manifests/custom-resources.yaml**

1.  You now need to define the Pod network, which by default is set to

    192.168.0.0/24, which in general is a bad idea. I suggest setting it to 10.10.0.0 - make sure this address range is not yet used for something else!

1.  **sed -i -e s/192.168.0.0/10.10.0.0/g custom-resources.yaml**

1.  **kubectl create -f custom-resources.yaml**

1.  **kubectl get pods -n calico-system**: wait until all pods show a state of Ready, this can take about 5 minutes!

1.  By default, user Pods cannot run on the Kubernetes control node. Use the following command to remove the taint so that you can schedule nodes on it: **kubectl taint nodes --all node-role.kubernetes.io/master-** 9.Type **kubectl get all** to verify the cluster works.

# Using Hosted Kubernetes in GCE

Kubernetes is supported by many cloud providers. As I cannot be all-inclusive, but most of all, as Google has donated Kubernetes to the world, I like to do something back and explain the procedure on GCE only.

1.  From a browser, go to **https://cloud.google.com** and click **Sign in**.

2.  Click **Go to console**

3.  Select **Compute > Kubernetes Engine > Clusters**

4.  Click **Create**

5.  Select **GKE Standard** and click **Configure**

6.  In the upper right corner, select **My first cluster**

7.  Click **Create Now** and wait a few minutes - typically not more than 5.

8.  Once the cluster appears, click **Connect**

9.  You'll see a command **gcloud container ...** Click **Run in Cloud Shell** to run it. This will deploy a cloud shell machine, which may take one or two minutes.

(NOTE: If you don't see the command, this is what you need to type in a cloud shell: **gcloud containers cluster get-credentials cluster-name --zone us-central1-c -project your-project-123**. Make sure the change the cluster name, project name as well as the zone to what applies to your environment.

1.  Press Enter to run the command, and click **Authorize** to authorize the cloud shell

2.  Type **kubectl get all** to check that the Kubernetes cluster works.
