
# Introduction

    In networking research we sometimes need to use Virtual Machines (VMs) to achieve a specific goal, e.g., creating a topology and emulating multiple nodes with a single host, separation of Host/Guest operating system, for example when developing kernel a module, etc.
    VMs also make it easy to share code along with its dependencies and to ease code reproducibility and longevity.
    This document covers the approach to VM management that we found to work best along with notes for some of the components.

## Management Software

    To manage VMs we use [Vagrant](https://www.vagrantup.com/).
    Vagrant is a tool that automates VM creation, manage, and deletion. 
    From the Vagrant docs:
    > Vagrant isolates dependencies and their configuration within a single disposable, consistent environment, without sacrificing any of your existing tools.
    To achieve this Vagrant uses Vagrantfiles.
    A Vagrantfile contains all the creation, provisioning, and setup instructions for the VM.
    
    You can find out more about Vagrant, how to install it, and manage VMs [here](https://developer.hashicorp.com/vagrant/tutorials/getting-started).


## Hypervisors

    To create a VM Vagrant needs a hypervisor. 
    A hypervisor is a piece of software that allows virtualization of resources and enables running virtual machines on a physical machine.

    My prefered hypervisor is [Qemu](https://www.qemu.org/).
    Qemu requires the [libvirt](https://libvirt.org/) provider to work with Vagrant.
    The vagrant tutorial specifies how to setup and install [Virtualbox](https://www.virtualbox.org/) as a hypervisor.
    The following explains my motivation for using qemu over Virtualbox.


### Using Qemu over Virtualbox
    
    For my specific usecase, using mininet within a VM to create a network topology, I found that using Virtualbox (6.1.x) creates latency variation within the mininet nodes.
    On the other hand, qemu with libvirt has shown consistent, expected, network latency with little to no deviation within the mininet nodes.
    As my work relies on accurate network RTT samples, I chose to use quemu over Virtualbox.

    I also found that sets up the VM-Host networking differently.
    Virtualbox sets up ssh access to the host machine by port forwarding port 22 on the VM to a port on the host.
    Qemu sets up a virtual network, assigns a private IP to the VM and uses that address and port 22 for secure shell access.
    This is a small quality of life feature which makes it easier to connect multiple VMs on one host without having to deal with port colision or manually dealing with port forwarding to instruct which host port should be used for which VM.

### Setting up Qemu and Libvirt with Vagrant

    The following [link](https://opensource.com/article/21/10/vagrant-libvirt) aids how to install libvirt and qemu on Debian- and RedHat-based distributions.
    For MacOS, both qemu and libvirt are available to install through homebrew.

    The vagrant-libvirt plugin is needed to start qemu and libvirt.
    To install the vagrant-libvirt you need to use ``vagrant plugin install vagrant-libvirt``.

# Sample Vagrantfile    

    A sample Vagrantfile is provided.
    It sets up an Ubuntu 20.04 VM from the generic hashicorp repository. 
    This VM does not come with any desktop or graphics packages. 
    These need to be installed separately if required.
    The sample Vagrantfile also has a section which shows how X11 forwarding can be enabled, 
    though use of X11 is not recommended as it can be very slow over the network.

## Using Multiple Providers

    If you have more than one vagrant provider you may need to specify the provider when issuing using up, e.g., ``vagrant up --provider=libvirt``.
    Ryo found that you can add the following [line]() to the vagrantfile to force use of a specific driver and provider (e.g., qemu and libvert).
     

## Shared Directories and moving data between the VM and the Host
    
    The provided sample Vagrantfile sets up a shared directory `/vagrant` at the VM and the directory containing the Vagrantfile `.` at the host.
    Using shared folders we encountered **many issues** with syncronization, host vs guest CPU clocks, 
    permissions, and others.
    The provided share folder uses NFS (version 4) to syncronize the files and forces use of TCP.
    This has been the most reliable method we have found to work.
    Setting host file file permission mask may also be required to ensure that files have the correct permissions.

    In general, we discourage the use of shared folders and suggest that all data genererated by the VM is kept in the VM while data generation occurs (e.g., single instance of an experiment run).
    After that the data may be transfered to the host, for example, using rsync.
    Using rsync is also easier to automate when qemu is used because of the network setup that qemu uses by default.

