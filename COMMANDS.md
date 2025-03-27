```bash
# Both nodes
apt install libopenmpi-dev
apt install openmpi-bin openmpi-common
apt install libnccl2 libnccl-dev
apt install sudo
adduser mpiuser
usermod -aG sudo mpiuser

MASTER_IP=10.0.167.131
WORKER_IP=10.0.25.85

su - mpiuser
ssh-keygen -t rsa
cd ~/.ssh
cat id_rsa.pub >> authorized_keys

# Master node
ssh-copy-id $WORKER_IP

# Worker node
ssh-copy-id $MASTER_IP

make MPI=1 MPI_HOME=/usr/lib/x86_64-linux-gnu/openmpi

./build/all_reduce_perf -b 8 -e 128M -f 2 -g 2

mpirun -np 4 -N 2 -hostfile hosts.txt ./build/all_reduce_perf -b 8 -e 8G -f 2 -g 1

mpirun --verbose -host $MASTER_IP,$WORKER_IP /workspace/nccl-tests/build/all_reduce_perf -b 8 -e 8G -f 2 -g 1

export NCCL_DEBUG=INFO
export NCCL_SOCKET_IFNAME=podnet1
mpirun --verbose -host $MASTER_IP,$WORKER_IP /workspace/nccl-tests/build/all_reduce_perf -b 8 -e 128M -f 2 -g 2 --timeout 10

export MASTER_IP=10.0.167.131
export WORKER_IP=10.0.25.85
export NCCL_DEBUG=INFO
export NCCL_SOCKET_IFNAME=podnet1
mpirun --verbose -host $MASTER_IP,$WORKER_IP /workspace/nccl-tests/build/all_reduce_perf -b 8 -e 256 -f 2 -g 2 --timeout 10

# MPI cluster setup
# Password 123




# Worker node setup
apt install sudo
apt install libopenmpi-dev
apt install libnccl2 libnccl-dev
adduser mpiuser
usermod -aG sudo mpiuser
# Password 123
```