# 🚀  Estudos AWS  

- [1) AWS](#1-aws)
  - [1.1) Volumes EBS](#11-volumes-ebs)
    - [1.1.1) SnapShot e Restore](#111-snapshot-e-restore)
  - [1.2) LoadBalancer](#12-loadbalancer)    
- [2) Kubernetes](#2-kubernetes)
  - [2.1) Provisionando Cluster](#21-provisionando-cluster)
    - [2.1.1) Kops](#211-kops)
  - [2.2) Deploy de Manifestos](#22-deploy-de-manifestos)
    - [2.2.1) Helm](#221-helm)  

## 1) AWS

  Esse material de estudo pode ser usado por qualquer pessoa que utiliza **AWS** como *Cloud Provider*. Nos tópicos abaixo serão utilizados Instâncias EC2 baseadas em **AWS Linux** e **Ubuntu**. 

  A documentação foi realizada usando uma conta **Free**, caso queira reproduzi-la atente-se em destruir o ambiente após o apredizado para evitar consumo do seus créditos.
  
  A ordem dos tópicos acima foram criadas de acordo com a necessidade de implantar os recursos na AWS. 

#### 1.1) Volumes EBS

  EBS é a forma de *Armazenamento de Bloco* utilizada pela AWS. No tópico abaixo será mostrado como fazer um *SnapShot da EC2* e restaurá-la em outra Instância.

#### 1.1.1) SnapShot e Restore

  * [SnapShot - AMI](https://github.com/Paulo-Rogerio/aws-doc/blob/main/aws-resources/volumes-ebs/snapshot/snapshot.md).

#### 1.2) LoadBalancer

  * [LoadBalancer](https://github.com/Paulo-Rogerio/aws-doc/blob/main/aws-resources/loadbalancer/loadbalancer.md)

## 2) Kubernetes
#### 2.1) Provisionando Cluster
#### 2.1.1) Kops
  * [Kops Kubernetes](https://github.com/Paulo-Rogerio/aws-doc/blob/main/kubernetes/kops/kops.md)
 
#### 2.2) Deploy de Manifestos
#### 2.2.1) Helm
  * [Helm](https://github.com/Paulo-Rogerio/aws-doc/blob/main/kubernetes/helm/helm.md)
