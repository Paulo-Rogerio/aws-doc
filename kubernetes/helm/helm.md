# Empacotando Manifestos com Chart Helm

- [1) Versão do Helm](#1-versão-do-helm)
- [2) Helm Chart](#2-helm-chart)
   - [2.1) Criando um Chart](#21-criando-um-chart)
   - [2.2) Estrutura de um Chart](#22-estrutura-de-um-chart) 
   - [2.3) Arquivos de Manifestos](#23-arquivos-de-manifestos)
        - [2.3.1) Cursos Deployment](#231-cursos-deployment)
        - [2.3.2) Cursos Services ClusterIP](#232-cursos-services-clusterIP)
        - [2.3.3) Cursos Ingress](#233-cursos-ingress)
   - [2.4) Definindo Variáveis](#24-definindo-variáveis)
        - [2.4.1) Arquivo de Manifesto Values](#241-arquivo-de-manifesto-values)        
- [3) Deploy de um Chart](#3-deploy-de-um-chart)
  - [3.1) Validando os Manifestos](#31-validando-os-manifestos)
  - [3.2) Aplicando os Manifestos](#32-aplicando-os-manifestos)
- [4) Pós Deploy](#4-pós-deploy)  
  - [4.1) Inspecionando os Deploys](#41-inspecionando-os-deploys)
  - [4.2) Histórico dos Deploys](#42-histórico-dos-deploys)
- [5) Manipulando meus Charts após Deploy](#5-manipulando-meus-charts-após-deploy)
  - [5.1) Downgrade de Versões Deployadas](#51-downgrade-de-versões-deployadas)
  - [5.2) Deletando um deployment](#52-deletando-um-deployment)
- [6) Hospendando meus Charts](#6-hospedando-meus-charts)
- [6.1) Criando um repositório local](#61-criando-um-repositório-local)
- [6.2) Checando meus repositórios](62-checando-meus-repositórios)
  - [6.2.1) Listando meus repositórios](#621-listando-meus-repositórios)
  - [6.2.2) Adicionando meu repositório](#622-adicionando-meu-repositório)
  - [6.2.3) Deletando um repositório](#623-deletando-um-repositório)
  - [6.3.4) Enviando Chart para meu repositório](#624-enviando-chart-para-meu-repositório)
- [7) Download de um repositório remoto](#7-download-de-um-repositório-remoto)
- [8) Comandos Úteis](#8-comandos-úteis)

## 1) Versão do Helm

Nessa documentação será adotada a versão *3* do helm. Antes na versão *2* era necessário que o cluster kubernetes executasse um processo chamado *TILLER*, o qual o helm se comunicava com esse serviço chamado *TILLER*, além de ter toda uma configuração necessária de RBAC para isso funcionar bem. Na versão *3* o helm faz uso das credencias do usuário, ou seja, ele usa a chave do usuário *KUBECONFIG* para interagir diretamente com o kube-api. 

Será preciso ter o *HELM CLI* instalado em sua máquina. Para isso visite o site oficial [Setup Helm](https://helm.sh/docs/intro/quickstart)


## 2) Helm Chart
#### 2.1) Criando um Chart

Estou criando um chart chamado *cursos-chart*.

```bash
[root@kops-server ~]# helm create cursos-chart
```

#### 2.2) Estrutura de um Chart

Após criado o chart, alguns arquivos podem ser excluídos para manter dentro do chart apenas os arquivos necessário para rodar os projetos.

```yaml
├── Chart.yaml
├── templates
│   ├── NOTES.txt
│   ├── _helpers.tpl
│   ├── cursos-cluster-ip-service.yaml
│   ├── cursos-deployment.yaml
│   └── cursos-ingress.yaml
└── values.yaml
``` 

#### 2.3) Arquivos de Manifestos

Esse projeto cursos, é um exemplo fictício para exemplificar como deployar uma app com chart.

#### 2.3.1) Cursos Deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: cursos-deployment
  namespace: {{ .Values.railsEnv }}
spec:
  replicas: {{ .Values.minReplicas }}
  selector:
    matchLabels:
      app: cursos
  template:
    metadata:
      annotations:
        reloader.stakater.com/auto: 'true'
      labels:
        app: cursos
        fluentd-tag: cursos
      {{- if .Values.metrics }}
      annotations:
        prometheus.io/screp: 'true'
        prometheus.io/port: '80'
      {{- end }}      
    spec:
      imagePullSecrets:
        - name: usuario_que_conecta_no_registry
      containers:
      - name: nginx
        image: "{{ .Values.image.repository }}/nginx:{{ .Values.image.tag }}"
        imagePullPolicy: Always
        securityContext:
          runAsUser: 1001
          runAsGroup: 1001
        resources:
          requests:
            memory: "{{ .Values.resources.nginx.request.ram }}"
            cpu: "{{ .Values.resources.nginx.request.cpu }}"
          limits:
            memory: "{{ .Values.resources.nginx.limit.ram }}"
            cpu: "{{ .Values.resources.nginx.limit.cpu }}"
        ports:
          - name: http
            containerPort: 8888
        livenessProbe:
          httpGet:
            path: /_health
            port: http
          initialDelaySeconds: 5
          timeoutSeconds: 5
          failureThreshold: 3
          successThreshold: 1
        readinessProbe:
          httpGet:
            path: /_health
            port: http
          initialDelaySeconds: 5
          timeoutSeconds: 5
          failureThreshold: 3
          successThreshold: 3
        env:
          - name: RAILS_ENV
            value: "{{ .Values.railsEnv }}"      
      - name: rails
        image: "{{ .Values.image.repository }}/rails:{{ .Values.image.tag }}"
        imagePullPolicy: IfNotPresent
        securityContext:
          runAsUser: 8888
          runAsGroup: 8888
        resources:
          requests:
            memory: "{{ .Values.resources.rails.request.ram }}"
            cpu: "{{ .Values.resources.rails.request.cpu }}"
          limits:
            memory: "{{ .Values.resources.rails.limit.ram }}"
            cpu: "{{ .Values.resources.rails.limit.cpu }}"
        ports:
          - name: probe
            containerPort: 33333
        livenessProbe:
          httpGet:
            path: /_liveness
            port: probe
          initialDelaySeconds: 5
          timeoutSeconds: 5
          failureThreshold: 3
          successThreshold: 1
        readinessProbe:
          httpGet:
            path: /_readiness
            port: probe
          initialDelaySeconds: 5
          timeoutSeconds: 5
          failureThreshold: 3
          successThreshold: 1
        env:
          - name: RAILS_ENV
            value: "{{ .Values.railsEnv }}"
          - name: RAILS_LOG_TO_STDOUT
            value: 'true'
          - name: RAILS_ENABLE_LOGRAGE
            value: 'true'
          - name: PUMA_MAX_THREADS
            value: "{{ .Values.resources.puma.maxThreads }}"
          - name: PUMA_MIN_THREADS
            value: "{{ .Values.resources.puma.minThreads }}"
            value: 'true'
          - name: VAULT_HOST
            valueFrom:
              secretKeyRef:
                name: vault-secret
                key: VAULT_HOST
          - name: VAULT_TOKEN
            valueFrom:
              secretKeyRef:
                name: vault-secret
                key: VAULT_TOKEN
```


#### 2.3.2) Cursos Services ClusterIP

```yaml
apiVersion: v1
kind: Service
metadata:
  name: cursos-cluster-ip-service
  namespace: {{ .Values.railsEnv }}
spec:
  type: ClusterIP
  selector:
    app: cursos
  ports:
    - port: 80
      targetPort: http
```

#### 2.3.3) Cursos Ingress

```yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: cursos-ingress
  namespace: {{ .Values.railsEnv }}
spec:
  rules:  
  - host: {{ .Values.domain.cursos }}
    http:
      paths:
        - path: / 
          backend:
            serviceName: cursos-cluster-ip-service
            servicePort: 80
```


#### 2.4) Definindo Variáveis

As variáveis definidas nos manifestos tem a seguinte syntaxe *{{ .Values.domain.cursos }}*, essa variáveis precisam ser definidos em algum arquivo, ou passados como argumentos. 

#### 2.4.1) Arquivo de Manifesto Values

Para que isso funcione é necessário editar o arquivo *values.yaml*. E nesse aquivo que definimos as variáveis.

```yaml
metrics: true
domain:
  cursos: cursos.meudominio.com.br
railsEnv:
minReplicas: 1
maxReplicas: 3
image:
  repository:
  tag:
resources:
  nginx:
    request:
      ram: 32Mi
      cpu: 50m
    limit:
      ram: 64Mi
      cpu: 200m
  rails:
    request:
      ram: 250Mi
      cpu: 100m
    limit:
      ram: 600Mi
      cpu: 800m
```

#### 2.4.1) Arquivo de Manifesto Values

## 3) Deploy de um Chart

No momento do deploy deve-se passar algumas variáveis de ambiente para construir os manifestos.

#### 3.1) Validando os Manifestos

Parametros Utilizados:

```yaml
helm-cursos  => Um  Nome que identifica o meu Chart Helm 
helm-cursos  => Nome do diretório que contém os manifestos
--set        => Usado para definir variáveis dinâmicamente.
```

```bash
[root@kops-server ~]# helm upgrade helm-cursos helm-cursos \
--namespace staging \
--set image.repository="registry.meudominio.com.br/sistemas/cursos/sistema-cursos" \
--set image.tag="1.0.0" \
--set railsEnv="staging" \
--set domain.cursos="cursos.meudominio.com.br" --dry-run --debug
```

#### 3.2) Aplicando os Manifestos

```bash
[root@kops-server ~]# helm upgrade helm-cursos helm-cursos \
--namespace staging \
--set image.repository="registry.meudominio.com.br/sistemas/cursos/sistema-cursos" \
--set image.tag="1.0.0" \
--set railsEnv="staging" \
--set domain.cursos="cursos.meudominio.com.br"
```

## 4) Pós Deploy

#### 4.1) Inspecionando os Deploys

```bash
[root@kops-server ~]# helm list -n staging

NAME        NAMESPACE REVISION  UPDATED                                 STATUS    CHART             APP VERSION
helm-cursos staging   3         2021-02-12 00:44:44.058644871 +0000 UTC deployed  helm-cursos-0.1.0 1.16.0

```

#### 4.2) Histórico dos Deploys

```bash
[root@kops-server ~]# helm history -n staging helm-cursos

REVISION  UPDATED                   STATUS      CHART             APP VERSION DESCRIPTION
1         Thu Feb 11 21:10:54 2021  superseded  helm-cursos-0.1.0 1.16.0      Install complete
2         Fri Feb 12 00:33:55 2021  superseded  helm-cursos-0.1.0 1.16.0      Upgrade complete
3         Fri Feb 12 00:44:44 2021  deployed    helm-cursos-0.1.0 1.16.0      Upgrade complete
```

## 5) Manipulando meus Charts após Deploy

#### 5.1) Downgrade de Versões Deployadas

Caso haja necessidade de fazer um rollback de um deploy.

```bash
[root@kops-server ~]# helm rollback -n staging helm-cursos 1
```

#### 5.2) Deletando um deployment

```
[root@kops-server ~]# helm delete -n staging helm-cursos
```

## 6) Hospendando meus Charts

Podemos hospedar os Charts em um repositório privado, para isso vamos usar um projeto OpenSource chamado *ChartMuseum*  * [ChartMuseum](https://chartmuseum.com/#Instructions)

#### 6.1) Criando um repositório local

```bash
[root@kops-server ~]# docker run --rm -it \
  -p 8080:8080 \
  -v $(pwd)/charts:/charts \
  -e DEBUG=true \
  -e STORAGE=local \
  -e STORAGE_LOCAL_ROOTDIR=/charts \
  chartmuseum/chartmuseum:latest
```

#### 6.2) Checando meus repositórios

#### 6.2.1) Listando meus repositórios

```bash
[root@kops-server ~]# helm repo list
```

#### 6.2.2) Adicionando meu repositório

```bash
[root@kops-server ~]# helm repo add meurepo http://localhost:8080
```

#### 6.2.3) Deletando um repositório

```bash
[root@kops-server ~]# helm repo del meurepo
```

#### 6.2.4) Enviando Chart para meu repositório

```bash
[root@kops-server ~]# helm push helm-cursos meurepo
```

## 7) Download de um repositório remoto

```bash
[root@kops-server ~]# helm pull meurepo/helm-cursos
```

## 8) Comando Uteis

```
helm repo add
helm search repo 
helm install
helm upgrade
helm rollback
helm create
helm delete
helm status
```