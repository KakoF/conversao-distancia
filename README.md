# conversao-distancia

Projeto de exemplo em Python para deploy no Kubernetes


# Docker

## Comandos:

Build da Imagem e Execução do Container Local:
```cmd
docker build -t conversao-distancia -f .\Dockerfile .

docker container run -d -p:8181:5000 conversao-distancia

docker inspect -f "{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}" {id-container}  

docker container rm - f {id-container}           
```

DockerHub:
* Setar tag da imagem base que vamos usar no Dockerfile

* Setar nome para o padrão de nomenclatura(namespace/repositorio:tag) do DockerHub
```cmd
docker build -t kakoferrare/conversao-distancia:v1 .

#Limpar images residuais
docker image prune

# Login no DockerHub
docker login

docker push kakoferrare/conversao-distancia:v1  

#Boa pratica é sempre bom setar uma versao lastest, assim vamos "taguear" v1 como latest
docker tag kakoferrare/conversao-distancia:v1  kakoferrare/conversao-distancia:latest

#Verificar localmente se criou uma nova imagem com a tag lastest, tambem notar que a ImageId da v1 e latest é a mesma
docker image ls

#Enviar a latest pro DockerHub
docker push kakoferrare/conversao-distancia:latest
```


# Kubernetes

Refrencia para solução de erro ao recuperar nodes:
https://stackoverflow.com/questions/78663883/kubectl-error-memcache-go265-couldnt-get-current-server-api-group-list
C:\Users\{user}\.kube

### Dependencias instaladas:

* Kubectl
* K3D

### Pods
Menor objeto do cluster kubernetes, nele que é executado os containers
### Replicaset
Ele quem vai cuidar da escalabilidade e resiliencia dos pods, é o controlador do pod... Verifica quantos podes seram executados e garantir que tenha essa quantidade em execução
### Deployment
Fica acima do **replicaset** e faz o gerenciamento dele, para fazer a troca de versão da aplicação
### Service
Quem expõe o pod/aplicação


## K3D
Comandos:

```cmd
#Criar um cluster kubernetes simples, com um só nó(nodes) rodando
k3d cluster create

k3d cluster delete

k3d cluster list

#Criar um cluster kubernetes com 3 control planes, e 3 worker nodes(nós)
k3d cluster create meucluster --servers 3 --agents 3
```

## Kubectl
Comandos:

```cmd
#Apos criarmos o nó local, podemos ver ele com o comando
kubectl get nodes

kubectl config get-contexts
```


# Deployment

Vamos criar um arquivo deployment.yaml, para especificar os pods

## Especificação

No kubernetes a forma correta de declarar o que precisamos, é pela forma declarativa (declaramos em um arquivo as especificação do que queremos do cluster) em um arquivo de manifesto

No arquivo de manifesto, tem 4 campos importantes que vai ser necessário em qualquer manifesto que vou criar:

* apiVersion -> versão da api do kubernetes que estou utilizando
Comando para ver a versão, como vamos usar um deployment, atentar com o version do mesmo
```cmd
kubectl api-resources
```

* kind -> Tipo de deploy
Comando para ver a versão, como vamos usar um deployment, atentar com kind do mesmo
```cmd
kubectl api-resources
```

* metadata -> Metadados do objeto, nome, labels... 

* spec -> Especificação do Deployment, definir tudo o que é preciso para rodar a aplicação

### Aplicar o manifesto no Kubernetes
```cmd
#Usando apenas o create, os objetos não podem existir ainda, porque se existir ele vai dar conflito e não vai criar
kubectl create -f k8s/deployment.yaml

#Usando apenas o aplpy, mesmo que existam, ele vai aplicar. Se não existir ele criar os objetos, se já existir ele fará as alterações necessárias
kubectl apply -f k8s/deployment.yaml
```

### Conferir o objeto criado

Verificar os pods
```cmd
kubectl get pods

#READY 1/1 -> Quantos containers eu tenho declarado/Quantos estão em execução
```

Informações detalhadas do pod
```cmd
kubectl describe pod conversao-distancia-79d5c9798-nhtzl
```

Informações sobre ReplicaSet, informações do controlador do pod
```cmd
kubectl get replicaset
# DESIRED -> Quantas replicas queremos
# CURRENT -> Quantas te, realmente
# READY -> Quantas estao executando
```

Deletar um pod
```cmd
#Replicaset controla o numero de pods ativos, se apagar, ele sobe outro
kubectl delete pod conversao-distancia-79d5c9798-bdmr2
```

Verificar o deployment
```cmd
kubectl get deployment
```


Verificar recursos usados pelo pod
```cmd
kubectl top pod
```


### Alterações nas replicar
Vamos alterar o tamanho de replicas que queremos no arquivo manifesto
```cmd
#Aplicamos a alteração
kubectl apply -f k8s/deployment.yaml
```

```cmd
#Visao aprofundada do pod, ip, etc...
kubectl get pod -o wide
```

### Acessar a aplicação pelo POD
Pra acessar localmente a aplicação rodando no pod, precisamos fazer portfoward
```cmd
kubectl port-forward pod/conversao-distancia-79d5c9798-nsbfj 8080:5000
```


# Service

Especificação do serviço que vai expor os pods. Tipos de services (mais utilizados)
Vamos criar um arquivo service.yaml, para expor os pods

### Podemos criar 2 arquivos
Um para deployment e outro para service e aplicar executando os 2 comandos
```cmd
kubectl apply -f k8s/deployment.yaml 
kubectl apply -f k8s/service.yaml
```


* Cluster IP -> Serviço que só é acessivel internamente no cluster. Mais utilizado quando queremos comunicar os pods entre si, dentro do mesmo cluster

* Node Port -> Serviço que expõe o pod para o mundo externo. Vai ser acessivel por qualquer Maquina(IP) que faça parte do seu cluster. utilizados em cenários OnPremise

* Load Balancer -> Mais comum em cenários cloud. Cria um load balance e serve um IP publico

### Conferir o objeto criado

Listar todos objetos
```cmd
kubectl get all

```


## Recriando cluster para expor service

### Recriar o cluster

```cmd
k3d cluster delete meucluster

#Recriamos o cluster fazendo um bind de porta com o -p
k3d cluster create meucluster --servers 1 --agents 2 -p "8080:30000@loadbalancer"
```



# Nova versão da aplicação V2
Alteramos o projeto, e vamos subir uma nova versão

```cmd
docker build -t kakoferrare/conversao-distancia:v2 --push .
```
Aumentamos o numero de replicar e mudamos a versão da imagem para pegar a v2.
Ele foi matando os pods e subindo os novos e mantendo um historico de replicaset com as versões

Podemos mudar, fazer um rollback de uma versão usando
```cmd
Ver historico
kubectl rollout history deployment conversao-distancia

kubectl rollout undo deployment conversao-distancia
```
