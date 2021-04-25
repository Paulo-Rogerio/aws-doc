# 🚀  Estudos AWS  

- [1) AWS](#1-aws)
  - [1.1) Volumes EBS](#11-volumes-ebs)
    - [1.1.1) SnapShot AMI](#111-snapshot-ami)
    - [1.1.2) Volumes e EC2](#112-volumes-e-ec2)
  - [1.2) LoadBalancer](#12-loadbalancer)
  - [1.3) Virtual Private Cloud](#13-virtual-private-cloud)
    - [1.3.1) Criando VPC](#131-criando-vpc)
  - [1.4) Pfsense como Concentrador](#14-pfsense-como-concentrador)
- [2) Kubernetes](#2-kubernetes)
  - [2.1) Provisionando Cluster](#21-provisionando-cluster)
    - [2.1.1) Kops](#211-kops)
  - [2.2) Deploy de Manifestos](#22-deploy-de-manifestos)
    - [2.2.1) Helm Chart](#221-helm-chart)
      
## 1) AWS

  Esse material de estudo pode ser usado por qualquer pessoa que utiliza **AWS** como *Cloud Provider*. Nos tópicos abaixo serão utilizados Instâncias EC2 baseadas em **AWS Linux** e **Ubuntu**. 

  A documentação foi realizada usando uma conta **Free**, caso queira reproduzi-la atente-se em destruir o ambiente após o apredizado para evitar consumo do seus créditos.
  
  A ordem dos tópicos acima foram criadas de acordo com a necessidade de implantar os recursos na AWS. 

#### 1.1) Volumes EBS

  EBS é a forma de *Armazenamento de Bloco* utilizada pela AWS. No tópico abaixo será mostrado como fazer um *SnapShot da EC2* e restaurá-la em outra Instância.

#### 1.1.1) SnapShot AMI

  * [SnapShot AMI](https://github.com/Paulo-Rogerio/aws-doc/blob/main/aws-resources/volumes-ebs/snapshot-ami/snapshot-ami.md)

#### 1.1.2) Volumes e EC2

  * [Volumes e EC2](https://github.com/Paulo-Rogerio/aws-doc/blob/main/aws-resources/volumes-ebs/volumes/volumes-ec2.md)

#### 1.2) LoadBalancer

  * [LoadBalancer com EC2](https://github.com/Paulo-Rogerio/aws-doc/blob/main/aws-resources/loadbalancer/loadbalancer.md)

#### 1.3) Virtual Private Cloud

É uma rede virtual dedicada à sua conta da AWS. É composta de Sub-rede, Tabela de Roteamento,Internet Gateway além de um intervalo de endereços IPs. No tópico abaixo vamos entrar mais detalhadamente nesse assunto.

#### 1.3.1) Criando VPC
  * [Criando Minha VPC](https://github.com/Paulo-Rogerio/aws-doc/blob/main/aws-resources/vpc/vpc.md)

#### 1.4) Pfsense como Concentrador
  * [Criando Conentrador com PfSense](https://github.comPaulo-Rogerio/aws-doc/blob/main/aws-resources/pfsense/pfsense.md)

## 2) Kubernetes
#### 2.1) Provisionando Cluster

#### 2.1.1) Kops
  * [Kops Kubernetes](https://github.com/Paulo-Rogerio/aws-doc/blob/main/kubernetes/kops/kops.md)
 
#### 2.2) Deploy de Manifestos

  Empacotando os manifestos usando o Helm Chart e aplicando-os no Kubernetes.
  
#### 2.2.1) Helm Chart
  * [Aplicando Maninfestos com Helm](https://github.com/Paulo-Rogerio/aws-doc/blob/main/kubernetes/helm/helm.md)
