# Empacotando Manifestos com Chart Helm

- [1) Versão do Helm](#1-versão-do-helm)
- [2) Helm Chart](#2-helm-chart)
   - [2.1) Criando um Chart](#21-criando-um-chart)

## 1) Versão do Helm

    Nessa documentação será adotada a versão *3* do helm. Antes na versão *2* era necessário que o cluster kubernetes executasse um processo chamado *TILLER*, o qual o helm se comunicava com esse serviço chamado *TILLER*, além de ter toda uma configuração necessária de RBAC para isso funcionar bem. Na versão *3* o helm faz uso das credencias do usuário, ou seja, ele usa a chave do usuário *KUBECONFIG* para interagir diretamente com o kube-api. 

    Será preciso ter o *HELM CLI* instalado em sua máquina. Para isso visite o site [[https://helm.sh/docs/intro/quickstart/]]


## 2) Helm Chart
#### 2.1) Criando um Chart

    Estou criando um chart chamado *cursos-chart*

```bash
[paulo@kops-server ~]$ helm create cursos-chart
```


h2. 2.1) Estrutura de um Chart

Após criar alguns arquivos podem ser excluídos para manter dentro do chart o necessário para rodar os projetos.

<pre>
├── Chart.yaml
├── templates
│   ├── NOTES.txt
│   ├── _helpers.tpl
│   ├── cursos-cluster-ip-service.yaml
│   ├── cursos-deployment.yaml
│   └── cursos-ingress.yaml
└── values.yaml
</pre>


h2. 2.1) Arquivos de Manifestos.

Esse projeto cursos, é um exemplo fictício para exemplificar como deployar uma app com chart.

h2. 2.1.1) Arquivo de Manifesto - cursos-deployment.yaml

<pre>
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
</pre>


h2. 2.1.2) Arquivo de Manifesto - cursos-cluster-ip-service.yaml

<pre>
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
</pre>


h2. 2.1.3) Arquivo de Manifesto - cursos-ingress.yaml

<pre>
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
</pre>


h2. 3) Definindo Variáveis 

As variáveis definidas nos manifestos , tem a seguinte syntaxe *{{ .Values.domain.cursos }}*, essa variáveis precisam ser em algum arquivo, ou passado como argumento. 

Para que isso funcione é necessário editar o arquivo *values.yaml*

h2. 3.1) Arquivo de Manifesto - values.yaml

<pre>
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
</pre> 

h2. 4) Fazendo Deploy

No momento do deploy deve-se passar algumas variáveis de ambiente para construir os manifestos.


h2. 4.1) Validando os Manifestos antes de aplica-los


<pre>
helm upgrade helm-cursos helm-cursos \
--namespace staging \
--set image.repository="registry.meudominio.com.br/sistemas/cursos/sistema-cursos" \
--set image.tag="1.0.0" \
--set railsEnv="staging" \
--set domain.cursos="cursos.meudominio.com.br" --dry-run --debug
</pre>


helm-cursos  => Um ( Nome | Apelido ) que identifica o Meu Helm 
helm-cursos  => Nome do diretório que contém os manifestos
--set        => Usado para definir variáveis dinâmicamente.


h2. 4.2) Aplicando os Manifestos


<pre>
helm upgrade helm-cursos helm-cursos \
--namespace staging \
--set image.repository="registry.meudominio.com.br/sistemas/cursos/sistema-cursos" \
--set image.tag="1.0.0" \
--set railsEnv="staging" \
--set domain.cursos="cursos.meudominio.com.br"
</pre> 

h2. 5) Inspecionando os Deploys

<pre>
helm list -n staging

NAME        NAMESPACE REVISION  UPDATED                                 STATUS    CHART             APP VERSION
helm-cursos staging   3         2021-02-12 00:44:44.058644871 +0000 UTC deployed  helm-cursos-0.1.0 1.16.0

</pre>

<pre>
helm history -n staging helm-cursos

REVISION  UPDATED                   STATUS      CHART             APP VERSION DESCRIPTION
1         Thu Feb 11 21:10:54 2021  superseded  helm-cursos-0.1.0 1.16.0      Install complete
2         Fri Feb 12 00:33:55 2021  superseded  helm-cursos-0.1.0 1.16.0      Upgrade complete
3         Fri Feb 12 00:44:44 2021  deployed    helm-cursos-0.1.0 1.16.0      Upgrade complete
</pre>

h2. 6) Fazendo Downgrade

<pre>
helm rollback -n staging helm-cursos 1
</pre>

h2. 7) Hospendando meus Charts

Vc pode hospedar seu Charts em um repositório privado, para isso vamos usar um projeto OpenSource chamado *ChartMuseum*

[[https://chartmuseum.com/#Instructions]]


h2. 7.1) Criando um repositório local

<pre>
docker run --rm -it \
  -p 8080:8080 \
  -v $(pwd)/charts:/charts \
  -e DEBUG=true \
  -e STORAGE=local \
  -e STORAGE_LOCAL_ROOTDIR=/charts \
  chartmuseum/chartmuseum:latest
</pre>

h2. 7.2) Checando meus repositórios

<pre>
helm repo list
helm repo add meurepo http://localhost:8080
helm repo del meurepo
helm repo add meurepo http://localhost:8080
helm push helm-cursos meurepo
</pre>

h2. 7.3) Baixando um repositório remoto

<pre>
helm pull meurepo/helm-cursos
</pre>

h2. 7.4) Deletando um deployment

<pre>
helm delete -n staging helm-cursos
</pre>

h2. 8) Comando Uteis ( Helm )

<pre>
helm repo add
helm search repo 
helm install
helm upgrade
helm rollback
helm create
helm delete
helm status
</pre>


