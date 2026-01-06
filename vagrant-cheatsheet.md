# Vagrant Essential Cheat Sheet

## The Lifecycle (Managing the Server)
- `vagrant init <box_name>` – Initializes a new Vagrant project by creating a Vagrantfile.

- `vagrant up` – Starts the virtual machine (and provisions it if it's the first run).

- `vagrant ssh` – Connects you to the server's Linux terminal via SSH.

- `vagrant halt` – Gracefully shuts down the virtual machine.

- `vagrant suspend` – Pauses the VM (saves the RAM state to disk).

- `vagrant resume` – Wakes up a suspended VM.

- `vagrant reload` – Restarts the VM to apply changes made in the Vagrantfile.

- `vagrant destroy` – Deletes the VM and all its associated data (wipes the drive).

## Provisioning (Running Scripts)
- `vagrant provision` – Re-runs your setup.sh or inline scripts on a running VM.

- `vagrant reload --provision` – Restarts the VM and forces the provisioning scripts to run.

## Information & Status
- `vagrant status` – Shows if the current VM is running, poweroff, or saved.

- `vagrant global-status` – Lists all Vagrant environments currently on your laptop.

- `vagrant port` – Displays the forwarded port mappings between host and guest.

- `vagrant version` – Checks the currently installed version of Vagrant.

## Box Management
- `vagrant box add <name>` – Downloads a specific box to your local machine for future use.

- `vagrant box list` – Lists all the OS boxes you have downloaded on your laptop.

- `vagrant box update` – Downloads the newest version of the current box image.

- `vagrant box remove <name>` – Deletes an OS box image from your laptop to save space.

## Pro-Tip:
If you are ever lost or forget a command, just type `vagrant` or `vagrant help` in your terminal to see the full list of available options.