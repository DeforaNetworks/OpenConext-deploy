Ansible-driven provisioning the OpenConext platform.
==============================

# Getting started

These step for setting up are based on Mac OSX. Please feel free to create a PR for other environments.

This playbook uses a custom vault, defined in filter_plugins/custom_plugins.py in order to encrypt data. We think this is a better solution than the ansible-vault because it allows us to do fine grained encryption instead of a big ansible-vault file.
Also, the creator of ansible-vault admits his solution is not the way to go. See [this blogpost](http://jpmens.net/2014/02/22/my-thoughts-on-ansible-s-vault/).

To encrypt and decrypt values use the scripts in `./scripts/encrypt.sh` and `./scripts/encrypt-file.sh`. Run them without arguments to see the help.

## Install Vagrant and VirtualBox

    brew tap caskroom/cask
    brew install brew-cask
    brew cask install vagrant
    brew cask install virtualbox

## Install Ansible

Ansible is the configuration tool we use to describe our servers. To
install for development:

    brew install python
    pip install --upgrade setuptools
    pip install --upgrade pip
    brew linkapps
    brew install ansible
    pip install python-keyczar==0.71c


# Getting started - OpenConext VM

The VM setup is intended for demo purposes only since it is using the `openconext-unsafe-keystore`.

Create the symlink so that our playbook can find the AES key in this project:

`ln -s $CURRENT_DIR/openconext-unsafe-keystore ~/.openconext-keystore`

The setup of the VM is using the `openconext-unsafe-keystore`, don't use that in production.


## Run playbooks

For the VM environment the certificates are generated on the fly. Typically you would only run `openconext-java.yml` for installing the java apps on a target environment.
The VM will install everything on a single box for demo purposes.

To provision the VM please run:

1. `vagrant up openconext-vm.yml` - This will setup a VM and will make sure the HOSTS file is able to handle the defined base_domain
2. `./ansible-vm openconext-storage.yml` - This will setup a MySQL server and LDAP for storage. In real environments it is advisable to install these on a separate box.
3. `./ansible-vm openconext-java.yml` - This will install all Java apps for the openconext platform.
4. `./ansible-vm openconext-php.yml` - This will install all PHP apps for the openconext platform.
5. `./ansible-vm openconext-mujina.yml` - This will install [mujina](https://github.com/OpenConext/Mujina) as IDP and SP for the VM environment.


# Getting started - Production versions

## Create your own keystore(s)

Create a keystore on an encrypted disk partition, to keep it extra safe (e.g. in case of laptop-loss).
Here's how to [create an encrypted folder](http://apple.stackexchange.com/questions/129720/how-can-i-encrypt-a-folder-in-os-x-mavericks) on your Mac.

This is how the keystore is created (you don't have to do this if it already exists for your project).
See [this blogpost](http://www.saltycrane.com/blog/2011/10/notes-using-keyczar-and-python/) for example.

`keyczart create --location=PATH_TO_FOLDER --purpose=crypt`

`keyczart addkey --location=PATH_TO_FOLDER --status=primary`

Create the symlink so that our playbook can find the AES key in this project:

`ln -s PATH_TO_FOLDER ~/.openconext-keystore`

Once created the keystore needs to be shared with your colleagues who also need to deploy. Whether you choose to check in into a git repo is all up to you.
As long as a ~/.openconext-keystore exists the provision scripts will work.

**Notice: When the crypted store is compromised you should change all encrypted values ASAP. The same goes for when you loose your home key, you should change the locks.**

## Create group_vars and inventory for you environment.

Currently in `./inventory` only the `vm` inventory file exists. This describes the vm environment. When for instance creating a production setup create an inventory file called `production`.
To deploy the openconext-java apps to this new production environment you should:

1. copy the openconext-java setup and adjust accordingly

        [openconext-java]
        PRODUCTION_ADDRESS_HERE

        [openconext-java-production:children]
        openconext-java

2. Create a `groups_vars/openconext-java-production.yml` and adjust values accordingly. There is a special property (`env`) that is also used to determine where to look for certificates and such.
    In this case use `production` as `env`.

3. Place your certificates in `java-production/certs` and `php-production/certs`. If you are going to checkin the private keys as well please make sure you encrypt them using `scripts/encrypt-file.sh`.

4. Run ansible-playbook -u USERNAME -i inventory/production openconext-java.yml (USERNAME needs sudo rights, use -K when a password is needed to sudo)