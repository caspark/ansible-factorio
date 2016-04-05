# Ansible Factorio playbook

An ansible playbook which installs a Factorio headless server on an Ubuntu x64 14.04 server and makes it manageable
using upstart (e.g. `service factorio start`) via [factorio-init](https://github.com/Bisa/factorio-init). You still need
to upload an initial Factorio save game to the server using e.g. `scp`.

The instructions are written for an AWS Ubuntu 14.04 server but they should work with other providers as well (see notes
on non-AWS servers below).

# Usage

## Local Setup (skip if you already have ansible >= 2.0 installed)

Make sure you have Python 2 (with the pip and virtualenv packages) and git installed locally already:

    sudo apt install -y git python python-dev python-pip
    sudo pip install virtualenv

Then install Ansible:

    virtualenv .ansible-venv      # create a virtual env to avoid interfering with python packages installed system-wide
    . .ansible-venv/bin/activate  # activate the python virtual env for this shell session
    pip install ansible           # install ansible into python virtual env

## Provision your server!

Assuming you've provisioned a 14.04 server via AWS (see below if using a different provider):

    ansible-playbook playbooks/factorio.yml -i 'hostnameofyourserver,' --user ubuntu --become

If it all succeeds, you should be able to SCP up your save game and start your server:

    scp my-factorio-savegame.zip ubuntu@hostnameofyourserver:/opt/factorio/saves/
    ssh ubuntu@hostnameofyourserver
    sudo service factorio help
    sudo service factorio start   # loads the most recently modified save into your server
    sudo service factorio screen  # view server logs using Screen (press "Ctrl+a, d" to leave Screen)
    sudo service factorio stop    # should save the game and quit (but make a client-side save just in case)

See https://github.com/Bisa/factorio-init for more info on the init script.

# Troubleshooting

Check ansible can talk to your server (by running ifconfig):

    ansible -a 'ifconfig' -i 'hostnameofyourserver,' --user ubuntu -vvvvv all

If that still doesn't work, try with ansible debug output switched on:

    ANSIBLE_DEBUG=true ansible -a 'ifconfig' -i 'hostnameofyourserver,' --user ubuntu -vvvvv all

Make sure ansible can use sudo:

    ansible -a 'ifconfig' -i 'hostnameofyourserver,' --user ubuntu -vvvvv all --become

If starting Factorio fails, then try running it directly

    sudu -u factorio bash
    /opt/factorio/bin/x64/factorio --start-server /opt/factorio/saves/my-factorio-savegame.zip

## Using with non-AWS servers

The usage instructions above assume you can SSH to your server using the `ubuntu` user without a password (i.e. your
SSH public key is already authorised for the `ubuntu` user), and that the Ubuntu user has sudo permission without
needing to enter a password for sudo. This configuration matches an AWS 14.04 instance at time of writing.

I don't test with other configurations, but that aside, you should be able to use the following tweaks:

* If you want to log in as root, you should specify `--user root`
* If you don't want your user account to use sudo (e.g. using root login), then leave off `--become`
* If sudo requires a password, include `--ask-become-pass` as an argument
* If you need to use a password to authenticate over SSH, include `--ask-pass` as an argument

For more special case scenarios, see the Ansible documentation.
