## Vagrant
https://www.vagrantup.com/docs/cli

### Commands:
Typing `vagrant` from the command line will display a list of all available commands.

Be sure that you are in the same directory as the Vagrantfile when running these commands!

# Creating a VM
- `vagrant init`           -- Initialize Vagrant with a Vagrantfile and ./.vagrant directory, using no specified base image. Before you can do vagrant up, you'll need to specify a base image in the Vagrantfile.
- `vagrant init <boxpath>` -- Initialize Vagrant with a specific box. To find a box, go to the [public Vagrant box catalog](https://app.vagrantup.com/boxes/search). When you find one you like, just replace it's name with boxpath. For example, `vagrant init ubuntu/trusty64`.

# Starting a VM
- `vagrant up`                  -- starts vagrant environment (also provisions only on the FIRST vagrant up)
- `vagrant resume`              -- resume a suspended machine (vagrant up works just fine for this as well)
- `vagrant provision`           -- forces reprovisioning of the vagrant machine
- `vagrant reload`              -- restarts vagrant machine, loads new Vagrantfile configuration
- `vagrant reload --provision`  -- restart the virtual machine and force provisioning

# Getting into a VM
- `vagrant ssh`           -- connects to machine via SSH
- `vagrant ssh <boxname>` -- If you give your box a name in your Vagrantfile, you can ssh into it with boxname. Works from any directory.

# Stopping a VM
- `vagrant halt`        -- stops the vagrant machine
- `vagrant suspend`     -- suspends a virtual machine (remembers state)

# Cleaning Up a VM
- `vagrant destroy`     -- stops and deletes all traces of the vagrant machine
- `vagrant destroy -f`   -- same as above, without confirmation

# Boxes
- `vagrant box list`              -- see a list of all installed boxes on your computer
- `vagrant box add <name> <url>`  -- download a box image to your computer
- `vagrant box outdated`          -- check for updates vagrant box update
- `vagrant boxes remove <name>`   -- deletes a box from the machine
- `vagrant package`               -- packages a running virtualbox env in a reusable box

# Saving Progress
-`vagrant snapshot save [options] [vm-name] <name>` -- vm-name is often `default`. Allows us to save so that we can rollback at a later time

# Tips
- `vagrant -v`                    -- get the vagrant version
- `vagrant status`                -- outputs status of the vagrant machine
- `vagrant global-status`         -- outputs status of all vagrant machines
- `vagrant global-status --prune` -- same as above, but prunes invalid entries
- `vagrant provision --debug`     -- use the debug flag to increase the verbosity of the output
- `vagrant push`                  -- yes, vagrant can be configured to [deploy code](http://docs.vagrantup.com/v2/push/index.html)!
- `vagrant up --provision | tee provision.log`  -- Runs `vagrant up`, forces provisioning and logs all output to a file

# Plugins
- [vagrant-hostsupdater](https://github.com/cogitatio/vagrant-hostsupdater) : `$ vagrant plugin install vagrant-hostsupdater` to update your `/etc/hosts` file automatically each time you start/stop your vagrant box.

# Notes
- If you are using [VVV](https://github.com/varying-vagrant-vagrants/vvv/), you can enable xdebug by running `vagrant ssh` and then `xdebug_on` from the virtual machine's CLI.
---
---
### Vagrant Project Setup
1. Start by creating a directory to store your Vagrant file:

```
mkdir vagrant-test
cd vagrant-test
```
2. Download the Ubuntu Trusty Tahr distribution from a common library and create a basic Vagrantfile with:

`vagrant init ubuntu/trusty64`

If you like, you can browse to https://app.vagrantup.com/boxes/search and download a Vagrantbox of your choosing.

When you run the init command, Vagrant installs the box to the current directory. The Vagrantfile is placed in the same directory and can be edited or copied.
Vagrant Boxes

The basic unit in a Vagrant setup is called a “box” or a “Vagrantbox.” This is a complete, self-contained image of an operating system environment.

A Vagrant Box is a clone of a base operating system image. Using a clone speeds up the launching and provisioning process.

1. Instead of using the init command above, you can simply download and add a box with the command:

`vagrant box add ubuntu/trusty64`

This downloads the box and stores it locally.

2. Next, you need to configure the Vagrantfile for the virtual box it will serve. Open the Vagrantfile with the command:

`vi vagrantfile`

3. Once the Vagrantfile is open, change the config.vm.box string from “base” to “ubuntu/trusty64”.

`config.vm.box = "ubuntu/trusty64"`

vagrantfile configuration base and config

You can add another line above the end command to specify a box version:

`config.vm.box_version = “1.0.1”`

Or you can specify a URL to link directly to the box:

`config.vm.box_url = “https://vagrantcloud.com/ubuntu/trusty64”`

If you’d like to remove a box, use the following:

`vagrant box remove ubuntu/trusty64`

VagrantFile

Instead of building out a complete operating system image and copying it, Vagrant uses a “Vagrantfile” to specify the configuration of the box.

Note: The download page for each box includes a configuration script that you can copy into your Vagrantfile.
Provisioning

If you spent enough in the guest OS, you may have noticed that it does not come loaded with many applications.

Fortunately, Vagrant supports automatic provisioning through a bootstrap.sh file saved in the same directory as the Vagrantfile.

To add a basic resource monitor, nmon, in the guest OS use the command:

`vi bootstrap.sh`

In that file, enter the following:
```
#!/usr/bin/env bash
apt-get update
apt-get install -y nmon
if ! [ -L /var/www ]; then
rm -rf /var/www
ln -fs /vagrant /var/www
fi
```
Save the file and exit. Next, edit the Vagrantfile and add the provisioning line. It should look as follows:

```
Vagrant.configure("2") do |config|
config.vm.box = "ubuntu/trusty64"
config.vm.provision :shell, path: "bootstrap.sh"
end
```
When Vagrant reads the Vagrantfile, it is redirected to read the bootstrap.sh file we just created. That bootstrap file will update the package manager, then install the nmon package. If you use vagrant up and vagrant ssh commands, you should now be able to run nmon for a display of the virtual machine’s resources.

Provisioning gives you a powerful tool for pre-configuring your virtual environment. You could also do the same with apache2, and create a web server in your virtual environment.
#### Providers
This tutorial shows you how to use Vagrant with VirtualBox. However, Vagrant can also work with many other backend providers.
To launches Vagrant using VMware run the command:

`vagrant up –provider=vmware_fusion`

Or you can launch Vagrant using Amazon Web Services with:

`vagrant up –provider=aws`

Once the initial command is run, subsequent commands will apply to the same provider.
----
### Launching and Connecting
Vagrant Up

The main command to launch your new virtual environment is:

`vagrant up`

This will run the software and start a virtual Ubuntu environment quickly. However, even though the virtual machine is running, you will not see any output. Vagrant does not give any kind of user interface.
----
### Vagrant SSH

You can connect to your virtual machine (and verify that it is running) by using an SSH connection:

`vagrant ssh`

This opens a secure shell connection to the new virtual machine. Your command prompt will change to vagrant@trusty64 to indicate that you are logged into the virtual machine.

Once you are done exploring the virtual machine, you can exit the session with CTRL-D. The virtual machine will still be running in the background, but the SSH connection will be closed.

To stop the virtual machine from running, enter:

`vagrant destroy`

The file you downloaded will remain, but anything that was running inside the virtual machine will be gone.
----
### Synced Folders

Vagrant automatically synchronizes content that is in your project directory with a special directory in the guest (virtual) system. The project directory is the one you created earlier, /vagrant-test. It’s also the same one that holds the Vagrantfile.

When you log into the virtual machine, by default it starts in the /home/vagrant/ directory. A different directory, /vagrant/, holds the same files that are on your host system.

You can use vagrant up and vagrant ssh to launch and log into the virtual machine, then create a test document in the /vagrant directory.

Use the exit command to close the SSH session, then use ls to list the contents of your vagrant-test directory. It should display the test file you created.

This is a handy way to manage files in the guest OS without having to use an SSH session.
----
### Networking

Vagrant includes options to place your virtual machine on a network. At the end of your Vagrantfile, just before the end command, use the config.vm.network command to specify network settings.

Example:

`config.vm.network “forwarded_port”, guest: 80, host: 8080`

For the changes to take place, save and reload Vagrant with the command:

`vagrant reload`

This creates a forwarded port for the guest system. You may also define private networks, public networks, and other more advanced options.
----
### Vagrant Share

Vagrant has a handy feature that you can use to share your Vagrant environment using a custom URL.

With your Vagrant environment running, use the following command:

`vagrant share`

The system will create a Vagrant Share session, then generate a URL. This URL can be copied and sent to another person. If you have Apache configured in your Vagrant session, anyone who uses this URL can see your Apache configuration page. This URL changes as you modify the contents of your shared folder.

You can close the sharing session with CTRL-C.
----
### Clean Up Vagrant

Once you are done working on your guest system, you have a few options how to end the session.

1. To stop the machine and save its current state run:

`vagrant suspend`

You can resume by running vagrant up again. This is much like putting the machine in sleep mode.

2. To shut down the virtual machine use the command:

`vagrant halt`

Again, vagrant up will reboot the same virtual machine, and you can resume where you left off. This is much like shutting down a regular machine.

3. To remove all traces of the virtual machine from your system type in the following:

`vagrant destroy`

Anything you have saved in the virtual machine will be removed. This frees up the system resources used by Vagrant.
