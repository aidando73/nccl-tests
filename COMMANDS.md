```bash
# Both nodes
sudo apt install -y libopenmpi-dev
sudo apt install -y openmpi-bin openmpi-common
sudo apt install -y libnccl2 libnccl-dev
sudo apt install -y sudo
sudo adduser mpiuser
sudo usermod -aG sudo mpiuser
su - mpiuser

ssh-keygen -t rsa
cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys


cd /workspace/nccl-tests
sudo chmod -R 777 .
cp sample.envrc .envrc
ip addr # Then fill in the IPs in the .envrc file on both nodes
direnv allow

# Do for both nodes
cat ~/.ssh/id_rsa.pub # Copy this

vim ~/.ssh/authorized_keys # On other node & paste in key

# Test connections
# From master to worker
ssh $WORKER_IP
# From worker to master
ssh $MASTER_IP

# Both nodes
make MPI=1 MPI_HOME=/usr/lib/x86_64-linux-gnu/openmpi

# Master
export NCCL_DEBUG=INFO
export NCCL_SOCKET_IFNAME=podnet1
mpirun --verbose -host $MASTER_IP,$WORKER_IP /workspace/nccl-tests/build/all_reduce_perf -b 8 -e 256 -f 2 -g 8 --timeout 10

./build/all_reduce_perf -b 8 -e 128M -f 2 -g 2

mpirun -np 4 -N 2 -hostfile hosts.txt ./build/all_reduce_perf -b 8 -e 8G -f 2 -g 1

mpirun --verbose -host $MASTER_IP,$WORKER_IP /workspace/nccl-tests/build/all_reduce_perf -b 8 -e 8G -f 2 -g 1

export NCCL_DEBUG=INFO
export NCCL_SOCKET_IFNAME=podnet1
mpirun --verbose -host $MASTER_IP,$WORKER_IP /workspace/nccl-tests/build/all_reduce_perf -b 8 -e 128M -f 2 -g 2 --timeout 10


# MPI cluster setup
# Password 123



export MASTER_IP="497nwj7n5u5r18.runpod.internal"
export WORKER_IP="2akfl65n0ptnqb.runpod.internal"
export NCCL_DEBUG=INFO
export NCCL_SOCKET_IFNAME=podnet1
mpirun --verbose -host "$MASTER_IP,$WORKER_IP" /workspace/nccl-tests/build/all_reduce_perf -b 8 -e 256 -f 2 -g 2 --timeout 10


# Worker node setup
apt install sudo
apt install libopenmpi-dev
apt install libnccl2 libnccl-dev
adduser mpiuser
usermod -aG sudo mpiuser
# Password 123


# Doesn't work
sudo sed -i 's/^#*PasswordAuthentication .*/PasswordAuthentication yes/' /etc/ssh/sshd_config && \
sudo sed -i 's/^#*PubkeyAuthentication .*/PubkeyAuthentication yes/' /etc/ssh/sshd_config
sudo systemctl restart sshd
```