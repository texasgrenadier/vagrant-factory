Create a CentOS 6 Vagrant Base Box from Scratch Using VirtualBox

These instructions describe the steps for building a vagrant box file from
scratch using VirtualBox. As written, these steps will create a CentOS 6.6
system. There are optional steps used to install the VirtualBox Guest
Additions. Other versions of the OS can be created by using a different iso
file.

These instructions are based on instructions found at the following sites:
http://thornelabs.net/2013/11/11/create-a-centos-6-vagrant-base-box-from-scratch-using-virtualbox.html
https://gist.github.com/fernandoaleman/5083680
http://xmodulo.com/2013/07/how-to-install-virtualbox-guest-additions-for-linux.html

The output of this process is a Vagrant box file. That file can be distributed so
others can use it. Place it in a location where other team members can access it.

The following steps were written for VirtualBox 4.3.20 and may differ for other versions.

Step 1: Download Virtual Box (4.3.20)

  https://www.virtualbox.org/wiki/Downloads

Step 2: Download Vagrant (Version 1.6.5)

  https://www.vagrantup.com/download-archive/v1.6.5.html

Step 3: Download CentOS 6.5 (minimal version)

  http://cosmos.cites.illinois.edu/pub/centos/6.6/isos/x86_64/
  CentOS-6.6-x86_64-minimal.iso

Step 4 (optional): Download guest additions

  If the guest additions are needed. Download the version that matches
  the version of VirtualBoxcorresponding to the version of VirtualBox from here:

  http://download.virtualbox.org/virtualbox/

  VBoxGuestAdditions_4.3.16.iso

Step 5: Create the virtual machine in VirtualBox

  Open VirtualBox and click New.
  Give the virtual machine a Name: centos-6.5-x86_64.
  From the Type dropdown menu choose Linux.
  From the Version dropdown menu choose Red Hat (64 bit).
  Under Memory size, leave RAM at 512 MB (Vagrant can change this on-the-fly later).
  Under Hard drive, select Create a virtual hard drive now, and click Create.
  Under File location, leave the default name.
  Under File size, change the size to 40.00 GB.
  Under Hard drive file type, select VDI (VirtualBox Disk Image).
  Under Storage on physical hard drive, select Dynamically allocated, and click Create.
  The virtual machine definition has now been created. Click the virtual machine name and click Settings.
  Go to the Storage tab, click Empty just under Controller: IDE, then on the right hand side of the window click the CD icon, and select Choose a virtual CD/DVD disk file….
  Navigate to where the CentOS-6.5-x86_64-bin-DVD1.iso was downloaded, select it, and click Open.
  Go to the Audio tab and uncheck Enable Audio.
  Go to the Ports tab, then go to the USB subtab, and uncheck Enable USB Controller.
  Click Ok to close the Settings menu.
  Finally, start up the virtual machine to begin installation.

Step 6: Install CentOS 6.5

  You can install the operating system manually or using a Kickstart Profile.
  I will be providing steps to install the operating system manually and using
  a Kickstart Profile.

  Be aware that no where in the following steps do I install the VirtualBox
  Guest Additions. So far, I have not found a need for them. Feel free to
  install the VirtualBox Guest Additions if you need them.

  Follow the steps in one of the following two sections, Manual Installation or
  Kickstart Profile Installation.

  Step 6a: Manual Installation

    Install the operating system however you like. Most of the default options
    can be used.

    Once the operating system has finished installing and booted, perform the
    following post-install steps to make it work with Vagrant.

    Open the virtual machine console and login as the root user (password is vagrant).

    By default, eth0 is not brought up, so bring it up:

      ifup eth0

    Install additional repository packages:

      yum install -y openssh-clients man git vim wget curl ntp

    Enable the ntpd service to start on boot:

      chkconfig ntpd on

    Set the time:

      service ntpd stop
      ntpdate time.nist.gov
      service ntpd start

    Enable the ssh service to start on boot:

    chkconfig sshd on

    Disable the iptables and ip6tables services from starting on boot:

      chkconfig iptables off
      chkconfig ip6tables off

    Set SELinux to permissive:

      sed -i -e 's/^SELINUX=.*/SELINUX=permissive/' /etc/selinux/config

    Add vagrant user:

      useradd vagrant

    Create vagrant user’s .ssh folder:

      mkdir -m 0700 -p /home/vagrant/.ssh

    If you want to use your own SSH public/private key then create an SSH
    public/private key on your workstation (you may already have), and copy
    the public key to /home/vagrant/.ssh/authorized_keys on the virtual
    machine.

    Otherwise, if you want to use the SSH public/private key provided by
    Vagrant, run the following command:

      curl https://raw.githubusercontent.com/mitchellh/vagrant/master/keys/vagrant.pub >> /home/vagrant/.ssh/authorized_keys

    Change permissions on authorized_keys files to be more restrictive:

      chmod 600 /home/vagrant/.ssh/authorized_keys

    Make sure vagrant user and group owns the .ssh folder and its contents:

      chown -R vagrant:vagrant /home/vagrant/.ssh

    Comment out requiretty in /etc/sudoers. This change is important because
    it allows ssh to send remote commands using sudo. Without this change
    vagrant will be unable to apply changes (such as configuring additional
    NICs) at startup:

      sed -i 's/^\(Defaults.*requiretty\)/#\1/' /etc/sudoers

    Allow user vagrant to use sudo without entering a password:

      echo "vagrant ALL=(ALL) NOPASSWD: ALL" >> /etc/sudoers

    Open /etc/sysconfig/network-scripts/ifcfg-eth0 and make it look exactly like the following:

      DEVICE=eth0
      TYPE=Ethernet
      ONBOOT=yes
      NM_CONTROLLED=no
      BOOTPROTO=dhcp

    Remove the udev persistent net rules file:

      rm -f /etc/udev/rules.d/70-persistent-net.rules

    Clean up yum:

      yum clean all

    Clean up the tmp directory:

      rm -rf /tmp/*

    Clean up the last logged in users logs:

      rm -f /var/log/wtmp /var/log/btmp

    Clean up history:

      history -c

    Shutdown the virtual machine:

      shutdown -h now

    Once the virtual machine is shutdown, open Settings for the virtual machine.

    Go to the Storage tab, select Controller: IDE, and click the green square
    with red minus icon in the lower right hand corner of the Storage Tree
    section of the Storage tab.

    Click OK to close the Settings menu.

    Next, jump to the Create the Vagrant Box section.

  Step 6a: Kickstart Profile Installation

    I used this Kickstart Profile to automate the build.

    When the CentOS boot menu appears, highlight Install or upgrade an existing
    system, hit the Tab key to bring up the anaconda boot line, and append the
    following:

      noverifyssl ks=https://raw.githubusercontent.com/jameswthorne/kickstart-profiles/master/centos-6-x86_64-vagrant-box.txt

    Hit the Enter key and wait for the installation to finish.

    At the end of the install, shutdown the virtual machine, and open Settings again for the virtual machine.

    Go to the Storage tab, select Controller: IDE, and click the green square
    with red minus icon in the lower right hand corner of the Storage Tree
    section of the Storage tab.

    Click OK to close the Settings menu.

    Next, jump to the Create the Vagrant Box section.

Step 8 (optional): Install VirtualBox Guest Additions

  Execute the following command on the VM:

    sudo yum -y update
    yum install -y gcc kernel-devel
    cd /opt
    sudo wget -c http://download.virtualbox.org/virtualbox/4.3.0/VBoxGuestAdditions_4.3.0.iso -O VBoxGuestAdditions_4.2.8.iso
    sudo mount VBoxGuestAdditions_4.3.0.iso -o loop /mnt
    cd /mnt
    sudo sh VBoxLinuxAdditions.run --nox11
    cd /opt
    vagrantup:~$ sudo rm *.iso
    sudo /etc/init.d/vboxadd setup
    sudo chkconfig --add vboxadd
    sudo chkconfig vboxadd on
    exit

Step 9: Create the Vagrant Box

  Execute the following commands to create the box and add it to the list of
  available boxes:

    vagrant package --output centos-6.6-x86_64.box --base centos-6.6-x86_64
    vagrant box add centos-6.6-x86_64 centos-6.6-x86_64.box

  Make sure the value of the base command line switch matches the name of the
  virtual machine used in VirtualBox:

  In addition, the VirtualBox virtual machine can be deleted.

Step 10: Create a Vagrant Project and Configure Vagrantfile

  You can have as many vagrant projects as you want. Each will contain
  different Vagrantfiles and different virtual machines. So, create a
  directory somewhere to house your Vagrantfile and associative virtual
  machines:

    mkdir -p ~/Development/vagrant-test
    cd ~/Development/vagrant-test

  Create the Vagrantfile:

    vagrant init centos-6.6-x86_64

  You now have a Vagrantfile that points to the centos-6.6-x86_64 Base Box you
  just created.

  If you are using your own SSH private/public key, and not the SSH
  private/public key provided by Vagrant, you need to tell Vagrantfile
  where to find your SSH private key, so add the following to your Vagrantfile:

    config.ssh.private_key_path = "~/.ssh/id_rsa"

  Lastly, if you do not want Shared Folders setup between your workstation and
  virtual machine, disable it by adding the following to your Vagrantfile:

    config.vm.synced_folder ".", "/vagrant", id: "vagrant-root", disabled: true

  Spin up your first virtual machine:

    vagrant up

  If everything spins up properly you can see the status of your virtual machine
  using vagrant status, you can SSH into your virtual machine using vagrant ssh,
  or you can destroy your virtual machine using vagrant destroy.

