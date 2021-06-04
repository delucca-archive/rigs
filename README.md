# Rigs

This repository aims to organize the setup automation for my personal rigs. Each rig is added to a given host group, being one of the following:

* **Workstation:** Allowed to code and execute projects
* **Farmer:** Responsible for farming Chia
* **Harvester:** Responsible for plotting Chia

* [Table of contents](#)
  * [Install](#install)
  * [Usage](#usage)
  * [Available playbooks](#available-playbooks)
  * [Adding a new rig](#adding-a-new-rig)
  * [License](#license)

## Install

Before launching any playbook, you need to install the following tools:

- [Ansible](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html)

After installing those dependencies, you need to setup your control node SSH keys and setup all your rigs by following the steps in ["adding a new rig"](#adding-a-new-rig). Finally, you can duplicate the example inventory with the command:

```sh
cp inventory.yaml.example inventory.yaml
```

And edit it by changing your node IPs to their proper value.

## Usage

To use my automation, you can choose a playbook in the ["available playbooks"](#available-playbooks) section and them run the following command:

```sh
ansible-playbook -i inventory.yaml <the-desired-playbook>
```

## Available playbooks

* [`setup`](./setup.yaml): Setup all your hosts based on their roles
* [`setup-workstations`](./setup-workstations.yaml): Setup only our workstation hosts
* [`setup-chia-farmers`](./setup-chia-farmers.yaml): Setup only our Chia farmer hosts
* [`setup-chia-harvesters`](./setup-chia-harvesters.yaml): Setup only our Chia harvesters hosts
* [`setup-chia-nodes`](./setup-chia-nodes.yaml): Setup all our Chia nodes

## Adding a new rig

The following steps are suggested for my `farmer` and `harvester` rigs. If you're bootstrapping a new `workerstation` rig, you should check [my Notion runbook](https://www.notion.so/odelucca/Workstation-Setup-Runbook-f19fdfa9b6e645c99fcf741cd38debaa)

### 1. Get the rig IP

If the rig is in your local network, you can get it by running the following command:

```sh
ip a
```

### 2. Copy your SSH credential to the rig

Knowing the rig user and password, you can run the following command to copy your SSH credential:

```sh
ssh-copy-id -i ~/.ssh/id_ed25519 <user>@<rig-IP>
```
> **IMPORTANT:** You must install the OpenSSH server on the remote rig before running this command

### 3. Login to the rig shell

```sh
ssh <user>@<rig-IP>
```

### 4. Remove the rig user password

First, change your sudo user to allow `NOPASSWD`. You need to run:

```sh
sudo visudo
```

Them, you can add the following line to that file:

```sh
ff-farmer ALL=(ALL) NOPASSWD:ALL
```
> **IMPORTANT:** Don't forget to change `ff-farmer` with your rig's user

Also, don't forget to enabled passwordless sudo by editing the following line:

```sh
# BEFORE
%sudo   ALL=(ALL:ALL) ALL
# AFTER
%sudo  ALL=(ALL:ALL) NOPASSWD:ALL
```

Now, you must delete your user password with the following command:

```sh
sudo passwd -d `whoami`
```

### 5. Enable autologin

First, launch the following command to edit the `getty` service:

```sh
sudo systemctl edit getty@tty1.service
```

Now, add the following contents to the file you're editting:

```ini
[Service]
ExecStart=
ExecStart=-/sbin/agetty --noissue --autologin <user> %I $TERM
Type=idle
```
> **IMPORTANT:** Don't forget to change `<user>` by your rig username

### 6. Add your rig to our inventory

After finishing the basic setup, go to your control node and add your rig to our hosts. You can do so by copying another rig structure in our `all.hosts` key and creating your own. We suggest using your machine unique identifier (use `cat /etc/machine-id` to find yours) as your `machine-id`.

## License

Distributed under the Apache 2.0 License. See [`LICENSE`](LICENSE) for more information.
