# Installing & running guide

## Install virtualbox
1. Download: https://www.virtualbox.org/
2. verify: `vagrant --version`


## Install vagrant
1. Download: Link: https://developer.hashicorp.com/vagrant/install
2. Verify: `VBoxManage --version`

## Project Initialization
Create a dedicated folder for your project. Vagrant relies on the folder structure to manage the machine.

`mkdir my-vagrant-server`
`cd my-vagrant-server`

### Initialize with Ubuntu 20.04 (Focal Fossa)
<!-- `vagrant init ubuntu/focal64` -->
`vagrant init bento/ubuntu-24.04`

Note: `vagrant init` creates a file named `Vagrantfile`. This file is the "blueprint" of your server.

## Start and Access the Server
```bash
# Power on the VM (This will download the image first)
vagrant up

# Enter the server via SSH
vagrant ssh
```

Once inside, you are in a Linux terminal. You can run `lsb_release -a` to prove you are on Ubuntu. To leave the server, type `exit`.

## Setup with Docker (The Fast Way)
Using Docker as a provider is much lighter than VirtualBox because it shares the host's kernel. This is perfect for people with older laptops or limited RAM.

### Step 1: The Vagrantfile Configuration
Instead of using the default configuration, you must tell Vagrant to use Docker.

- Create a new folder: mkdir vagrant-docker && cd vagrant-docker
- Create a file named Vagrantfile: nano Vagrantfile
- Paste this configuration:
```bash
Vagrant.configure("2") do |config|
  config.vm.define "web" do |v|
    # Use a Docker image instead of a VirtualBox box
    v.vm.provider "docker" do |d|
      d.image = "ubuntu:focal"
      d.has_ssh = true
    end
  end

  # Allow SSH access into the container
  config.ssh.username = "root"
end
```

### Step 2: Running with Docker Provider
You must explicitly tell Vagrant which "engine" to use if you have both installed.
```bash
# Start the container
vagrant up --provider=docker

# Check status
vagrant status
```


Check if VM is running: `vagrant status`
Save state & stop (Sleep): `vagrant suspend`
Full shutdown: `vagrant halt`
Apply changes in Vagrantfile: `vagrant reload`
Delete everything: `vagrant destroy`


## Writing the Vagrantfile (The DNA)
The `Vagrantfile` tells Vagrant three main things: What OS to use, how to Connect to it, and what Folders to share.

### The Core Structure
Every Vagrantfile starts and ends with a configuration block. Inside this block, we define our settings.

```bash
Vagrant.configure("2") do |config|
  # All settings go inside here
end
```

## Key Features Explained
1. Boxes (The Operating System)
Instead of installing an OS from an ISO file for 20 minutes, Vagrant uses "Boxes" (clones of pre-installed systems).

Code: `config.vm.box = "ubuntu/focal64"`

Where to find them: Vagrant Cloud is the "App Store" for boxes.
Link: https://portal.cloud.hashicorp.com/vagrant/discover


2. Networking (Connecting to the App)
There are two ways to access the web server running inside your VM:

**Forwarded Ports:** This maps a port on your physical laptop to a port inside the VM.

- Code: `config.vm.network "forwarded_port", guest: 80, host: 8080`
- Result: You visit `http://localhost:8080` on your laptop to see the server.

**Private Network (Static IP):** This gives the VM its own IP address on a virtual private network.

- Code: `config.vm.network "private_network", ip: "192.168.33.10"`
- Result: You visit `http://192.168.33.10` directly.

3. Synced Folders (Live Coding)
This is the most important feature for developers. It syncs a folder on your laptop with a folder in the VM.

- Code: `config.vm.synced_folder "./code", "/var/www/html"`
- The Magic: You edit a PHP file in your Windows/Mac code editor (VS Code), and the Nginx server inside the Linux VM sees the change instantly. No need to "upload" files.

## Hands-on: The "Ultimate" Vagrantfile Template
Create a new Vagrantfile and paste this block. It combines all the features mentioned above.

```bash
Vagrant.configure("2") do |config|
  # 1. Define the OS
  config.vm.box = "ubuntu/focal64"

  # 2. Set the Network (Access via http://localhost:8080)
  config.vm.network "forwarded_port", guest: 80, host: 8080
  
  # 3. Set a Static IP (Access via http://192.168.33.10)
  config.vm.network "private_network", ip: "192.168.33.10"

  # 4. Sync the folders
  # Create a folder named 'html' in your project directory first!
  config.vm.synced_folder "./html", "/var/www/html"

  # 5. Provider-Specific settings (Optional: Giving the VM more RAM)
  config.vm.provider "virtualbox" do |vb|
     vb.memory = "1024"
     vb.cpus = 2
  end
end
```

## Provisioning Servers with Vagrant
Provisioning is the process of automatically installing software and configuring the environment after the OS has booted.

### Why use Provisioning?
- Zero Manual Work: No more typing `sudo apt install` every time you create a VM.
- Consistency: Every student gets the exact same versions of Nginx, PHP, and MySQL.
- Documentation: The `Vagrantfile` becomes the documentation of what is installed on the server.

### Types of Provisioners
Vagrant supports several tools, but for beginners, we focus on Shell Provisioning (Bash scripts).
- Shell: Simple `.sh` scripts (What we will use).
- Ansible/Chef/Puppet: Professional configuration management tools (Used in large companies).
- File: Simply moving a file from your laptop into the VM.

### The Hands-on Guide: "The Auto-LAMP Stack"
This script will run only the first time we run `vagrant up`.

1. Create a new bash file
```bash
    #!/bin/bash
    echo "Updating system packages..."
    sudo apt-get update -y
    
    echo "Installing Nginx..."
    sudo apt-get install -y nginx
    
    echo "Installing MySQL..."
    sudo apt-get install -y mysql-server
    
    echo "Setup Complete! Visit http://localhost:8080"
```

- Create a file named setup.sh in the same folder.
- Add your commands there.
- In your Vagrantfile, use this one line: `config.vm.provision "shell", path: "setup.sh"`

## Important Concepts
1. **The "First Time" Rule:** Provisioning only runs on the very first `vagrant up`. If you change the script later, you must run `vagrant provision` or `vagrant reload --provision` to apply the changes.
2. **The `-y` Flag:** We must use `-y` (e.g., `apt install nginx -y`). If the script waits for a user to type "Yes," the provisioning will hang and fail because there is no interactive screen.
3. **Root Privileges:** Vagrant runs provisioning as the `root` user by default, so you don't always need `sudo`, but it’s good practice to keep it for clarity.
