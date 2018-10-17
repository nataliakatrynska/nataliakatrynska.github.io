# Setting up a Local Virtual Environment for the OpenShift Platform

To freely develop the OpenShift platform and build on top of it other applications, you need to have access to a local environment. Since you use Windows as an operating system and the platform runs on RedHat Linux, the idea is to set up a virtual environment with the platform that reflects the real one on Amazon Web Services. You are going to use a set of tools that help you to aim this goal - all of them are listed below.

This guide contains of two steps where the first one instructs you how to build a virtual machine and the second part presents how to make the virtual machine running with the OpenShift platform installed.

## Basic Terms

Below you can find a list of the terms / plugins / applications mentioned in this guide: 

  - **CentOS**: A free RedHat Linux compatible image
  -- [CentOS official website](https://www.centos.org/)
  - **Local DEV cluster**: A local environment that is a mirror of the remote Amazon Web Services (AWS) dev cluster with the OpenShift platform installed. The cluster enables you to commit required changes and give you full control while developing. 
  - **OpenShift** (the OpenShift platform): The platform installed on a remote AWS cluster.
  - **Remote DEV cluster**: A remote Amazon Web Services (AWS) dev cluster with the OpenShift platform installed.
  - **Vagrant** (version 2.0.2): Open-source software that enables you to build and maintain portable virtual environments, for example for VirtualBox.
  -- [HashiCorp Vagrant 2.0.2 official website](https://www.hashicorp.com/blog/hashicorp-vagrant-2-0-2)
  - **Vagrant image** (Vagrant boxes): A base Vagrant image that allows to quickly clone a virtual machine.
  - **VirtualBox** (version 5.2.12): A virtual machine provider
  -- [VirtualBox official website](https://www.virtualbox.org/)

## Prerequisites
To run the OpenShift platform on a virtual machine, you will need the following tools installed:

  - **Apache Maven**
  - **Cygwin**
  - **Git**
  - **Maven**
  - **Nugrant**: The installation details are provided in this guide.
  - **PowerShell version 4.0 or higher**: At least 4.0 version is required since PowerShell is internally used by Vagrant and all lower versions make Vagrant freeze. The installation details are provided in this guide.
  - **Vagrant 2.0.2 / Vagrant 2.1.1**: The installation details are provided in this guide.
  - **Vagrant image 1.0.6**
  - **VirtualBox 5.2.12**: Note that Guest Additions are propagated to the image from your local VirtualBox therefore, it is required to have the proper version of VirtualBox installed.
  - **Windows 7 SP1**
 
## Step 1: Building a CentOS Linux Virtual Machine with the OpenShift Platform

To build a Linux virtual environment for the OpenShift platform, follow the steps:

> **Note**: Keep in mind that you need to perform all commands from your root directory unless otherwise specified.

1. Install Packer:
    1. Download and unzip the correct package from the Packer website: 
        - For general information, go to the [Packer official website](https://www.packer.io)
        - To learn more how to install Packer, go to the [Packer installation procedure](https://www.packer.io/intro/getting-started/install.html)
        
    2. Add the Packer directory to the `PATH`; see the [instruction on setting it on Windows](https://stackoverflow.com/questions/1618280/where-can-i-set-path-to-make-exe-on-windows) 
 
2. You need to have PowerShell, version 4.0 or higher, installed:

    1. Open PowerShell and type:
        ```sh
        $PSVersionTable.PSVersion
        ```
        to determine the version.

    2. If need be, upgrade PowerShell. Go to the [Microsoft Windows Management Framework 4.0](https://www.microsoft.com/en-us/download/details.aspx?id=40855) page to download the package; you can also find the detailed installation instruction there.

3. Check out the `box-build` sources from the [Git repository project](http://git.server.com/box-build.git).
4. Find the `centos-base-variables.json` file and edit it. Provide the credentials for the proxy server you use:
    ```sh
    "proxy": "http://PROXY_USER:PROXY_PASSWORD_HERE@proxy.server.com:80/
    ```
    > **Note**: Keep in mind that you need to encode non-alphanumeric characters.

5. Double check whether all files in the project contain UNIX-style line endings. If not, you need to convert them to UNIX format line endings if you want to use them on the virtual machine.

6. Open Cygwin:

    1. Go to your project's directory.
    2. Set an environmental variable:
        ```sh
        export ISO_URL=”YOUR_ISO_URL”
        ```
        where:
    
        `YOUR_ISO_URL` must point to either the URL with the CentOS ISO image or to its location on a disk, for example `C:\\centos-image.iso`.
        >**Note**: 
        > The [CentOS-7-x86_64-DVD-1804.iso](http://isoredirect.centos.org/centos/7/isos/x86_64/CentOS-7-x86_64-DVD-1804.iso) (DVD) version is required.
        > This is a Linux virtual machine, so keep in mind to escape backslashes with a backslash.

    3. Execute the following command:
        ```sh
        packer build -var-file=centos-base-variables.json -force centos-base.json
        ```
    4. Once the Packer build process has been completed, you should find the `centos-base.box` file in the current directory.
    5. Find the `centos-openshift-variables.json` file and edit it. Provide the credentials for the proxy server you use:
        ```sh
        "proxy_user": "PROXY_USER", 
        "proxy_pass": "PROXY_PASSWORD"
        ```
    6. Open a Cygwin terminal and go to the project’s directory.
    7. Execute the following command:
        ```sh
        packer build -var-file=centos-openshift-variables.json -force centos-openshift.json
        ```
    8. Once the Packer process has been completed, you will find the `centos-7-openshift.box` file.
7. Upload the `centos-7-openshift.box` file to the [Maven repository](http://maven.repository.com). Make sure that the credentials are configured for a server with the `user-upload` ID in your `~/.m2/settings.xml` file.
    1. Execute the following command:
        ```sh
        mvn deploy:deploy-file -DgroupId=com.example -DartifactId=vagrant-centos-open-shift -Dversion=1.0.7 -Dpackaging=box -Dfile=centos-7-openshift.box -DrepositoryId=user-upload -Durl=http://maven.repository.com/content/repositories/releases
        ```
8. After successfully uploading the file, create a Git tag with the deployed version: 1.0.X.
9. Update `-Dversion` from the above command (step 7a.) according to the pattern 1.0.X+1 and push the file.

## Step 2: Running the OpenShift Platform on Your Virtual Machine

Once the virtual machine has been built, you need to run the local virtual machine. To do so, perform the following steps:

> **Note**: Keep in mind that you need to execute all Vagrant commands in the root directory of the project.

1. Install Vagrant:
    1. Download the package from the Vagrant website: 
        * For general information, go to the [Vagrant official website](https://www.vagrantup.com/)
        * To learn more how to install Vagrant, go to the [Vagrant installation procedure](https://www.vagrantup.com/docs/installation/)
    2. Add the Vagrant directory to the `PATH`; see the [instruction on setting it on Windows](https://www.vagrantup.com/docs/other/environmental-variables.html).
    > **Note**: You should install the 2.1.1 version of Vagrant and the 1.0.6 version of the Vagrant image.
2. Install Nugrant:
    1. Go to the [Nugrant Github repository](https://github.com/maoueh/nugrant). To learn more how to install the Nugrant plugin, go to the [Nugrant installation procedure](https://github.com/maoueh/nugrant#installation).
    2. Set `http_proxy` and `https_proxy` environment variables.
        ```sh
        export http_proxy=http://PROXY_USER:PROXY_PASSWORD_HERE@proxy.server.com:80/
        export https_proxy=http://PROXY_USER:PROXY_PASSWORD_HERE@proxy.server.com:80/
        ```
        > **Info**: If your password contains non-alphanumeric characters, you need to encode it before exporting variables. Use this site to encode your password and put the value instead of the `PROXY_PASSWORD_HERE` string.

    3. Execute the following command:
        ```sh
        vagrant plugin install nugrant
        ```
    Once both, Vagrant and Nugrant, are installed, you can use either PowerShell, cmd, or Cygwin when working with Vagrant.
    
    > **Info**: If you do not have a private key generated, you can use `PuTTYgen.exe` to generate one. To do so, follow the [key generator instruction](https://www.ssh.com/ssh/putty/windows/puttygen). Note that the private key shouldn't be password protected.
    
    - If you use Cygwin you need to have a private key in `~/.ssh/id_rsa`.
    - If you use PowerShell or cmd, you need to have a private key in `%USERPROFILE%\.ssh\id_rsa`.
3. Check out the `box-run` sources from the [Git project](http://git.server.com/box-run.git).
4. Edit `.vagrantuser` file to specify whether to:
    1. Use CNTLM to get guest access
    2. Limit of memory and CPUs assigned
    3. Install additional software
    4. Start GUI
    
    > **Note**: If you run the image for the very first time or the global password has changed export the following environment variables: `PROXY_USER` and `PROXY_PASS`. This can be done only once as the proper configuration file will be kept in the virtual machine. Do not HTML encode password if it contains non-alphanumeric characters but you should wrap it with double quotes.
5. Before running any command, double check whether all files in the project contain UNIX-style line endings. If not, you need to convert them to UNIX format line endings.
6. Unset `http_proxy` and `https_proxy` environment variables.
7. Add a box (the image) to the Vagrant registry:
    1. Open a terminal and go the project’s directory.
    2. Execute the following command:
        ```sh
        vagrant box add metadata.json
        ```
        > **Note**: 
        > You can run the command only if there is no image in the local repository or a new version of the image is required.
        > If you already have the image in the repository and want to replace it, first of all, remove it:
        > ```sh
        > vagrant box remove centos-7-openshift.box
        > ```
        > and then, run the command to add the new image.
        
8. Check if the image has been added to the Vagrant registry:
    ```sh
    vagrant box list
    ```
9. When the box is successfully deployed in the Vagrant registry, you can run the virtual machine by executing the following command:
    ```sh
    vagrant up
    ```
10. When the virtual machine is up and running, you can ssh into the box using:
    ```sh
    vagrant ssh
    ```
    > **Info**: When you stop working with the instance and want to halt the virtual machine, execute the command:
    > ```sh
    > vagrant halt
    > ```
