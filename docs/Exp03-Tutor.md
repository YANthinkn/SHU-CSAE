# Exp03 Tutor

## 1. Install VMWare Workstation Pro

## 2. Download Ubuntu 23 Desktop iso File

Official website: [link](https://ubuntu.com/download/desktop).

Official CD mirrors: [link](https://launchpad.net/ubuntu/+cdmirrors).

Download the file named something like "**ubuntu-23.10.1-desktop-amd64.iso**".

## 3. Install Three Ubuntu 23 Virtual Machines in VMWare

💡**Note**:
- Set the computer name (host name) to be "node1", "node2" and "node3" repectively.
- Create users with the same username on each virtual machine.

## 4. Environment Setup

### 4.1. Software Installation

Run following commands on every virtual machine:

```bash
sudo apt update

sudo apt install -y open-vm-tools open-vm-tools-desktop
```

Restart every virtual machine, and now you are able to copy and paste any content or files from your host machine to the virtual machines!

Install some other softwares on every virtual machine:

```bash
sudo apt install -y build-essential cmake git vim-gtk3 openssh-server

# Turn down the firewall for ssh:
sudo ufw allow ssh
```

### 4.2. SSH Key Generation

Generate ssh keys on every virtual machine:

```bash
# Replace <username> with your current username
# Replace <1/2/3> with the corresponding number of the virtual machine
ssh-keygen -t rsa -b 4096 -C "<username>@node<1/2/3>"
```

You should be able to find the generated keys in `~/.ssh/` directory.

### 4.3. SSH Key Sharing

Check the ip address of every virtual machine:

```bash
ip a
```

Suppose you get following results:

| Host Name | IP Address |
| :---: | :---: |
| node1 | 192.168.x.10 |
| node2 | 192.168.x.11 |
| node3 | 192.168.x.12 |

💡**Note**: 
- These IP addresses are just examples, you should replace them with your own IP addresses.
- Try to ping every virtual machine from each other to make sure they can communicate with each other.

Share the public key of every virtual machine to every other virtual machine:

```bash
# Replace <username> with your current username
# Replace <IP-Address> with the corresponding IP address of the virtual machine
ssh-copy-id <username>@<IP-Address>
```

The public keys are now written to the `~/.ssh/authorized_keys` file on every virtual machine.

### 4.4. Hosts File Configuration

Edit the `/etc/hosts` file on every virtual machine:

```bash
sudo vim /etc/hosts
```

Add following lines to the file:

```bash
# [Note]:
#   If there has been something like: 
#   > 127.0.1.1 node<1/2/3>
#   Remove this line.

# Replace `192.168.x.10`, `192.168.x.11` and `192.168.x.12` with your own IP addresses.
192.168.x.10 node1
192.168.x.11 node2
192.168.x.12 node3
```

### 4.5. Reboot Every Virtual Machine

Reboot every virtual machine. 

### 4.6. Try to SSH to Every Virtual Machine

From every virtual machine, SSH to each other.

```bash
ssh node1
exit

ssh node2
exit

ssh node3
exit
```

## 5. Install zlib

Download source code of zlib from the official website [[link](https://zlib.net/)].

💡**Note**: 
- You can download it on your host machine and then transfer it to every virtual machine with `scp` command.  
  For example:

  ```bash
  # Transfer "./target-file" on your local machine to "~/Desktop/target-file"
  scp ./target-file <username>@<virutal-machine-ip-address>:~/Desktop
  ```

Suppose your downloaded file is named **zlib-1.3.1.tar.gz**.

Extract the file and install it:

```bash
tar -xf zlib-1.3.1.tar.gz

cd zlib-1.3.1

sudo ./configure --prefix=/usr/local/zlib-1.3.1

sudo make -j $(nproc) all

sudo make install
```

Edit **/etc/bash.bashrc** file on every virtual machine with command `sudo vim /etc/bash.bashrc`.  Add following lines to the file:

```bash
export ZLIB_HOME=/usr/local/zlib-1.3.1

export LD_LIBRARY_PATH=$ZLIB_HOME/lib:$LD_LIBRARY_PATH
```

Then run `source /etc/bash.bashrc` to make the changes take effect.

## 6. Install openmpi

Download source code of openmpi from the official website [[link](https://www.open-mpi.org/software/)].

Supposed the file you downloaded is named **openmpi-5.0.3.tar.bz2**.  

Open your terminal and change directory to the path of the compressed file.

```bash
# Extact the file, and you will get a folder named "openmpi-5.0.3"
tar -xf openmpi-5.0.3.tar.bz2

cd ./openmpi-5.0.3

sudo ./configure --prefix=/usr/local/openmpi-5.0.3 --with-zlib=$ZLIB_HOME

sudo make -j $(nproc) all

sudo make install
```

Edit **/etc/bash.bashrc** file with command `sudo vim /etc/bash.bashrc`, add following lines to the end of that file:

```bash
# >>> openmpi >>>
export OPENMPI_HOME="/usr/local/openmpi-5.0.3"
alias OPENMPI_INIT="export PATH=$OPENMPI_HOME/bin:$PATH && export LD_LIBRARY_PATH=$OPENMPI_HOME/lib:$LD_LIBRARY_PATH"
# <<< openmpi <<<
```

Now, every time you open a new terminal, you can initialize the environment of OpenMPI with the following command: 

```bash
OPENMPI_INIT
```

You can also check the path of mpi executables after initialized the environment:

```bash
# Check where is mpicxx
which mpicxx
# Check where is mpirun
which mpirun
```

## 7. Build MPI Program

💡**Note**: 
- Following steps should be done on every virtual machine.

### 7.1. Clone the Project

Clone ***this*** project if you haven't done it yet:

```bash
git clone --recursive git@github.com:jamesnulliu/SHU-Computer-System-Architecture-Experiments.git SHU-CSAE

cd SHU-CSAE
```

### 7.2. Build Yutils

Yutils is a submoudle and shuold be built once for all.

```bash
bash scripts/build-libs.sh
```

You should be able to find "libYutils.a" in "./vendor/Yutils/lib/" directory.

💡**Note**: 
- You don't have to build yutils again if you have already built it.

### 7.3. Build Exp03

```bash
OPENMPI_INIT

CC=mpicc CXX=mpicxx bash scripts/build-exp03.sh
```

You should be able to find the executable file named "exp03" in "./exp03-mpi/bin" directory.

### 7.4. Create a Hostfile

Create a hostfile named "hostfile" in the "./" directory. Write following lines to the file:

```bash
node1 slots=2
node2 slots=2
node3 slots=2
```

Where `slots` is the number of processes you want to run on each node.

### 7.5. Run the Program

Run the program with following command; Replace `<nProcess>` with the overall number of processes you want to run:

```bash
mpirun --hostfile hostfile -np <nProcesses> ./exp03-mpi/bin/Release/Linux_x86_64/hellompi
```

You should be able to see some output like:

```
Hello, I am rank 5
Hello, I am rank 3
Hello, I am rank 4
Hello, I am rank 2
Hello, I am rank 1
Hello, I am rank 0
```

