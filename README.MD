# Preparation

## Install kubectl
```
brew install kubernetes-cli
```

## Install helm
```
$ brew install kubernetes-helm
$ helm repo add gitlab https://charts.gitlab.io/
$ helm repo update
```

## Install eksctl
```
brew tap weaveworks/tap
brew install weaveworks/tap/eksctl
```

## Clone the gitlab k8s depolyment repository
```
git clone https://gitlab.com/gitlab-org/charts/gitlab.git
cd gitlab/
chmod +x ./*
chmod +x ./*.sh
```

```
$ ./scripts/eks_bootstrap_script up
...
[✔]  EKS cluster "gitlab-cluster" in "ap-northeast-1" region is ready

$ ./scripts/eks_bootstrap_script list
NAME    REGION
gitlab-cluster  ap-northeast-1
```

---------
# Option 1

```
export NAMESPACE=gitlab
```

```
$ helm upgrade --install $NAMESPACE gitlab/gitlab \
  --namespace $NAMESPACE \
  --values gitlab_config.yaml
```

```
$ kubectl get pods --namespace $NAMESPACE
$ kubectl get --namespace $NAMESPACE services
```

```
$ kubectl describe service $NAMESPACE-nginx-ingress-controller --namespace $NAMESPACE | grep Ingress
```

## Get root's password
```
kubectl get secret gitlab-gitlab-initial-root-password \
  --namespace $NAMESPACE \
  -ojsonpath='{.data.password}' | base64 --decode ; echo
```

https://gitlab.k8s.xulei.app/admin/runners

```
$ helm install --namespace $NAMESPACE --name gitlab-runner -f gitlab_runner_config.yaml gitlab/gitlab-runner
```

```
$ helm ls --all $NAMESPACE
$ helm delete gitlab --purge
```

---------
# Option 2

```
export NAMESPACE=gitlab-eks
```

```
$ openssl req -x509 -nodes -days 365 \
  -newkey rsa:2048 -keyout eks.xulei.app.key \
  -out eks.xulei.app.pem \
  -subj "/CN=*.eks.xulei.app/O=*.eks.xulei.app"

$ kubectl create secret tls eks.xulei.app --key eks.xulei.app.key --cert eks.xulei.app.pem

```

```
$ helm upgrade --install $NAMESPACE gitlab/gitlab \
  --namespace $NAMESPACE \
  --values gitlab_config_eks.yaml
```

```
$ kubectl get pods --namespace $NAMESPACE
$ kubectl get --namespace $NAMESPACE services
```

```
$ kubectl describe service $NAMESPACE-nginx-ingress-controller --namespace $NAMESPACE | grep Ingress
```

## Get root's password
```
kubectl get secret $NAMESPACE-gitlab-initial-root-password \
  --namespace $NAMESPACE \
  -ojsonpath='{.data.password}' | base64 --decode ; echo
```


```
$ helm ls --all $NAMESPACE
$ helm delete $NAMESPACE --purge
```


---------
# Option 3

```
export NAMESPACE=gitlab
```

```
$ openssl req -x509 -nodes -days 365 \
  -newkey rsa:2048 -keyout k8s.xulei.app.key \
  -out k8s.xulei.app.pem \
  -subj "/CN=*.k8s.xulei.app/O=*.k8s.xulei.app"

$ kubectl create secret tls k8s.xulei.app --key k8s.xulei.app.key --cert k8s.xulei.app.pem

```

```
$ helm upgrade --install $NAMESPACE gitlab/gitlab \
  --namespace $NAMESPACE \
  --values gitlab_config.yaml
```

```
$ kubectl get pods --namespace $NAMESPACE
$ kubectl get --namespace $NAMESPACE services
```

```
$ kubectl describe service $NAMESPACE-nginx-ingress-controller --namespace $NAMESPACE | grep Ingress
```

## Get root's password
```
kubectl get secret $NAMESPACE-gitlab-initial-root-password \
  --namespace $NAMESPACE \
  -ojsonpath='{.data.password}' | base64 --decode ; echo
```


```
$ helm ls --all $NAMESPACE
$ helm delete $NAMESPACE --purge
```

---------


https://dev.classmethod.jp/cloud/aws/deploy-the-gitlab-on-eks-with-helm/
https://docs.gitlab.com/charts/installation/cloud/eks.html
https://docs.gitlab.com/charts/installation/deployment.html
https://docs.gitlab.com/charts/installation/tls.html
https://docs.gitlab.com/charts/charts/globals.html

https://docs.aws.amazon.com/eks/latest/userguide/alb-ingress.html
https://kubernetes-sigs.github.io/aws-alb-ingress-controller/guide/controller/setup/
https://kubernetes-sigs.github.io/aws-alb-ingress-controller/guide/external-dns/setup/