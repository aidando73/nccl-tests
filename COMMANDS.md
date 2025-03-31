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
export FI_EFA_USE_DEVICE_RDMA=1
export NCCL_DEBUG=TRACE
export FI_LOG_LEVEL=debug
/opt/amazon/openmpi/bin/mpirun \
-x FI_EFA_USE_DEVICE_RDMA=1 \
-x NCCL_DEBUG=TRACE \
-x FI_LOG_LEVEL=debug \
--verbose \
-host $MASTER_IP,$WORKER_IP \
/usr/local/cuda-12.4/efa/test-cuda-12.4/all_reduce_perf \
-b 8 \
-e 256M \
-f 2 \
-g 8 \
--timeout 10

```