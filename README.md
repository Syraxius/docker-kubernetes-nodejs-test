Warning: This is not working now. It will be updated when I have time.

-----

On the new machine, Docker needs to be installed first.

sudo apt-get install docker-engine

We need to then enable cgroups and swap accounting. To do so, add in /etc/defaults/grub.conf:

GRUB_CMDLINE_LINUX='cgroup_enable=memory swapaccount=1'

To apply the changes, we run:

sudo update-grub
sudo init 6

-----

To start Kubernetes, we need to:
1) Run etcd
2) Run master process
3) Run service proxy

To run etcd, we use:

docker run --volume=/var/etcd:/var/etcd --net=host -d gcr.io/google_containers/etcd:2.0.12 /usr/local/bin/etcd --addr=127.0.0.1:4001 --bind-addr=0.0.0.0:4001 --data-dir /var/etcd/data

To run the master process, we use:

sudo docker run \
--volume=/:/rootfs:ro \
--volume=/sys:/sys:ro \
--volume=/dev:/dev \
--volume=/var/lib/docker:/var/lib/docker:ro \
--volume=/var/lib/kubelet:/var/lib/kubelet:rw \
--volume=/var/run:/var/run:rw \
--net=host \
--pid=host \
--privileged=true \
-d \
gcr.io/google_containers/hyperkube:v1.0.1 \
/hyperkube kubelet --containerized --hostname-override="127.0.0.1" --address="0.0.0.0" --api-servers=http://localhost:8080 --config=/etc/kubernetes/manifests

Finally, we run the service proxy using:

sudo docker run -d --net=host --privileged gcr.io/google_containers/hyperkube:v1.0.1 /hyperkube proxy --master=http://127.0.0.1:8080 --v=2

Now we need to download kubectl to interface with the master:

wget https://storage.googleapis.com/kubernetes-release/release/v1.0.1/bin/linux/amd64/kubectl
chmod +x kubectl
mkdir bin
mv kubectl bin
. .profile
