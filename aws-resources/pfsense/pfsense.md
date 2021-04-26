# Usando Pfsense como Concentrador de Redes na AWS

- [1) Porque usar Pfsense como concentrador?](#1-porque-usar-pfsense-como-concentrador)
- [2) Qual cenário vamos abordar?](#2-qual-cenário-vamos-abordar)  
- [3) VPCs](#3-vpcs)
  - [3.1) Primeira VPC](#31-primeira-vpc)
    - [3.1.1) Criando VPC](#311-criando-vpc)
    - [3.1.2) Criando SubNet](#312-criando-subnet)
    - [3.1.3) Criando Internet Gateway](#313-criando-internet-gateway) 
    - [3.1.4) Tabela de Roteamento](#314-tabela-de-roteamento)   
  - [3.2) Segunda VPC](#32-primeira-vpc)
    - [3.2.1) Criando VPC](#321-criando-vpc)
    - [3.2.2) Criando SubNet](#322-criando-subnet)
    - [3.2.3) Criando Internet Gateway](#323-criando-internet-gateway) 
    - [3.2.4) Tabela de Roteamento](#324-tabela-de-roteamento)
- [4) EC2 PfSense ](#4-ec2-pfsense)
    - [4.1) Criando EC2](#41-criando-ec2)
    - [4.2) Conectando na Instância](#42-conectando-na-instância)
    - [4.3) Adicionando Usuário Admin](#43-adicionando-usuário-admin)
    - [4.4) Configurações Básicas](#44-configurações-básicas)
    - [4.5) Certificado Web Browser](#45-certificado-web-browser)
    - [4.6) Adicionando Interface de Rede](#46-adicionando-interface-de-rede)
    - [4.7) Desabilitando Firewall](#47-desabilitando-firewall)
    - [4.8) Adicionando Ip Estático na EC2](#48-adicionando-ip-estático-na-ec2)
- [5) OpenVPN](#5-openvpn)
    - [5.1) Criando uma CA](#51-criando-uma-ca)
    - [5.2) Criando uma Certificado](#52-criando-um-certificado)
    - [5.3) Criando para Usuário](#53-certificado-para-usuario)
    - [5.4) Instalação de Pacotes](#54-instalação-de-pacotes)
        - [5.4.1) FreeRadios](#541-freeradios)
        - [5.4.2) OpenVPN Exporter](#542-openvpn-exporter)
    - [5.5) FreeRadios Configurações](#55-freeradios-configurações)
        - [5.5.1) Interfaces](#551-interfaces)
        - [5.5.2) Nas Clients](#552-nas-clients)
        - [5.5.3) Usuário FreeRadios](#553-usuário-freeradios)    
    - [5.6) OpenVPN Wizard](#56-openvpn-wizard)
    - [5.7) Adicionando Usuário de VPN](#57-adicionando-usuário-de-vpn)
    - [5.8) Exportando Configuração do Cliente](#58-exportando-configuração-do-cliente)
    - [5.9) VPN Connect](#59-vpn-connect)
- [6) IPSec](#6-ipsec)
    - [6.1) IPSec Wizard](#61-ipsec-wizard)
        - [6.1.1) Permissões AWS Necessárias](#611-permissões-aws-necessárias)
        - [6.1.2) Pacotes Pfsense Necessários](#612-pacotes-pfsense-necessários)
        - [6.1.3) Fechando Túnel](#613-fechando-tunel)
    - [6.2) IPSec Manualmente](#62-ipsec-manualmente)
        - [6.2.1) Fechando com Primeiro IP Público](#621-fechando-com-primeiro-ip-público)
            - [6.2.1.1) Fase 1](#6211-fase-1)
            - [6.2.1.1) Fase 2](#6211-fase-2)
        - [6.2.2) Fechando com Segundo IP Público](#622-fechando-com-segundo-ip-público)
            - [6.2.2.1) Fase 1](#6221-fase-1)
            - [6.2.2.1) Fase 2](#6221-fase-2)
    - [6.3) Fechando Conexão com Primeiro IP Público](#63-fechando-conexão-com-primeiro-ip-público)
    - [6.4) Fechando Conexão com Segundo IP Público](#63-fechando-conexão-com-segundo-ip-público)        

## 1) Porque usar Pfsense como concentrador?

A própria **AWS** forcesse um serviço de Túnel VPN entre as VPC, porém isso é cobrado. O objetivo é utilizar o **pfSense** para interligar as redes VPC.

## 2) Qual cenário vamos abordar?

Teremos **2 VPC** na mesma região, cada qual com sua respectiva faixa de IP. O desenho abaixo ilustrará melhor o papel do Pfsense.

![alt text](img/diagrama-rede/1.png)

## 3) VPCs

### 3.1) Primeira VPC

Vamos criar nossa **Primeira VPC**. Observer que não temos nenhuma VPC associada nessa conta.

![alt text](img/primeira-vpc/vpc/1.png)

#### 3.1.1) Criando VPC

Os passos abaixos irá criar uma **Nova VPC**, a faixa de Rede usada será: **10.170.0.0/16**

![alt text](img/primeira-vpc/vpc/2.png)

![alt text](img/primeira-vpc/vpc/3.png)

![alt text](img/primeira-vpc/vpc/4.png)

Habilitando essa flag, fará com que as EC2 criadas nessa VPC já tenham os nomes das instâncias associadas ao DNS da AWS.

![alt text](img/primeira-vpc/vpc/5.png)

#### 3.1.2) Criando SubNet

Localize o menu **Subnet**, e clique no menu **Create subnet**, conforme mostra na imagem abaixo.

![alt text](img/primeira-vpc/subnet/1.png)

Escolha qual VPC irá associar a essa subnet.

![alt text](img/primeira-vpc/subnet/2.png)

Um detalhe importante é que podemos criar várias subnets dentro de uma VPC, para esse material iremos criar uma Subnet na classe C, ou seja, a subnet terá o seguinte endereço de rede: **10.170.1.0/24**

![alt text](img/primeira-vpc/subnet/3.png)

Para que uma **EC2** possa ter um serviço acessado **externamente**, ou seja, que outras pessoas possam acessar o conteúdo dessa **EC2** é necessário que ela tenha um **IP Público** vinculado a essa **EC2**, quando habilitado essa flag dentro da subnet, em outras palavras, vc está permitindo que essa **EC2** vinculada a essa Subnet possa ser acessada externamente por um **IP Publico**.

**Obs.:** Caso queira criar uma **Subnet apenas com acesso interno**, basta *NÃO* habilitar o recurso abaixo. 

![alt text](img/primeira-vpc/subnet/4.png)

![alt text](img/primeira-vpc/subnet/5.png)

#### 3.1.3) Criando Internet Gateway

No menu localize **Internet Gateway**, e clique em **Create internet gateway**.

![alt text](img/primeira-vpc/internet-gateway/1.png)

![alt text](img/primeira-vpc/internet-gateway/2.png)

![alt text](img/primeira-vpc/internet-gateway/3.png)

Após criado deve-se attachar qual VPC estará vinculada a esse internet gateway.

![alt text](img/primeira-vpc/internet-gateway/4.png)


#### 3.1.4) Tabela de Roteamento   

É aqui que vamos definir se a subnet pode acessar a Internet, devemos criar uma rota default apontando para o gateway de internet criado no passo anterior.

![alt text](img/primeira-vpc/route-table/1.png)

![alt text](img/primeira-vpc/route-table/2.png)

Adiconando a rota default para internet.

![alt text](img/primeira-vpc/route-table/3.png)

![alt text](img/primeira-vpc/route-table/4.png)

Por fim precisamos associar qual subnet essa configuração de rota irá aplicar-se.

![alt text](img/primeira-vpc/route-table/5.png)


### 3.2) Segunda VPC

Vamos criar nossa **Segunda VPC**. 

#### 3.2.1) Criando VPC

Basicamente iremos realizar os mesmos passos feitos na criação da primeira VPC, será enfatizado apenas os aspectos específicos dessa segunda VPC.

Será criado uma VPC para atender o ambiente de desenvolvimento, cuja faixa de rede é: **10.160.0.0/16**

![alt text](img/segunda-vpc/vpc/1.png)

![alt text](img/segunda-vpc/vpc/2.png)

#### 3.2.2) Criando SubNet

Criar uma nova subnet e vincular e esssa recém criada **VPC => 10.160.0.0/16**

![alt text](img/segunda-vpc/subnet/1.png)

A subnet que será criada terá o seguinte endereço associado: **10.160.1.0/24**

![alt text](img/segunda-vpc/subnet/2.png)

#### 3.2.3) Criando Internet Gateway

Agora vamos criar um outro internet gateway e vincular a segunda VPC **10.160.0.0/24**

![alt text](img/segunda-vpc/internet-gateway/1.png)

Depois de criado deve-se atachar qual **VPC** esse internet gateway irá atender.

![alt text](img/segunda-vpc/internet-gateway/2.png)

![alt text](img/segunda-vpc/internet-gateway/3.png)

#### 3.2.4) Tabela de Roteamento


Vamos definir se a subnet pode acessar a Internet criando uma rota default apontando para o gateway de internet criado no passo anterior.

![alt text](img/segunda-vpc/route-table/1.png)

![alt text](img/segunda-vpc/route-table/2.png)

Adiconando a rota default para internet.

![alt text](img/segunda-vpc/route-table/3.png)

Por fim precisamos associar qual subnet essa configuração de rota irá aplicar-se.

![alt text](img/segunda-vpc/route-table/4.png)

## 4) EC2 PfSense

### 4.1) Criando EC2

Nesse passo a passo vamos criar uma Instância EC2 com Pfsense.

![alt text](img/ec2/pfsense/startup/1.png)

![alt text](img/ec2/pfsense/startup/2.png)

![alt text](img/ec2/pfsense/startup/3.png)

![alt text](img/ec2/pfsense/startup/4.png)

![alt text](img/ec2/pfsense/startup/5.png)

![alt text](img/ec2/pfsense/startup/6.png)

![alt text](img/ec2/pfsense/startup/7.png)

### 4.2) Conectando na Instância

Para descobrir a senha gerada pelo instalador, será necessário fazer um snapshot do boot do Sistema onde a senha é visualizada no browser, para realizar o primeiro acesso.

![alt text](img/ec2/pfsense/conectando-pfsense/1.png)

![alt text](img/ec2/pfsense/conectando-pfsense/2.png)

![alt text](img/ec2/pfsense/conectando-pfsense/3.png)

### 4.3) Adicionando Usuário Admin

Após login, vamos dar inicio as configurações inicias. A primeira configuração a ser realizada, será a criação de um usuário com perfil **Admin**.

![alt text](img/ec2/pfsense/user-admin/1.png)

![alt text](img/ec2/pfsense/user-admin/2.png)

![alt text](img/ec2/pfsense/user-admin/3.png)

![alt text](img/ec2/pfsense/user-admin/4.png)

![alt text](img/ec2/pfsense/user-admin/5.png)

![alt text](img/ec2/pfsense/user-admin/6.png)

### 4.4) Configurações Básicas

Algumas configurções básicas são necessárias para garantir acesso externo do Pfsense para internet.

![alt text](img/ec2/pfsense/configuracoes-basicas/1.png)

![alt text](img/ec2/pfsense/configuracoes-basicas/2.png)

![alt text](img/ec2/pfsense/configuracoes-basicas/3.png)

![alt text](img/ec2/pfsense/configuracoes-basicas/4.png)

### 4.5) Certificado Web Browser

Se vc tem um certificado válido para um domínio, pode-se usá-lo no frontend do Pfsense.

![alt text](img/ec2/pfsense/certificado-web/1.png)

![alt text](img/ec2/pfsense/certificado-web/2.png)

![alt text](img/ec2/pfsense/certificado-web/3.png)

### 4.6) Adicionando Interface de Rede

Nessa etapa , vamos adicionar uma segunda interface de rede. Essa interface funcionará como uma rede local. 

![alt text](img/ec2/pfsense/interface-rede/1.png)

![alt text](img/ec2/pfsense/interface-rede/2.png)

![alt text](img/ec2/pfsense/interface-rede/3.png)

![alt text](img/ec2/pfsense/interface-rede/4.png)

![alt text](img/ec2/pfsense/interface-rede/5.png)

![alt text](img/ec2/pfsense/interface-rede/6.png)

![alt text](img/ec2/pfsense/interface-rede/7.png)

![alt text](img/ec2/pfsense/interface-rede/8.png)

![alt text](img/ec2/pfsense/interface-rede/9.png)

### 4.7) Desabilitando Firewall

Nesse momento, o objtivo é concentrar o acesso e não bloquear os acessos, por isso, vamos desabilitar o firewall.

![alt text](img/ec2/pfsense/desabilitar-firewall/1.png)

![alt text](img/ec2/pfsense/desabilitar-firewall/2.png)

![alt text](img/ec2/pfsense/desabilitar-firewall/3.png)

![alt text](img/ec2/pfsense/desabilitar-firewall/4.png)

![alt text](img/ec2/pfsense/desabilitar-firewall/5.png)

![alt text](img/ec2/pfsense/desabilitar-firewall/6.png)

### 4.8) Adicionando Ip Estático na EC2

Nessa etapa vamos adicionar um **IP Fixo** para facilitar os acessos via VPN, para isso desligue a VM e localize o menu, **Elastic IPS**

![alt text](img/ec2/pfsense/ip-elastic/1.png)

![alt text](img/ec2/pfsense/ip-elastic/2.png)

![alt text](img/ec2/pfsense/ip-elastic/3.png)

![alt text](img/ec2/pfsense/ip-elastic/4.png)

![alt text](img/ec2/pfsense/ip-elastic/5.png)

![alt text](img/ec2/pfsense/ip-elastic/6.png)


## 5) OpenVPN

### 5.1) Criando uma CA

Nessa etapa vamos configurar o OpenVPN com autenticação baseada em FreeRadios + Google Autentication. Para iniciarmos os trabalhos, vamos criar primeiramente o Autoridade Certificadora auto assinado.

![alt text](img/ec2/pfsense/openvpn/ca/1.png)

![alt text](img/ec2/pfsense/openvpn/ca/2.png)

![alt text](img/ec2/pfsense/openvpn/ca/3.png)

![alt text](img/ec2/pfsense/openvpn/ca/4.png)

### 5.2) Criando uma Certificado

Agora vamos criar um certificado para nosso serviço.

![alt text](img/ec2/pfsense/openvpn/certificado/1.png)

![alt text](img/ec2/pfsense/openvpn/certificado/2.png)

![alt text](img/ec2/pfsense/openvpn/certificado/3.png)

### 5.3) Criando para Usuário

Cada usuário que precisar conectar-se em nossa VPN precisará de um certificado assinado pela certificadora **OpenVPN_CA**, gerada nos passos anteriores.

![alt text](img/ec2/pfsense/openvpn/certificado-usuario/1.png)

![alt text](img/ec2/pfsense/openvpn/certificado-usuario/2.png)

### 5.4) Instalação de Pacotes

Instalar os pacotes abaixo para exportar as configurações do usuário.

#### 5.4.1) FreeRadios

Instalar FreeRadius

![alt text](img/ec2/pfsense/openvpn/packages/5.png)

![alt text](img/ec2/pfsense/openvpn/packages/6.png)

![alt text](img/ec2/pfsense/openvpn/packages/7.png)

#### 5.4.2) OpenVPN Exporter

Instalar OpenVPN Exporter

![alt text](img/ec2/pfsense/openvpn/packages/1.png)

![alt text](img/ec2/pfsense/openvpn/packages/2.png)

![alt text](img/ec2/pfsense/openvpn/packages/3.png)

![alt text](img/ec2/pfsense/openvpn/packages/4.png)

### 5.5) FreeRadios Configurações

#### 5.5.1) Interfaces

Qual interface o serviço FreeRadiu vai ficar listen?

![alt text](img/ec2/pfsense/openvpn/freeradius/1.png)

![alt text](img/ec2/pfsense/openvpn/freeradius/2.png)

![alt text](img/ec2/pfsense/openvpn/freeradius/3.png)

#### 5.5.2) Nas Clients

Aqui vamos definir qual usuário irá conectar-se no FreeRadius para validar a autenticação do usuário que está requisitando o login.

![alt text](img/ec2/pfsense/openvpn/nas-client/3.png)

![alt text](img/ec2/pfsense/openvpn/nas-client/4.png)

![alt text](img/ec2/pfsense/openvpn/nas-client/5.png)

#### 5.5.3) Usuário FreeRadios    

![alt text](img/ec2/pfsense/openvpn/user-freeradius/6.png)

![alt text](img/ec2/pfsense/openvpn/user-freeradius/7.png)

![alt text](img/ec2/pfsense/openvpn/user-freeradius/8.png)

![alt text](img/ec2/pfsense/openvpn/user-freeradius/9.png)

### 5.6) OpenVPN Wizard

![alt text](img/ec2/pfsense/openvpn/openvpn-wizard/1.png)

![alt text](img/ec2/pfsense/openvpn/openvpn-wizard/2.png)

![alt text](img/ec2/pfsense/openvpn/openvpn-wizard/3.png)

![alt text](img/ec2/pfsense/openvpn/openvpn-wizard/4.png)

![alt text](img/ec2/pfsense/openvpn/openvpn-wizard/5.png)

![alt text](img/ec2/pfsense/openvpn/openvpn-wizard/6.png)

Aqui você precisa definir qual é a faixa de Rede da VPN.

![alt text](img/ec2/pfsense/openvpn/openvpn-wizard/7.png)

![alt text](img/ec2/pfsense/openvpn/openvpn-wizard/8.png)

![alt text](img/ec2/pfsense/openvpn/openvpn-wizard/9.png)

![alt text](img/ec2/pfsense/openvpn/openvpn-wizard/10.png)

Depois de Configurado o Wizard, vamos ajustar alguns detalhes.

![alt text](img/ec2/pfsense/openvpn/openvpn-wizard/11.png)

Mude o autenticador para usar a base do FreeRadius.

![alt text](img/ec2/pfsense/openvpn/openvpn-wizard/12.png)

![alt text](img/ec2/pfsense/openvpn/openvpn-wizard/13.png)

Aqui é um detalhe insteressante, pode-se definir rotas dinamicamentes e até mesmo definir os logs do serviço.

![alt text](img/ec2/pfsense/openvpn/openvpn-wizard/14.png)

![alt text](img/ec2/pfsense/openvpn/openvpn-wizard/15.png)

![alt text](img/ec2/pfsense/openvpn/openvpn-wizard/16.png)

![alt text](img/ec2/pfsense/openvpn/openvpn-wizard/17.png)

![alt text](img/ec2/pfsense/openvpn/openvpn-wizard/18.png)

![alt text](img/ec2/pfsense/openvpn/openvpn-wizard/19.png)

![alt text](img/ec2/pfsense/openvpn/openvpn-wizard/20.png)

![alt text](img/ec2/pfsense/openvpn/openvpn-wizard/21.png)

![alt text](img/ec2/pfsense/openvpn/openvpn-wizard/22.png)

![alt text](img/ec2/pfsense/openvpn/openvpn-wizard/23.png)

### 5.7) Adicionando Usuário de VPN
### 5.8) Exportando Configuração do Cliente
### 5.9) VPN Connect