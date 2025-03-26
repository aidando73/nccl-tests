```bash
apt install mpich
apt install libnccl2 libnccl-dev

make MPI=1 MPI_HOME=/usr/include/x86_64-linux-gnu/mpich
```