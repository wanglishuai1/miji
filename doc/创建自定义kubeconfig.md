
# 创建自定kubeconfig
```shell
#!/bin/bash

# 创建一个k8s用户，并赋予与csi-ls-node-sa相同的权限
UserName=wanglishuai
ApiServerEndpoints=`awk '$0~/server/{print $NF}' ~/.kube/config`
ClusterName=qa-test
NS=kube-system
mkdir -p /etc/kubernetes/pki/client/${UserName}
cd /etc/kubernetes/pki/client/${UserName}

# 创建用户证书
openssl genrsa -out ${UserName}.key 2048
openssl req -new -key ${UserName}.key -out ${UserName}.csr -subj "/CN=${UserName}"
openssl x509 -req -in ${UserName}.csr -CA /etc/kubernetes/pki/ca.crt \
-CAkey /etc/kubernetes/pki/ca.key -CAcreateserial -out ${UserName}.crt -days 3650

# 创建kubeconfig文件
kubectl config set-cluster ${ClusterName} \
  --embed-certs=true \
  --server=${ApiServerEndpoints} \
  --certificate-authority=/etc/kubernetes/pki/ca.crt \
  --kubeconfig=${UserName}.kubeconfig

kubectl config set-credentials ${UserName} \
  --client-certificate=${UserName}.crt \
  --client-key=${UserName}.key \
  --embed-certs=true \
  --kubeconfig=${UserName}.kubeconfig

kubectl config set-context ${UserName}@${ClusterName} \
  --cluster=${ClusterName} \
  --user=${UserName} \
  --namespace=${NS} \
  --kubeconfig=${UserName}.kubeconfig

kubectl config use-context ${UserName}@${ClusterName} --kubeconfig=${UserName}.kubeconfig

# 创建RBAC规则(把创建的用户绑定已经有了的clusterrole)
kubectl create clusterrolebinding ${UserName}-binding \
  --clusterrole=ls-external-provisioner-role \
  --user=${UserName}

```