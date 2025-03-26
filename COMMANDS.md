```bash
apt install libopenmpi-dev
apt install libnccl2 libnccl-dev

make MPI=1 MPI_HOME=/usr/lib/x86_64-linux-gnu/openmpi

./build/all_reduce_perf -b 8 -e 128M -f 2 -g 2

mpirun -np 4 -N 2 --allow-run-as-root ./build/all_reduce_perf -b 8 -e 8G -f 2 -g 1
```