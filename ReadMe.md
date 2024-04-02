# Pre-requsite
- Disable the selinux
- Configure the default storage class, eg: nfs-client

- To configure NFS Storage Class

  Execute the below command on the node which you want to be the NFS server.
```  
  yum install nfs-utils -y
  mkdir /var/nfs/general -p
  chown -R nobody /var/nfs/general
  chmod -R 755 /var/nfs/general
  echo '/var/nfs/general *(rw,sync,no_root_squash,no_subtree_check)' > /etc/exports
  systemctl restart nfs-server
  systemctl enable nfs-server
```
- Install NFS utilities in all of the nodes in the cluster.
  yum install nfs-utils -y

- Add the Repo, and update
  helm repo add nfs-subdir-external-provisioner https://kubernetes-sigs.github.io/nfs-subdir-external-provisioner
  helm repo update

- Install the helm release, change the IPAddress to NFS server IP, and path as per the env.
  helm install nfs-subdir-external-provisioner nfs-subdir-external-provisioner/nfs-subdir-external-provisioner --set nfs.server=172.31.36.141 --set nfs.path=/var/nfs/general

- Make the SC nfs-client as default
  kubectl patch storageclass nfs-client -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"true"}}}'


# Install Prometheus, Grafana

kubectl create ns monitoring

helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
helm install prometheus prometheus-community/kube-prometheus-stack --namespace monitoring -f kube-prometheus-stack_values.yaml

# Download the mke.toml file, change the cadvisor value to true, and then upload the mke.toml. This will install the cadvisor as a daemonset in kube-system.

Define the following environment variables:

export MKE_USERNAME=<mke-username>
export MKE_PASSWORD=<mke-password>
export MKE_HOST=<mke-fqdm-or-ip-address>

Obtain and define an AUTHTOKEN environment variable:

AUTHTOKEN=$(curl --silent --insecure --data '{"username":"'$MKE_USERNAME'","password":"'$MKE_PASSWORD'"}' https://$MKE_HOST/auth/login | jq --raw-output .auth_token)

Download the current MKE configuration file.
curl --silent --insecure -X GET "https://$MKE_HOST/api/ucp/config-toml" -H "accept: application/toml" -H "Authorization: Bearer $AUTHTOKEN" > mke-config.toml

Edit the MKE configuration file, as needed. For comprehensive detail, refer to Configuration options.

Upload the newly edited MKE configuration file:

Note
You may need to reacquire the AUTHTOKEN, if significant time has passed since you first acquired it.
curl --silent --insecure -X PUT -H "accept: application/toml" -H "Authorization: Bearer $AUTHTOKEN" --upload-file 'mke-config.toml' https://$MKE_HOST/api/ucp/config-toml

# Install the Cadvisor for container metrics

Install the Cadvisor ServiceMonitor
kubectl apply -f cadvisor-servicemonitor.yaml

# Install Loki
helm repo add grafana https://grafana.github.io/helm-charts
helm repo update
helm install loki grafana/loki-stack -f loki-stack_values.yaml -n monitoring


# Login to Grafana, add the prometheus as a datasource
http://prometheus-kube-prometheus-prometheus:9090
Add the dashboard, 15758, 15757, 15760, 15759
