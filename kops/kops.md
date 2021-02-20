# Cluster Kubernetes AWS 

  - [1) Preparando Host Compartilhado](#1-preparando-host-compartilhado)
    - [1.1) Instância EC2 com Kops](#11-inst%C3%A2ncia-ec2-com-kops)
    - [1.2) Previlégios Necessários](#12-previl%C3%A9gios-necess%C3%A1rios)
    - [1.3) Criando Roles](#13-criando-roles)
    - [1.4) Instalando AWS Client](#14-instalando-aws-client)
    - [1.5) Instalando Kubectl](#15-instalando-kubectl)
    - [1.6) Instalando Kops](#16-instalando-kops)
    - [1.7) Par de Chaves](#17-par-de-chaves)            
  - [2) Route53 DNS Server](#2-route53-dns-server)

## 1) Preparando Host Compartilhado

  Nesse documento estarei assumindo que teremos uma máquina dentro da **AWS** compartilhada, essa máquina será a responsável por gerenciar o **KOPS**. Isso nos permitirá um controle mais granular de quem realmente manipulará o kops.

### 1.1) Instância EC2 com Kops
  
  Essa máquina compartilhada é apenas um provisionador, não há necessidade dessa instância EC2 ter um hardware performático. Nessa documentação foi utilizado as seguintes configurações:

```ruby
tipo: t2.micro 
s.o : awslinux
cpu : 1
mem : 1
```

![alt text](img/1-setup.png "Instância EC2")

### 1.2) Previlégios Necessários

  Para que o kops possa interagir com os serviços da **AWS**, será necessário conceder previlégios. Nada impede de possuir uma conta de usuário *IAM* e especificar dentro da máquina **KOPS** as credencias necessárias ( **AWS Access Key ID** / **AWS Secret Access Key** ).
  
  Nessa documentação iremos adotar uma outra abordagem, vamos criar uma **ROLE** e anexar essa role na instância, assim não há necessidade de 

### 1.3) Criando Roles

### 1.4) Instalando AWS Client

### 1.5) Instalando Kubectl

### 1.6) Instalando Kops

### 1.7) Par de Chaves

## 2) Route53 DNS Server