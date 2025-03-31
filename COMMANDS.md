```bash
# Both nodes
sudo apt install -y libopenmpi-dev
sudo apt install -y openmpi-bin openmpi-common
sudo apt install -y libnccl2 libnccl-dev
sudo apt install -y sudo

cat ~/.ssh/id_ed25519.pub # Copy this

vim ~/.ssh/authorized_keys # On other node & paste in key

cd /workspace/nccl-tests
cp sample.envrc .envrc
aws_metadata_token=`curl --silent -X PUT "http://169.254.169.254/latest/api/token" -H "X-aws-ec2-metadata-token-ttl-seconds: 21600"`
echo $(curl --silent -H "X-aws-ec2-metadata-token: $aws_metadata_token" http://169.254.169.254/latest/meta-data/local-ipv4)
direnv allow

# Test connections
# From master to worker
ssh $WORKER_IP
# From worker to master
ssh $MASTER_IP

# Both nodes
make MPI=1 MPI_HOME=/opt/amazon/openmpi NCCL_HOME=/opt/nccl/build CUDA_HOME=/usr/local/cuda

# Master
export NCCL_DEBUG=TRACE
export FI_LOG_LEVEL=debug
source .envrc
/opt/amazon/openmpi/bin/mpirun \
-x FI_EFA_USE_DEVICE_RDMA=1 \
-x NCCL_DEBUG=TRACE \
-x FI_LOG_LEVEL=debug \
-x LD_LIBRARY_PATH=/opt/nccl/build/lib:/usr/local/cuda/lib64:/opt/amazon/efa/lib:/opt/amazon/openmpi/lib:/opt/amazon/ofi-nccl/lib:$LD_LIBRARY_PATH \
--verbose \
-host $MASTER_IP,$WORKER_IP \
--mca pml ^cm --mca btl tcp,self --mca btl_tcp_if_exclude lo,docker0 --bind-to none \
/workspace/nccl-tests/build/all_reduce_perf \
-b 8 \
-e 1M \
-f 2 \
-g 8 \
--timeout 10

./build/all_reduce_perf -b 8 -e 128M -f 2 -g 2

mpirun -np 4 -N 2 -hostfile hosts.txt ./build/all_reduce_perf -b 8 -e 8G -f 2 -g 1

mpirun --verbose -host $MASTER_IP,$WORKER_IP /workspace/nccl-tests/build/all_reduce_perf -b 8 -e 8G -f 2 -g 1

export NCCL_DEBUG=INFO
export NCCL_SOCKET_IFNAME=podnet1
mpirun --verbose -host $MASTER_IP,$WORKER_IP /workspace/nccl-tests/build/all_reduce_perf -b 8 -e 128M -f 2 -g 2 --timeout 10


# Doesn't work
sudo sed -i 's/^#*PasswordAuthentication .*/PasswordAuthentication yes/' /etc/ssh/sshd_config && \
sudo sed -i 's/^#*PubkeyAuthentication .*/PubkeyAuthentication yes/' /etc/ssh/sshd_config
sudo systemctl restart sshd
```