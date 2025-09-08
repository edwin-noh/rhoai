# Red Hat OpenShift Day2 GitOps

## Usecase
공통 부분과 오버레이 부분을 식별하기 위해 몇가지 유스케이스를 살펴본다.
예를 들어, 일반적인 클러스터를 설정하는 과정
1. 오픈시프트 설정
- OpenShift GitOps Operator 설치
- ArgoCD instance 설치
- ArgoCD 설정 (AppProject, RBAC, Repo, Notification, Plugins)
- 필요 Operator 설치 및 설정 (Tekton, Service Mesh 등) **cluster config applicationset**등과 같이 ArgoCD를 통해 설치
- 필요 컴포넌트 설정 (NFS Provisioner 등)  **cluster config applicationset**등과 같이 ArgoCD를 통해 설치
2. RHOAI 설정
분기되는 부분만 확인하면
필요 Operator 설치, 설정이 틀리고...
MinIO등 필요 컴포넌트 설정이 틀림
3. 같은 오픈시프트 구성이지만 버전이 틀린 경우
전체 과정에서 GitOps등과 같이 오퍼레이터 버전 등이 틀림

**결론**
클러스터 타입과 버전별로 오버레이 구성

## Prerequisite
1. Create namespace
``` bash
% oc create ns openshift-gitops-operator
# Optional
% oc label namespace openshift-gitops-operator openshift.io/cluster-monitoring=true
```
2. Create Operator group
``` bash
cat <<EOF > "gitops-operator-group.yaml"
apiVersion: operators.coreos.com/v1
kind: OperatorGroup
metadata:
  name: openshift-gitops-operator
  namespace: openshift-gitops-operator
spec:
  upgradeStrategy: Default
EOF

oc apply -f gitops-operator-group.yaml
```
3. Create subscription
``` bash
cat <<EOF > "openshift-gitops-sub.yaml"
apiVersion: operators.coreos.com/v1alpha1
kind: Subscription
metadata:
  name: openshift-gitops-operator
  namespace: openshift-gitops-operator
spec:
  channel: latest 
  installPlanApproval: Automatic
  name: openshift-gitops-operator
  source: redhat-operators
  sourceNamespace: openshift-marketplace
EOF

oc oc apply -f openshift-gitops-sub.yaml
```


## Design Concept

### 1. BootStrap 
1.1. Base - Bootstrap process is installing argocd instnace and argocd components
1.2. Overlays - selecting cluster. i.e. rhoai, application, hub cluster.
### 2. Cluster
2.1. Base - basic setting of cluster...maybe app project of argocd, nfs provisioner...
2.2. Overlays - Cluster overlays have a set of operators and installation manifest to enable cluster for the purpose via applicationset. 
### 3. 
Components have operators and tools for the clusters

Git repo to install and configure RHOAI in GitOps way through ArgoCD.

``` sh

```