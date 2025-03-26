```bash
apt install libopenmpi-dev
apt install openmpi-bin openmpi-common
apt install libnccl2 libnccl-dev

make MPI=1 MPI_HOME=/usr/lib/x86_64-linux-gnu/openmpi

./build/all_reduce_perf -b 8 -e 128M -f 2 -g 2

mpirun -np 4 -N 2 -hostfile hosts.txt ./build/all_reduce_perf -b 8 -e 8G -f 2 -g 1


# MPI cluster setup
apt install sudo
adduser mpiuser
usermod -aG sudo mpiuser
# Password 123

su - mpiuser
ssh-keygen -t rsa
cd ~/.ssh
cat id_rsa.pub >> authorized_keys
ssh-copy-id worker1

```