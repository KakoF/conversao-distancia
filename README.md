# conversao-distancia




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