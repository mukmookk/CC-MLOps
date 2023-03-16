# Kubeflow를 활용한 MLOps 개발환경 설정

- [Kubeflow를 활용한 MLOps 개발환경 설정](#kubeflow를-활용한-mlops-개발환경-설정)
    - [Prerequisites](#prerequisites)
  - [Step 1 Install LVM](#step-1-install-lvm)
  - [Step 2 - Install Rook](#step-2---install-rook)
  - [Step 3 - Provision Storage Class and make it default](#step-3---provision-storage-class-and-make-it-default)
  - [Step 4 - Install kustomize](#step-4---install-kustomize)
  - [Step 5 - Install Kubeflow](#step-5---install-kubeflow)
  - [Step 6 - Connect to your Kubeflow Cluster (Port-Forward)](#step-6---connect-to-your-kubeflow-cluster-port-forward)


### Prerequisites
|                    |         |
|------------------------|---------|
| **kubernetes version** | >= 1.25 |
| **cpu**                | >= 0.6     |
| **Storage**            | >= 10GB    |
| **Default Storage Class**| with dynamic provisioning|

특히 default storage class를 생성하는 부분이 상당히 당황스러웠는데, minikube에서는 기본적으로 addon 형태로 달려있는 것을 확인할 있지만, kubeadm을 통해 만들어진 kubernetes 클러스터는 그렇지 않다. 직접 설정해줘야 한다.

이때 사용할 수 있는 대표적인 솔루션이 Rook과 Cephfs이다. 
> CephFS는 Standalone으로 사용할 수 있는 분산 파일 시스템이며, Rook은 Ceph(Cephfs 포함)를 배포하고 관리할 수 있는 쿠버네티스용 스토리지 오케스트레이터입니다. 

>  CephFS is a distributed file system that provides scalable and reliable storage for files. It is a component of the larger Ceph storage system and can be used as a standalone file system or as part of a Ceph cluster. CephFS is designed to provide POSIX-compliant file system semantics and is highly available and fault-tolerant.

> Rook has built-in support for Ceph, which means that it can be used to deploy and manage a Ceph cluster within a Kubernetes environment. This includes managing CephFS as a storage solution within the Kubernetes cluster. Rook abstracts the complexities of managing Ceph and exposes a simplified API for deploying and managing the storage system.

> Rook integrates with CSI to provide a standard interface for integrating different storage systems with Kubernetes. Rook uses CSI drivers to manage different storage systems, such as Ceph or CockroachDB, within a Kubernetes environment. The CSI drivers provide a standard set of APIs that Rook can use to manage the storage systems, making it easier for users to deploy and manage different storage solutions within their Kubernetes environment.

> 요약하자면, ceph는 분산 파일 스토리지, Rook은 이를 오케스트레이션해주는 서비스이다. 이때, 쿠버네티스 내의 각기 다른 파일시스템(Ceph/CockroachDB 등)에 대해서 Rook은 CSI(Container Storage Interface)에서 제공하는 Interface를 통해 비교적 쉽게 제어가 가능하다.

## Step 1 Install LVM

Ceph는 raw block device, partitions, LVM logical volumnes를 포함하여 다양한 종류의 block storage device를 사용할 수 있는데, 해당 문서에서는 이슈 발생을 최대한 피하기 위해 raw device에 LVM을 설치하는 방식으로 진행한다.

먼저 해당 머신에 ceph 구성에 활용될 수 있는 partiion이나 device가 있는지를 확인한다.
```
lsblk
lsblk -f
```
이떄 만약 FSTYPE 필드가 비어있지 않다면, 해당 장치 위에 파일 시스템이 있다는 것을 의미한다. 해당 문서에서는 *sdb*를 사용하여 Ceph를 구성하기로 한다.

```
sudo apt-get install -y lvm2
```

## Step 2 - Install Rook

현재 가장 최신 버전인 1.11.1 버전을 설치

```
git clone --single-branch --branch v1.11.1 https://github.com/rook/rook.git
```

```
cd rook/deploy/examples
kubectl create -f crds.yaml -f common.yaml -f operator.yaml
```
CRD, ClusterRole과 RoleBinding이 생성되는 것을 볼 수 있다.

이후 Rook 클러스터를 생성해준다.
```
kubectl create -f cluster.yaml
```

성공이 되었는지 확인해주자.
```
kubectl get pod -n Rook-ceph
```

## Step 3 - Provision Storage Class and make it default

ceph를 도입한 당초 목적이였던 Storage Class를 프로비저닝하고 이를 쿠버네티스의 default 스토리지로 설정하는 단계이다.

마찬가지로 앞서 Clone 해왔던 Rook의 저장소에서 다음의 주소에 있는 yaml 파일을 적용시켜 준다.

*(Rook/deploy/examples/csi/rbd)*

```
kubectl apply -f storageclass-test.yaml
```

storage 클래스가 생성되었다면, 이를 default로 설정해보자
```
kubectl patch sc rook-ceph-block -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"true"}}}'
```

## Step 4 - Install kustomize

그동안 3.2.0 버전의 kustomize를 고수해오던 kubeflow가 2023년 2월경부터 5.0.0 버전을 요구하기 시작했다. *sortOptions라는 새로운 필드를 사용하기 위해서라고 한다*

kustomize repo에 들어가서 머신에 적합한 압축파일을 받아준다. [kustomize 릴리즈](https://github.com/kubernetes-sigs/kustomize/releases)

```
wget https://github.com/kubernetes-sigs/kustomize/releases/download/kustomize%2Fv5.0.1/kustomize_v5.0.1_linux_amd64.tar.gz
```

압축 풀어주고 
```
tar -xvfz kustomize_v5.0.1_linux_amd64.tar.gz
```

PATH 경로로 옮겨주면 구성 완료
```
mv kustomize /usr/local/bin
```

버전 확인해주며 마무리
```
kustomize version
```

## Step 5 - Install Kubeflow

쿠버네티스를 설치하는 방식에는 크게 두 가지가 있다.
1. Single-command installation of all components under apps and common
2. Multi-command, individual components installation for apps and common

2번의 경우 각각의 개별적인 구성 요소들을 독립적으로 설치하는 옵션이다. 해당 [사이트](https://github.com/kubeflow/manifests#connect-to-your-kubeflow-cluster)에 들어간 이후 설명대로 복사 붙여넣기만 하면 된다. 다만, 상당히 귀찮은 것을 확인할 수 있다. 

해당 문서에서 진행하고자 하는 것은 Single-command로 명령어 한줄로 manifest들을 적용하여 kubeflow의 모든 구성요소를 설치하는 옵션이다. 

manifest를 일단 가져와서 준비해준다.
```
cd ~
git clone https://github.com/kubeflow/manifests.git
cd manifests
```

다음 명령어를 통해 설치를 진행한다.
```
while ! kustomize build example | awk '!/well-defined/' | kubectl apply -f -; do echo "Retrying to apply resources"; sleep 10; done
```

약 10분 가량의 시간이 흐르고 설치가 완료되었다.

## Step 6 - Connect to your Kubeflow Cluster (Port-Forward)

설치가 완료되었다면, Pod이 완전히 다 뜨는 것을 확인 후에 다음 단계로 넘어가자
```
kubectl get pods -n cert-manager
kubectl get pods -n istio-system
kubectl get pods -n auth
kubectl get pods -n knative-eventing
kubectl get pods -n knative-serving
kubectl get pods -n kubeflow
kubectl get pods -n kubeflow-user-example-com
```

kubeflow에 접근하는 기본적인 방식은 Port-forwarding이며, 이를 통해 해당 kubernets 환경에 별도 요구사항 없이도 kubeflow와 유저 간의 연결이 가능해진다.

앞서 설치된 Istio의 ingressgateway를 port-forwarding하여 Local Port 8080을 Remote port 80(pod 쪽)로 연결한다.

이후 다음을 통해 Kubeflow Central Dashboard에 접속한다.
1. 브라우저를 열고 `http://localhost:8080`으로 접속한다. Dex login 화면이 보인다면 성공
2. default user credential로 로그인한다. default email 주소는 `user@example.com`이고 비밀번호는 `12341234`이다.

> NodePort/LoadBalancer/Ingress를 사용하기 위해서는 HTTPS를 사용해야만 한다. 이는 kubeflow의 많은 web app들이 Secure Cookie를 사용하고 있기 때문이다.