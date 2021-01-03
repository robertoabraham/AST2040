# AST2040 - Technical setup 

As an experiment I'm trying to manage the setup for this course using my iPad. The heavy lifting is all done using remote access to a virtual machine running on the Digital Ocean cloud.

## Setting up the VM

I purchased a monthly plan for an Ubuntu 18.04 VM using the Working Copy iPad Git client. I kept notes on how this was done in Bear. It was really easy. To provision the machine, first log in as root and set up a user account. Then log in with the user account and install the needed software. Both steps are stored as Termius "snippets" but here they are again.

1. Create user account

The following steps create a user account named "abraham". This account has `sudo` access and a password (though logging in from a remote machine will still require an SSH key).

```
mkdir -p /home/abraham/.ssh
touch /home/abraham/.ssh/authorized_keys
useradd -d /home/abraham abraham
usermod -aG sudo abraham
usermod -s /bin/bash abraham
chown -R abraham:abraham /home/abraham/
chmod 700 /home/abraham/.ssh
chmod 644 /home/abraham/.ssh/authorized_keys
cat .ssh/authorized_keys >> /home/abraham/.ssh/authorized_keys
passwd abraham
```

2. Install software

```
#!/bin/bash
#
# THIS SHOULD BE RUN IN YOUR USER ACCOUNT AND NOT AS ROOT!
# 
# If this is a fresh Digital Ocean VM you should first create 
# your user account using a separate snippet.
# 
# Follow the steps in this URL, which shows you how to add
# a user account and how to set things up to access it using 
# a private SSH key:
# https://shandou.medium.com/testing-out-digitalocean-droplet-1-steps-for-ssh-into-droplet-as-non-root-user-with-sudo-access-c2a7a5229cd6


#echo "*************************************************"
#echo Update to latest packages
#echo "*************************************************"
sudo apt update

#echo "*************************************************"
#echo Installing mosh
#echo "*************************************************"
sudo apt-get -y install software-properties-common
sudo add-apt-repository ppa:keithw/mosh
sudo apt-get update
sudo apt-get -y install mosh

echo "*************************************************"
echo "Installing gcc, node, and npm"
echo "*************************************************"
sudo apt-get -y install gcc nodejs npm

echo "*************************************************"
echo "Installing Miniconda Python 3.8"
echo "*************************************************"
mkdir -p tmp
cd tmp
curl -O https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh
bash ./Miniconda3-latest-Linux-x86_64.sh -b
echo 'export PATH="/home/abraham/miniconda3/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc

echo "*************************************************"
echo "Installing AstroConda (might prompt for a 'yes')"
echo "*************************************************"
conda config --add channels http://ssb.stsci.edu/astroconda
conda create -n astroconda stsci
source activate astroconda

echo "*************************************************"
echo "Final configuration"
echo "*************************************************"
git config --global user.name "Roberto Abraham"
git config --global user.email abraham@astro.utoronto.ca

```

## Using the VM

### Step 0. Log in and start up a TMUX session

To log in to the machines, use SSH private key authentication. The key is embedded in Working Copy, in Termius, and can be retreived from the Digital Ocean account webpage. I believe it is best to use `mosh` rather than `ssh` though both will work.

Run stuff from your user account, not from the root account.

Access to VMs from mobile devices is inherently a bit flakey, which is why we're using `mosh` instead of `ssh`. Another element in helping to preserve console state and keep stuff on the VM running is to use `tmux`. Here is a handy tmux cheat sheet: https://tmuxcheatsheet.com.

To create a new session (which you shoudl have to do rarely)

```
tmux new -s astro_session
```

To detatch from this session:

```
^b
d
```

When you log back in, reattach to the session:

```
tmux a
```

### Step 1. Activate Anaconda

I'm relying on Anaconda to provide most of the stuff I need, so the first thing to do is to activate the astroconda virtual environment:

```
source activate astroconda
```

## Step 2. Run the Jupyter notebook server. 

### Running the Jupyter server

You can start up the server from anywhere. If you're worried, don't do it from your root directory. Probably `/home/abraham/git` is as good a choice as any.

```
jupyter notebook --no-browser
```

This will provide a set of links which you can click on. However, the URL has a long token, and if you close the tab with the notebook interface you'll have to log back into the VM and do some digging around to figure out the tokens. It's better to avoid making the user worry about dealing with tokens by first seting a password this way:

```
jupyter notebook password
```

When a user first goes to the web page with the notebook they will be prompted for the password. If you forget the password you can reset it using the same command.

### Viewing Jupyter notebooks

The notebook runs on port 8080 in the VM. _To get access to the notebook server, use an SSH tunnel to VM from your local machine_. I currently use Termius on my iPad for this, and it is easy to set up. Once the VM is set up all you need to do is fire up Safari on the iPad and go to: `http://localhost:8080`


## Appendix. For bonus Points: run a VSCode server.

This step is optional, as it's perfectly easy to just log in to the VM using termux and 
edit files using vi. However, an alternative way to access files on the VM is to run the Coder VSCode server. This works fairly well, though it has some issues with using the trackpad to select stuff, and the cursor can sometimes get lost after trackpad moves (though you can get it back by tapping with your finger). In any case, it's pretty cool and is completely trivial to set up! Instructions are here: https://github.com/cdr/code-server/releases/tag/v3.8.0, and installation is as simple as doing this:

```
curl -fsSL https://code-server.dev/install.sh | sh
```

To start up the server:

```
code-server &
```

Access the server using an SSH tunnel, this time to port 8080.