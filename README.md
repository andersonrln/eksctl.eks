# INSTALANDO EKSCTL
curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
sudo mv /tmp/eksctl /usr/local/bin

# INSTALANDO HELM
bash ./get_helm.sh 

# CRIANDO CLUSTER EKS COM EKSCTL
eksctl create cluster -f cluster.yaml --profile $PROFILE

# DELETE AWS-CNI
kubectl delete -f https://raw.githubusercontent.com/weaveworks/eksctl/d2cabf35b4508ead7ffb63252b0a0a6d3213e62d/pkg/addons/default/assets/aws-node.yaml

# INSTALL WEAVE-NET
 kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')"

https://github.com/weaveworks/weave/issues/3335#issuecomment-441522517

# CLUSTER AUTOSCALER
kubectl apply -f https://raw.githubusercontent.com/kubernetes/autoscaler/master/cluster-autoscaler/cloudprovider/aws/examples/cluster-autoscaler-autodiscover.yaml

kubectl -n kube-system annotate deployment.apps/cluster-autoscaler cluster-autoscaler.kubernetes.io/safe-to-evict="false"
kubectl -n kube-system edit deployment.apps/cluster-autoscaler


Edite o comando de contêiner cluster-autoscaler para substituir <YOUR CLUSTER NAME> pelo nome do cluster e adicione as seguintes opções.

--balance-similar-node-groups

--skip-nodes-with-system-pods=false
```yaml
    spec:
      containers:
      - command:
        - ./cluster-autoscaler
        - --v=4
        - --stderrthreshold=info
        - --cloud-provider=aws
        - --skip-nodes-with-local-storage=false
        - --expander=least-waste
        - --node-group-auto-discovery=asg:tag=k8s.io/cluster-autoscaler/enabled,k8s.io/cluster-autoscaler/<YOUR CLUSTER NAME>
        - --balance-similar-node-groups
        - --skip-nodes-with-system-pods=false
```
kubectl -n kube-system set image deployment.apps/cluster-autoscaler cluster-autoscaler=k8s.gcr.io/cluster-autoscaler:v1.14.7


# HELM 2
kubectl create clusterrolebinding kube-backup --clusterrole cluster-admin --serviceaccount=kube-system:kube-backup;
kubectl create clusterrolebinding tiller --clusterrole cluster-admin --serviceaccount=kube-system:tiller;
helm init --service-account tiller --upgrade;
# API SERVER 

 eksctl utils update-cluster-endpoints  -f cluster.yaml --approve
