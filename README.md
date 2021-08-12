# jetson-nano-mpi-cluster
Use mpich or openmpi above jetson nano to build the cluster.   

# Setup SSH
### All nodes must have the same username.   

## for all nodes  
```
sudo apt-get install openssh-server openssh-client   
mkdir ~/.ssh   
chmod 700 ~/.ssh   
```bash
## for master node
```
ssh-keygen -t rsa   
ssh-copy-id <WORKER-NODE-IP-ADDRESS>   
```
now we can access each node without a password via ssh.   

# Setup NFS
## for master node
```bash
sudo apt-get install nfs-kernel-server   
sudo mkdir -p /home/$USER/Desktop/sharedfolder   
sudo chown nobody:nogroup /home/$USER/Desktop/sharedfolder   
sudo chmod 777 /home/$USER/Desktop/sharedfolder   
sudo vi /etc/exports   
```
add   
```bash
/home/$USER/Desktop/sharedfolder <worker1IP>(rw,sync,no_subtree_check)   
/home/$USER/Desktop/sharedfolder <worker2IP>(rw,sync,no_subtree_check)   
```
```bash
sudo exportfs -a   
sudo systemctl restart nfs-kernel-server   
```
ufw status should be inactive
```bash
sudo apt-get install ufw   
sudo ufw status   
```
## for worker nodes
```bash
sudo apt-get install nfs-common   
sudo mkdir -p /home/$USER/Desktop/sharedfolder   
sudo mount <masterIP>:/home/$USER/Desktop/sharedfolder /home/$USER/Desktop/sharedfolder   
```
insert the file to make sure it's properly built.   
note: nfs unmount when the power is shut down, so we need to mount it again after booting.   

# Setup OpenMPI
## for all nodes
```bash
sudo apt-get install openmpi-bin openmpi-common libopenmpi-dev libgtk2.0-dev   
cd ~/Desktop   
wget https://download.open-mpi.org/release/open-mpi/v4.1/openmpi-4.1.1.tar.gz   
tar -xvf openmpi-4.1.1.tar.gz   
cd openmpi-4.1.1   
./configure --prefix="/home/$USER/.openmpi"   
make   
sudo make install   
```
add to ~/.zshrc   
```bash
vi ~/.zshrc   
export PATH="$PATH:/home/$USER/.openmpi/bin"   
export LD_LIBRARY_PATH="$LD_LIBRARY_PATH:/home/$USER/.openmpi/lib"   
```

# Execute hello-mpi.cpp
```bash
cd ~/Desktop/sharedfolder   
```
make cluster list that contain nodes' ip-addr in cluster_list
```bash
vi cluster_list
```
```bash
<masterIP>
<worker1IP>
<worker2IP>
```
add bear if you use   
```bash
bear mpic++ hello-mpi.cpp -o hello-mpi   
mpirun -np <number of process> --hostfile cluster_list ./hello-mpi   
```
