# Criando Minha VPC

- [1) O que é VPC?](#1-o-que-é-vpc)
  - [1.1) Como é criado uma VPC?](#11-como-é-criado-uma-vpc)
- [2) Qual cenário vamos abordar?](#2-qual-cenário-vamos-abordar)  
- [3) O que preciso saber antes?](#3-o-que-preciso-saber-antes)
  - [3.1) Calculando quantidade de endereços IPs](#31-calculando-quantidade-de-endereços-ips)
- [4) Primeira VPC](#4-primeira-vpc)
  - [4.1) Criando VPC](#41-criando-vpc)
  - [4.2) Criando SubNet](#42-criando-subnet)
  - [4.3) Criando Intenet Gateway](#43-criando-intenet-gateway) 
  - [4.4) Route Tables Liberando Internet](#44-route-tables-liberando-internet)   
- [5) EC2 Pertencentes à Primeira VPC](#5-ec2-pertencentes-à-primeira-vpc)
  - [5.1) EC2 VPC PauloEstudos](#51-ec2-vpc-pauloEstudos)
  - [5.2) EC2 Comunicação Entre SubRedes](#52-ec2-comunicação-entre-subredes)
- [6) Segunda VPC](#6-segunda-vpc)
- [7) PeeringConnect](#7-peeringconnect)
- [8) Comunicando as 2 VPC](#8-comunicando-as-2-vpc)  
## 1) O que é VPC?

Virtual é nuvem privada virtual, configurável de recursos compartilhados de computação alocados dentro de um ambiente de nuvem pública, fornecendo um certo nível de isolamento entre as diferentes organizações, utilizando os recursos da AWS

### 1.1) Como é criado uma VPC?

A Própria *AWS* pode criar automaticamente um VPC no formato *default*, mas na grande maioria dos cenários encontrados há uma necessidade de fazer esse processo manualmente. E é isso que essa documentação se compromete a passar.

## 2) Qual cenário vamos abordar?

Teremos 2 VPC, sendo que uma terá multiplas sub-redes e essas VPC se comunicarão.

![alt text](img/1-diagrama.png "Cenario")

## 3) O que preciso saber antes?

### 3.1) Calculando quantidade de endereços IPs

Nessa etapa deve-se atentar em alguns detalhes, principalmete no que diz respeito a quantidade de **Host** que essa VPC irá suportar. O que preciso saber?

Para montarmos uma Rede IPV4 precisamos usar alguns endereços contidos nas classes A, B ou C que respectivamente apontam para:

```yaml
Classe A    10.0.0.0   – 10.255.255.255
Classe B    172.16.0.0 – 172.31.255.255
Classe C    192.168.0.0 – 192.168.255.255
```
Nesse cenário escolhemos que a *VPC* terá uma rede no seguinte formato: **10.1.0.0/16**
Isso nos permitirá criarmos até **65.536 hosts**

#### Entendo como é feito os calculos

```yaml
Rede: 10.1.0.0/255.255.0.0
```

A mascara é formada por octetos, ou seja, uma mascara de rede contém até 32 bits, dividos em 4 blocos.

```yaml
    1º octeto   |   2º octeto     |  3º octeto      |    4º octeto
-----------------------------------------------------------------------
_._._._._._._._ | _._._._._._._._ | _._._._._._._._ | _._._._._._._._ 
``` 

Cada bit é representado por **0** ou **1**, sendo assim uma mascara no formato: *255.255.0.0* tem o seguinte escrita binária.

```yaml
1.1.1.1.1.1.1.1 | 1.1.1.1.1.1.1.1 | 0.0.0.0.0.0.0 | 0.0.0.0.0.0.0 
```

Sendo que o *BITS REPRESENTADOS POR* **0**, são definidos como *HOSTS*.

#### Como calcular a quantidade de Hosts?

A Conta é bem simples: **2 ( elevado a quantidade de NUMEROS 0 ) - 2**

Porque - 2? Porque um dos endereço é destinado **ENDEREÇO DE REDE** e outro **GATEWAY DA REDE**. Sendo assim temos o seguinte resultado.

2<sup>16</sup> -2 = **65.536 hosts**

## 4) Primeira VPC

Nesse cenário estou presumindo que não temos nenhuma VPC, Sub-Net, Intenet Gateway, pois iremos criar isso manualmente. Obsever que não possuimos nada criado.

## 4.1) Criando VPC

![alt text](img/1-vpc.png)

Click em *Create VPC*, conforme mostra a imagem abaixo 

![alt text](img/2-vpc.png)

Defina o enderaçamento IP que será utililzado.

![alt text](img/3-vpc.png)

#### Definindo que as EC2 serão resolvidas pelo DNS da AWS (ec2.intenal)

![alt text](img/4-vpc.png)
![alt text](img/5-vpc.png)

#### Resumo do que foi instalado até o momento

![alt text](img/6-vpc.png)

## 4.2) Criando SubNet

Clique em create subnet conforme mostra na imagem abaixo.

![alt text](img/1-subnet.png)

Selecione a VPC

![alt text](img/2-subnet.png)

Definia um **NOME**, e um **BLOCO DE REDE** que atenderá essa sub-rede, lembrando que essa sub-rede deve possuir uma quantidade de hosts menor para caber dentro da VPC. Observer que estou criando as sub-redes com a sub-rede no formato (**255.255.255.0**), isso garantirá que tenhamos 254 Hosts nessa sub-rede.

![alt text](img/3-subnet.png)

Foram criadas 3 Sub-Redes sendo:

![alt text](img/4-subnet.png)

Após criar as 3 Sub-Redes devemos definir uma configuração em cada uma das sub-redes para garantir que quando um **EC2** for criada já seja vinculado um **IP Público** para as instância **EC2**.

Repita os passos abaixos para cada uma das sub-redes.

![alt text](img/5-subnet.png)
![alt text](img/6-subnet.png)

## 4.3) Criando Intenet Gateway

Nesse momento estarei apenas criando o Gateway de Internet, isso *NÃO QUER DIZER* que minhas instâncias EC2 já terão internet nesse momento, isso ainda precisa ser explicitado em outra configuração.

![alt text](img/1-internet-gateway.png)
![alt text](img/2-internet-gateway.png)

Agora precisamos vincular qual VPC esse **INTENET GATEWAY** irá atender.

![alt text](img/3-internet-gateway.png)
![alt text](img/4-internet-gateway.png)

## 4.4) Route Tables Liberando Internet

Aqui que fato vamos ligar qual Sub-Rede pode sair para Internet. Claro que não é somente Internet, mas qualquer tipo de Rota devo tratar nesse menu.

Selecione a route table criada, clique na *ABA* routes e depois em **Edit Routes**

![alt text](img/1-route-tables.png)

Add uma rota cujo o destino seja qualquer coisa, e o gateway (**TARGET**) seja o *intenet gateway* criado no passo anterior

![alt text](img/2-route-tables.png)

Selecione a *ABA* **SubNet Associations** para definir qual Sub-Rede poderá acessar essas rotas definidas.

![alt text](img/3-route-tables.png)
![alt text](img/4-route-tables.png)

## 5) EC2 Pertencentes à Primeira VPC

Vamos criar 3 *EC2* sendo que cada *EC2* será criada em uma sub-rede diferente, ao final, teremos o seguinte resultado.

![alt text](img/1-ec2-vpc1.png) 

### 5.1) EC2 VPC PauloEstudos

Observer que cada *EC2* tem seu respectivo IP Address definido na Sub-Rede.

![alt text](img/2-ec2-vpc1.png)
![alt text](img/3-ec2-vpc1.png)
![alt text](img/4-ec2-vpc1.png)

### 5.2) EC2 Comunicação Entre SubRedes

Para facilitar vou escrever no arquivos **HOSTS** os **IPS** e o nome que quero dá para essa maquinas.

![alt text](img/1-ec2-comunicacao-subrede.png)

Agora vamos tentar **PINGAR** os hosts

![alt text](img/2-ec2-comunicacao-subrede.png)

#### UAI !!! Não estou na mesma rede ( VPC ), não deveriam comunicar-se?

Vamos liberar no firewall para que essas sub-redes possam se comunicar.

**OBS.:  Esses Passos devem ser feitos em Todos os Security Grupo Criados na Hora que criou-se a Instância EC2**

![alt text](img/3-ec2-comunicacao-subrede.png)
![alt text](img/4-ec2-comunicacao-subrede.png)

#### Testando a Comunicação entre as Sub-Redes após tratar no Firewall.

![alt text](img/5-ec2-comunicacao-subrede.png)

## 6) Segunda VPC
## 7) PeeringConnect
## 8) Comunicando as 2 VPC