---
layout: post
title:  "Kubeflow On MicroK8s"
date:   2019-06-05 00:18:23 +0900
categories: [kubernetes,kubeflow,ml,pipeline]
---

바야흐로 Container의 전성시대라 볼수 있을 것 같다. 이제는 SOA(Service Orientied Architecture)나 MSA(Micro Service Architecture) Container를 생각 하지 않을 수 없게 되었다. 이러한 Container들에 대해 관리를 위해 많은 제품들이 있지만, Kubernetes가 대세가 된지는 오래 되었고, 암묵적인 표준으로 자리 잡게 되었다. 이미, Public Cloud Provider들은 Managed Kubernetes 서비스들을 출시하고 있다. 

그리고, Machine Learning과 Bigdata와 같은 유연한 자원을 필요로 하는 분야에서도 Container Base의 Architecture를 생각 하게 되었다. 그 예로, Redis Cluster나 Kafka와 같은 MiddleWare의 Clustering은 Container Base의 Stateful한 Container를 통해 무중단의 Service가 가능한 상태의 서비스를 구축 한다. 자세한 내용은 Redis Lab의 [Redis Cluster for Kubernetes](https://sanderp.nl/running-redis-cluster-on-kubernetes-e451bda76cad)내용을 참고 하기 바란다. 

## Compute Engien Provisioning

Google Cloud Compute Engine에서 Compute Engine을 생성 한다. 이때, OS는 Ubuntu (내지는 Debian) Linux를 사용 한다. 이때 Kubeflow에서 사용하기 위해 GPU또한 Provisioning 해야 하는데, Google Cloud Compute Engine에서는 us-west1-b, europe-west4-a (NVIDIA® Tesla® P100), us-west1-b, europe-west4-a (NVIDIA® Tesla® K80)에서 사용 가능하다. 이때, Linux인스턴스나 Windows Server Instance에서 Driver를 설치 해야 하는데, 지원되는 Driver Version은 아래와 같다. 

여기서는 Unbuntu 18.04-LTS를 기준으로 설명 하고자 한다. 

> - **Linux 인스턴스:**
>
>   - NVIDIA 410.79 이상 드라이버
>
>     - 예) Install
>
>    ```shell
>    # Remove Install Driver 
>    $ sudo rm /etc/apt/sources.list.d/cuda*
>    $ sudo apt remove nvidia-cuda-toolkit
>    $ sudo apt remove nvidia-*
>    # Update OS Package
>    $ sudo apt update -y
>    # Update OS Package
>    $ sudo apt update -y
>    # GPU driver Repository Add
>    $ sudo add-apt-repository ppa:graphics-drivers/ppa
>    # Add Repository 
>   
>    $ sudo bash -c 'echo "deb http://developer.download.nvidia.com/compute/cuda/repos/ubuntu1804/x86_64 /" > /etc/apt/sources.list.d/cuda.list'
>    $ sudo bash -c 'echo "deb http://developer.download.nvidia.com/compute/machine-learning/repos/ubuntu1804/x86_64 /" > /etc/apt/sources.list.d/cuda_learn.list'
>    # All apt update & upgrade
>   
>    $ sudo apt update -y
>    $ sudo apt upgrade -y
>    # CUDA Install & libcudnn7
>    $ sudo apt install cuda-10-1 -y && sudo apt install libcudnn7 -y && sudo apt install nvidia-cuda-toolkit -y
>    # Add .profile
>    # set PATH for cuda 10.1 installation
>     if [ -d "/usr/local/cuda-10.1/bin/" ]; then
>         export PATH=/usr/local/cuda-10.1/bin${PATH:+:${PATH}}
>         export LD_LIBRARY_PATH=/usr/local/cuda-10.1/lib64${LD_LIBRARY_PATH:+:${LD_LIBRARY_PATH}}
>     fi
>    # .profile End
>    # nvcc install check
>    $ nvcc --version
>    # nvidia driver install check
>    $ nvidia-smi 
>    ```
>
> - **Windows Server 인스턴스:**
> - NVIDIA 411.98 이상 드라이버

nvidia의 Driver Version은 [여기]([http://developer.download.nvidia.com/compute/cuda/repos/](http://developer.download.nvidia.com/compute/cuda/repos/))에서 확인 할 수 있다. 



## Microk8s 설치 

GPU Driver가 설치 되면, ubuntu 18.04-lts는 snap package가 설치 되어 있다. microk8s는 [snap](https://snapcraft.io/microk8s)을 이용하여 설치 된다. 기본적으로 ubuntu Linux에는 snap이 설치 되어 있다. 

> ```shell
> $ sudo snap install microk8s --classic 
> $ microk8s.status
> microk8s is running
> addons:
> jaeger: disabled
> fluentd: disabled
> gpu: disabled
> storage: disabled
> registry: disabled
> rbac: disabled
> ingress: disabled
> dns: disabled
> metrics-server: disabled
> linkerd: disabled
> prometheus: disabled
> istio: disabled
> dashboard: disabled
> $ 
> ```

Kubernetes에서 가장 쉽게 사용하기 위해 snap을 이용하여 kubectl에 별칭을 부여 하도록 한다. 

> ```shell
> $ snap alias microk8s.kubectl kubectl
> $ kubectl get namespace
> ```
>
> 

이제 MicroK8s에서 사용 하기 위한 gpu 설정이다. 아주 간단하다. 

> ```shell
> $ microk8s.enable gpu
> Enabling NVIDIA GPU
> NVIDIA kernel module detected
> Enabling DNS
> Applying manifest
> service/kube-dns created
> serviceaccount/kube-dns created
> configmap/kube-dns created
> deployment.extensions/kube-dns created
> Restarting kubelet
> DNS is enabled
> Applying manifest
> daemonset.extensions/nvidia-device-plugin-daemonset created
> NVIDIA is enabled
> $ 
> ```
>
> 

## Kubeflow Install

사실 Microk8s에 Kubeflow를 설치 하기 위해 위의 Process를 거칠 필요는 없다.(GPU Driver 설치는 필수) Microk8s설치는 한번 짚고 넘어가는 차원에서 이야기 한것이다. 귀찮지만 새로운 VM을 띄워 2Step으로 K8s에 Kubeflow를 설치하는 도록 할 것이다. 

```shell
$ git clone https://github.com/canonical-labs/kubernetes-tools
$ sudo kubernetes-tools/setup-microk8s.sh
$ microk8s.enable gpu
```

```shell 
$ git clone https://github.com/canonical-labs/kubeflow-tools
$ kubeflow-tools/install-kubeflow.sh
```

