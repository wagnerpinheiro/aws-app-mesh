## Pre reqs 

- AWS CLI; 
- eksctl 
- kubectl
- docker
- o iam role atrelado aos nodes (Ec2) precisam ter permissão para alguns serviços da AWS 
    - x-ray
    - App Mesh
    - CloudWatch Logs
    - CloudMap
- criar dominio no route 53;
    - domínio: appmeshworkshop.hosted.local
    - record: crystal -> type A -> Url External AWS Load Balancer
- Images no ECR 
    - nodejs Backend
    - crystal Backend
    - Ruby Frontend
    - nodejsv2 Backend

## AWS Services 

- Amazon EKS *
- AWS App Mesh **
- Aws CloudMap *
- Amazon Route53 *
- Amazon Ec2
- AWS Cloud9
- Amazon ECR
- AWS X-Ray *


## Push de imagens para o ECR

Login do docker com o ECR

```
aws ecr get-login-password --region region | docker login --username AWS --password-stdin aws_account_id.dkr.ecr.region.amazonaws.com
```
Build da imagem 

```
docker build . -t tag-image:version
``` 

Tag da imagem 

```
docker tag name-umage aws_account_id.dkr.ecr.us-west-2.amazonaws.com/my-repository:tag
```

Push images
```
docker push aws_account_id.dkr.ecr.us-west-2.amazonaws.com/my-repository:tag
```

## Criação do cluster EKS (eksctl)
```
eksctl create cluster \
  --name app-mesh-2 \
  --version 1.25 \
  --region us-west-1 \
  --nodegroup-name standard-workers \
  --node-type t2.micro \
  --nodes 3 \
  --nodes-min 1 \
  --nodes-max 4
```


# Cenário atual EKS
Aplicação que será utilizada na demosntração do App mesh será: 

![Aplicacao para estudar o App Mesh](
  https://images-blackbelt.s3.us-west-1.amazonaws.com/appmesh1.png?response-content-disposition=inline&X-Amz-Security-Token=IQoJb3JpZ2luX2VjEJz%2F%2F%2F%2F%2F%2F%2F%2F%2F%2FwEaCXNhLWVhc3QtMSJGMEQCIFa1w866vhUbG%2F%2FdpnAAA4PfrKRpcpll7YdXm8ANYx7IAiBWlHJL9j4u0mIxhFRlBm6EbF%2BICfPDrz5LVeDkkhl6MyrmAgg1EAAaDDEyMzQzODUzMjU3NCIMx2lrtNw%2FL7cK8ZJ0KsMCwhcdGm3dDYXG6hqiZb9NtmmogCHc%2FmKYYxdoXAKTSJCK0SGQdN3ZpWf7blH7PwQFI%2FOdf59QA98BvPAUmjoTChOz%2BRUxVGwirmCXf%2FnxYmFT4CRlXlBR5jSORUvVvo53EkKTp%2FDQi%2BYEgSu8DhCuhelDi0Et8gluCBg2tJ%2B0gCNKmNYoz0GG5wu8rGPQI4FU580a8iEzf7rQJ0J8wxW4H3z5zJ1v80VEKLan1X30iauF6X9YRGb1RbV9NIldm6stdVxa9ztunvGIVecrpcMEK6M5Ia9yCkUtnpiJ6LfgXD3hHhUuugdBzdSxp2Jws6Qzn56dFElJzwQKD25ON5vOSxUM9PCepmSx%2F0aFT11fBu%2Fj%2FIyMJLnRALeR3YdBkiXuvba2gvLayznMY0kLIIX4fQ64fQFbmAaHuh9XpK00O6cfXxww78L%2FpQY6iAIYrIydxsFr2jWq7zVF7grysdRtyp3oNCH8PjLpmkP1flccKG1503uy16Ddr0ewzfe1hWAg%2BDQ8lUvougQ20CtHYMSsnIuFp3MQwLzQg6rOLKAmyMczgUxJOIMcRknfJmbvjEb2gn3udm6lTMYUp%2FUH6OCfvRuLQAwuQtIVZttAaPFv80XO4TcS56gwe3%2BHurGqJsqBvcLFW5NO1otR9pV6nsWVTpvUwIZYcJujjjSn4N4a2hSQlmqpwvwN4V71sogKCn3A1BiAFxGdnMcKEESKEX8oowc5ipdI5vKMBzTrxVR4hvL9oDZNKMtLVziml9HPoj%2FRG5klC6c1BHnELgkvrPIER34cQ%2BQ%3D&X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20230725T202122Z&X-Amz-SignedHeaders=host&X-Amz-Expires=43200&X-Amz-Credential=ASIARZPMCQ7PPD5UT76Q%2F20230725%2Fus-west-1%2Fs3%2Faws4_request&X-Amz-Signature=230a813c5b0037f0a6850a2501cb1436c363bb7e254f7b0ffe917eb7babc09fd)


Antes de iniciarmos as configurações, realizar os seguintes passos para avaliar cada retorno. 

Vamos acessar diretamente um pod para conseguir executar o comando abaixo e analisar o retorno dessa chamada. 

```
kubectl get pods -n lab-appmesh
kubectl exec -it ${POD} -n lab-appmesh -- /bin/bash
curl -v http://nodejs.appmeshworkshop.hosted.local:3000/
```

Vamos utilizar o browser
http://${External_LB}


## AWS App mesh

**Breve descrição sobre Service Mesh**

Um Service Mesh é uma infraestrutura de rede que gerencia a comunicação entre os serviços de um sistema distribuído. Ele fornece uma camada de abstração para a comunicação entre microservices, tornando a comunicação mais confiável, segura e escalável. 

**Mas e o AWS App Mesh?**

O AWS App Mesh facilita monitorar, controlar e depurar a comunicação entre os serviços. **O App Mesh usa o Envoy**, um proxy de malha de serviço de **código aberto**, **que é implantado juntamente com seus contêineres de microsserviços.** O App Mesh é **integrado aos serviços da AWS para monitoramento e rastreamento** e funciona com muitas ferramentas populares de terceiros. O App Mesh pode ser usado com contêineres de microsserviços gerenciados pelo **Amazon ECS, Amazon EKS e AWS Fargate, com Kubernetes executado na AWS e com serviços executados no Amazon EC2.**

- Agilize operações, implemente regras personalizadas de roteamento de tráfego, e configure e padronize o modo como os fluxos de tráfego fluem entre os serviços.
- Capture métricas, logs e rastreamentos de suas aplicações para identificar e isolar rapidamente os problemas e otimizar a aplicação.
- Aprimore a segurança de rede com os controles de autenticação e requisições criptografadas entre serviços, mesmo dentro da rede privada.

## AWS App Mesh - Instalação para K8s

**Incluir nesse ponto aqui quais são os itens que precisam ser instalados no ec2 e no ecs para utilizar o mesh! Vou seguir só com o k8s** 

### App mesh controller para k8s 

O AWS App Mesh Controller para K8s é um controlador para ajudar a gerenciar os recursos do App Mesh para um cluster Kubernetes e injetar sidecars nos pods do Kubernetes. O controlador observa os recursos personalizados em busca de alterações e reflete essas alterações na API do App Mesh. O controlador mantém os recursos customizados (CRDs): meshes, virtualnodes, virtualrouters, virtualservices, virtualgateways e gatewayroutes. Os recursos personalizados são mapeados para objetos da API App Mesh.


```
helm version --short
helm repo add eks https://aws.github.io/eks-charts
helm repo list | grep eks-charts
```

Criação do namespace para instalação do app mesh no k8s
```
kubectl create ns appmesh-system
```
**Ponto importante**: eksctl é uma ferramenta CLI para criar e gerenciar clusters (EKS).

Criando seu OIDC identity provider para o cluster
```
eksctl utils associate-iam-oidc-provider \
  --cluster app-mesh \
  --approve
```

 Criando IAM role para o appmesh-controller service account
```
eksctl create iamserviceaccount \
  --cluster app-mesh \
  --namespace appmesh-system \
  --name appmesh-controller \
  --attach-policy-arn  arn:aws:iam::aws:policy/AWSCloudMapFullAccess,arn:aws:iam::aws:policy/AWSAppMeshFullAccess \
  --override-existing-serviceaccounts \
  --approve
```

Vamos instalar o App Mesh Controller no namespace appmesh-system usando o chart do Helm, especificando a conta de serviço que criamos anteriormente.

```
helm upgrade -i appmesh-controller eks/appmesh-controller \
  --namespace appmesh-system \
  --set region=us-west-1 \
  --set serviceAccount.create=false \
  --set serviceAccount.name=appmesh-controller
```

Para verificar se a instalação foi bem-sucedida, liste os objetos no namespace appmesh-system e certifique-se de que a instância do pod appmesh-controller esteja em um estado "Running" antes de continuar.

```
kubectl -n appmesh-system get all
```

Você também pode ver que as definições de recursos personalizados (CRD) do App Mesh foram instaladas.

```
kubectl get crds | grep appmesh
```

## Vincular o App Mesh instalado com o Cluster que está rodando

Primeiramente, precisamos afiliar a aplicação que está rodando no EKS com o Mesh que criamos anteriormente. Isso é feito adicionando um rótulo do mesh ao namespace do aplicativo.

```
kubectl label namespace appmesh-workshop-ns mesh=app-mesh
```

Neste ponto, também configuraremos o App Mesh Controller para injetar automaticamente os contêineres de proxy secundário do Envoy em nossos pods de aplicativo neste namespace. Isso é feito adicionando uma segunda label ao namespace.

```
kubectl label namespace appmesh-workshop-ns appmesh.k8s.aws/sidecarInjectorWebhook=enabled
```

## Criação do mesh através do cluster k8s

Após a instalaçao do App Mesh controller para kubernetes conseguimos realizar a criação dos recursos de Mesh através de comandos kubernetes que refletirão no serviço da AWS App Mesh.

Vamos agora criar o mesh dentro do cluster com os seguintes comandos. Observe que a especificação Mesh indicado no **namespaceSelector** será **resposável por vincular o Mesh com nosso cluster.**

```
apiVersion: appmesh.k8s.aws/v1beta2
kind: Mesh
metadata:
  name: app-mesh
spec:
  namespaceSelector:
    matchLabels:
      mesh: app-mesh
```

Para configurar nossos serviços para rodar dentro do App Mesh, vamos criar um **VirtualNode** para o deployment e service do nodejs-app (aplicação).

**O que é o Virtual Node no AWS AppMesh?**

Um "nó virtual" (virtual node) no AWS App Mesh é uma abstração que representa um serviço individual dentro do mesh de serviços.
Ele define como o tráfego é roteado para esse serviço específico e quais políticas de tráfego, como balanceamento de carga, regras de roteamento e circuit breakers, são aplicadas a ele.

Então podemos dizer que:

* O nó virtual representa um serviço real (que será o back-end desse nó virtual).
* Podemos especificar como o tráfego é roteado para o serviço atrelado ao nó virtual.
* Os nós virtuais permitem que o App Mesh colete informações possibilitando a observabilidade do tráfego. (Monitoramento e telemetria)
* O App Mesh permite que você aplique políticas de segurança em cada nó virtual para controlar o tráfego e garantir a comunicação segura entre os serviços.

Para conseguir buscar o valor do hostname, basta executar o seguinte comando:

```
kubectl get service nodejs-app-service -n appmesh-workshop-ns -o json | jq -r '.status.loadBalancer.ingress[].hostname'
```

```
apiVersion: appmesh.k8s.aws/v1beta2
kind: VirtualNode
metadata:
  name: nodejs-app
  namespace: appmesh-workshop-ns
spec:
  podSelector:
    matchLabels:
      app: nodejs-app
  listeners:
    - portMapping:
        port: 3000
        protocol: http
  serviceDiscovery:
    dns:
      hostname: internal-a6a3345fa57bc41479bb94783c5a7c48-99146466.us-west-1.elb.amazonaws.com
```

Agora que nosso deployment/service do k8s está vinculado com o nosso node virtual, vamos criar um **virtual service** e um **virtual router**.

**O que é o Service Node no AWS AppMesh?** 
 
 Um serviço virtual é uma abstração de um serviço real que é fornecido por um nó virtual, direta ou indiretamente, por meio de um roteador virtual. 

Virtual Service
```
apiVersion: appmesh.k8s.aws/v1beta2
kind: VirtualService
metadata:
  name: nodejs
  namespace: appmesh-workshop-ns
spec:
  awsName: nodejs.appmeshworkshop.hosted.local
  provider:
    virtualRouter:
      virtualRouterRef:
        name: nodejs-router
```


**O que é o Router Node no AWS AppMesh?**
Os roteadores virtuais manipulam o tráfego de um ou mais serviços virtuais dentro de um mesh.

Virtual Router

```
apiVersion: appmesh.k8s.aws/v1beta2
kind: VirtualRouter
metadata:
  name: nodejs-router
  namespace: appmesh-workshop-ns
spec:
  listeners:
    - portMapping:
        port: 3000
        protocol: http
  routes:
    - name: route-to-nodejs-app
      httpRoute:
        match:
          prefix: /
        action:
          weightedTargets:
            - virtualNodeRef:
                name: nodejs-app
              weight: 1
```

Nós conseguimos configurar os recursos necessários para que os novos pods subam com o container adicional do Envoy, entretanto, será necessário o restart dos pods que o novo container seja atribuido a eles. 

```
kubectl -n appmesh-workshop-ns rollout restart deployment nodejs-app
```

Agora vamos validar quais containers nós temos rodando dentro do nossos pods.

```
kubectl -n appmesh-workshop-ns get pods ${POD} -o jsonpath='{.spec.containers[*].name}';
```

Para auxiliar na análise podemos avaliar os logs de cada container dentro do pod com o seguinte comando, podemos ver se os logs do envoy estão com algum problema.

```
kubectl logs ${POD} -n appmesh-workshop-ns -c envoy
```

Vamos avaliar novamente o retorno de uma chamada curl, agora a resposta não será diretamente da nossa aplicação e sim da camada de proxy (envoy) que adicionamos.

Vamos acessar diretamente um pod para conseguir executar o comando abaixo e analisar o retorno dessa chamada. 

```
kubectl exec -it ${POD} -n appmesh-workshop-ns -- /bin/bash
curl -v http://nodejs.appmeshworkshop.hosted.local:3000/
```

Tudo funcionando? Perfeito, vamos aplicar esses mesmos passos para as outras apliações Crytasl Backend e Frontend.

Vamos analisar como ficou o painel (AWS)!

![App Mesh with Envoy](https://images-blackbelt.s3.us-west-1.amazonaws.com/appmesh2.png?response-content-disposition=inline&X-Amz-Security-Token=IQoJb3JpZ2luX2VjEJz%2F%2F%2F%2F%2F%2F%2F%2F%2F%2FwEaCXNhLWVhc3QtMSJGMEQCIFa1w866vhUbG%2F%2FdpnAAA4PfrKRpcpll7YdXm8ANYx7IAiBWlHJL9j4u0mIxhFRlBm6EbF%2BICfPDrz5LVeDkkhl6MyrmAgg1EAAaDDEyMzQzODUzMjU3NCIMx2lrtNw%2FL7cK8ZJ0KsMCwhcdGm3dDYXG6hqiZb9NtmmogCHc%2FmKYYxdoXAKTSJCK0SGQdN3ZpWf7blH7PwQFI%2FOdf59QA98BvPAUmjoTChOz%2BRUxVGwirmCXf%2FnxYmFT4CRlXlBR5jSORUvVvo53EkKTp%2FDQi%2BYEgSu8DhCuhelDi0Et8gluCBg2tJ%2B0gCNKmNYoz0GG5wu8rGPQI4FU580a8iEzf7rQJ0J8wxW4H3z5zJ1v80VEKLan1X30iauF6X9YRGb1RbV9NIldm6stdVxa9ztunvGIVecrpcMEK6M5Ia9yCkUtnpiJ6LfgXD3hHhUuugdBzdSxp2Jws6Qzn56dFElJzwQKD25ON5vOSxUM9PCepmSx%2F0aFT11fBu%2Fj%2FIyMJLnRALeR3YdBkiXuvba2gvLayznMY0kLIIX4fQ64fQFbmAaHuh9XpK00O6cfXxww78L%2FpQY6iAIYrIydxsFr2jWq7zVF7grysdRtyp3oNCH8PjLpmkP1flccKG1503uy16Ddr0ewzfe1hWAg%2BDQ8lUvougQ20CtHYMSsnIuFp3MQwLzQg6rOLKAmyMczgUxJOIMcRknfJmbvjEb2gn3udm6lTMYUp%2FUH6OCfvRuLQAwuQtIVZttAaPFv80XO4TcS56gwe3%2BHurGqJsqBvcLFW5NO1otR9pV6nsWVTpvUwIZYcJujjjSn4N4a2hSQlmqpwvwN4V71sogKCn3A1BiAFxGdnMcKEESKEX8oowc5ipdI5vKMBzTrxVR4hvL9oDZNKMtLVziml9HPoj%2FRG5klC6c1BHnELgkvrPIER34cQ%2BQ%3D&X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20230726T004331Z&X-Amz-SignedHeaders=host&X-Amz-Expires=43200&X-Amz-Credential=ASIARZPMCQ7PPD5UT76Q%2F20230726%2Fus-west-1%2Fs3%2Faws4_request&X-Amz-Signature=ffdec279bc89a78e0378549cd235dfcae8f6af493fbd22b839aa648aaf43507c)

## Aceleradores com o App Mesh (Recursos AWS)

## Observabilidade 

Agora que temos o App Mesh configurado dentro do nosso cluster k8s, temos que entender quais são os benefícios da utilização desse recurso e em quais cenários ele pode nos auxiliar no dia a dia. 
Um dos grndes tunfos do App Mesh é que ele tem uma comunicação muito boa com o ecossistema da AWS, por exemplo, no nosso próximo passo vamos fazer com que as informações coletadas pelo App Mesh sejam encaminhadas para o X-Ray para aumentar nosso nível de observabilidade dentro do nosso cluster. 

**Primeiro, o que é x-ray?**

O AWS X-Ray ajuda desenvolvedores a analisar e depurar aplicações distribuídas de produção, como as criadas usando uma arquitetura de microsserviços. Com o X-Ray, é possível entender a performance de aplicativos e de seus serviços subjacentes para identificar e solucionar problemas e erros de performance. O X-Ray disponibiliza uma visualização completa sobre as solicitações, conforme elas percorrem o aplicativo, além de mostrar um mapa dos componentes subjacentes do aplicativo. É possível usar o X-Ray para analisar as aplicações em desenvolvimento e em produção, abrangendo de simples aplicações de três camadas a aplicações complexas de microsserviços compostas por milhares de serviços.

A primeira etapa é alterar a configuração do App Mesh Controller para adicionar automaticamente o contêiner X-Ray a novos pods e configurar os contêineres proxy Envoy para enviar dados a eles:

```
helm upgrade -i appmesh-controller eks/appmesh-controller \
  --namespace appmesh-system \
  --set region=us-west-1 \
  --set serviceAccount.create=false \
  --set serviceAccount.name=appmesh-controller \
  --set tracing.enabled=true \
  --set tracing.provider=x-ray
```

Restart a aplicação para que o container x-ray-daemon suba no pod

obs.: Incluir em todas as aplicações nodejs-app, crystal e frontend. 
```
kubectl -n appmesh-workshop-ns rollout restart deployment nodejs-app
```

Validar quais containers estão rodando dentro do POD

```
kubectl -n appmesh-workshop-ns get pods ${POD} -o jsonpath='{.spec.containers[*].name}'
```

Para auxiliar na análise podemos avaliar os logs de cada container dentro do pod com o seguinte comando, podemos ver se os logs do xray-daemon estão com algum problema.

```
 kubectl logs ${POD} -n appmesh-workshop-ns -c xray-daemon
```
Vamos validar no painel como está o trace do X-ray que está recebendo informações do Envoy enquanto as requisições são realizadas.

![App Mesh with X-ray](https://images-blackbelt.s3.us-west-1.amazonaws.com/appmesh3.png?response-content-disposition=inline&X-Amz-Security-Token=IQoJb3JpZ2luX2VjEJz%2F%2F%2F%2F%2F%2F%2F%2F%2F%2FwEaCXNhLWVhc3QtMSJGMEQCIFa1w866vhUbG%2F%2FdpnAAA4PfrKRpcpll7YdXm8ANYx7IAiBWlHJL9j4u0mIxhFRlBm6EbF%2BICfPDrz5LVeDkkhl6MyrmAgg1EAAaDDEyMzQzODUzMjU3NCIMx2lrtNw%2FL7cK8ZJ0KsMCwhcdGm3dDYXG6hqiZb9NtmmogCHc%2FmKYYxdoXAKTSJCK0SGQdN3ZpWf7blH7PwQFI%2FOdf59QA98BvPAUmjoTChOz%2BRUxVGwirmCXf%2FnxYmFT4CRlXlBR5jSORUvVvo53EkKTp%2FDQi%2BYEgSu8DhCuhelDi0Et8gluCBg2tJ%2B0gCNKmNYoz0GG5wu8rGPQI4FU580a8iEzf7rQJ0J8wxW4H3z5zJ1v80VEKLan1X30iauF6X9YRGb1RbV9NIldm6stdVxa9ztunvGIVecrpcMEK6M5Ia9yCkUtnpiJ6LfgXD3hHhUuugdBzdSxp2Jws6Qzn56dFElJzwQKD25ON5vOSxUM9PCepmSx%2F0aFT11fBu%2Fj%2FIyMJLnRALeR3YdBkiXuvba2gvLayznMY0kLIIX4fQ64fQFbmAaHuh9XpK00O6cfXxww78L%2FpQY6iAIYrIydxsFr2jWq7zVF7grysdRtyp3oNCH8PjLpmkP1flccKG1503uy16Ddr0ewzfe1hWAg%2BDQ8lUvougQ20CtHYMSsnIuFp3MQwLzQg6rOLKAmyMczgUxJOIMcRknfJmbvjEb2gn3udm6lTMYUp%2FUH6OCfvRuLQAwuQtIVZttAaPFv80XO4TcS56gwe3%2BHurGqJsqBvcLFW5NO1otR9pV6nsWVTpvUwIZYcJujjjSn4N4a2hSQlmqpwvwN4V71sogKCn3A1BiAFxGdnMcKEESKEX8oowc5ipdI5vKMBzTrxVR4hvL9oDZNKMtLVziml9HPoj%2FRG5klC6c1BHnELgkvrPIER34cQ%2BQ%3D&X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20230726T005050Z&X-Amz-SignedHeaders=host&X-Amz-Expires=43200&X-Amz-Credential=ASIARZPMCQ7PPD5UT76Q%2F20230726%2Fus-west-1%2Fs3%2Faws4_request&X-Amz-Signature=d6bd0add67a7f60d95736212f9e07d3f6ab6d1121e004e0b6ee109f3b96dc45d)


## Service discovery com AWS CloudMap

O AWS Cloud Map é um serviço de descoberta de recursos. O Cloud Map permite que você nomeie seus recursos de aplicativo com nomes personalizados e atualiza automaticamente os locais desses recursos que mudam dinamicamente.

Primeiro passo é criar o namespace no AWS CloudMap.
Nomde do namespace: appmesh.pvt.local
Instance discovery: API calls and DNS queries in VPCs

Para realizar o vinculo com o Aws CloudMap podemos utilizar o seguinte manifesto:

```
apiVersion: appmesh.k8s.aws/v1beta2
kind: VirtualNode
metadata:
  name: nodejs-app
  namespace: appmesh-workshop-ns
spec:
  podSelector:
    matchLabels:
      app: nodejs-app
  listeners:
    - portMapping:
        port: 3000
        protocol: http
  serviceDiscovery:
    awsCloudMap:
     namespaceName: "appmesh.pvt.local"
     serviceName: nodejs
```


Listar os Ipds dos Pods para bater com os IPs do AWS Cloud Map.

```
kubectl get pods -n appmesh-workshop-ns -o wide
```

Vamos avaliar se os mesmos IPs estão sendo listados no cloudMap? Vamos avaliar diretamente pelo painel.

Agora podemos apontar o dominio inicial **appmeshworkshop.hosted.local** para o domínio criado pelo cloudMap **appmesh.pvt.local** e não mais para o load balancer como estava inicialmente.  (Utilziando CNAME)

Agora podemos alterar o deployment feito inicialmente incluindo uma replica e podemos ver que ela foi descoberta automaticamente. 

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nodejs-app
  labels:
    app: nodejs-app
  namespace: appmesh-workshop-ns
spec:
  replicas: 4
  selector:
    matchLabels:
      app: nodejs-app
  strategy:
    rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 25%
    type: RollingUpdate
  template:
    metadata:
      labels:
        app: nodejs-app
    spec:
      containers:
      - image: 123438532574.dkr.ecr.us-west-1.amazonaws.com/app-mesh-lab-partner:latest
        imagePullPolicy: Always
        name: nodejs-app
        ports:
        - containerPort: 3000
          protocol: TCP
```

Além disso podemos excluir o service atrelado ao internal load balancer. Pronto, nossa aplicação e comunicação está funcionando totalmente atrelada ao Mesh sem a necessidade de um service com load balancer.

![AppMesh with AWS cloudMAp](https://images-blackbelt.s3.us-west-1.amazonaws.com/appmesh4.png?response-content-disposition=inline&X-Amz-Security-Token=IQoJb3JpZ2luX2VjEJz%2F%2F%2F%2F%2F%2F%2F%2F%2F%2FwEaCXNhLWVhc3QtMSJGMEQCIFa1w866vhUbG%2F%2FdpnAAA4PfrKRpcpll7YdXm8ANYx7IAiBWlHJL9j4u0mIxhFRlBm6EbF%2BICfPDrz5LVeDkkhl6MyrmAgg1EAAaDDEyMzQzODUzMjU3NCIMx2lrtNw%2FL7cK8ZJ0KsMCwhcdGm3dDYXG6hqiZb9NtmmogCHc%2FmKYYxdoXAKTSJCK0SGQdN3ZpWf7blH7PwQFI%2FOdf59QA98BvPAUmjoTChOz%2BRUxVGwirmCXf%2FnxYmFT4CRlXlBR5jSORUvVvo53EkKTp%2FDQi%2BYEgSu8DhCuhelDi0Et8gluCBg2tJ%2B0gCNKmNYoz0GG5wu8rGPQI4FU580a8iEzf7rQJ0J8wxW4H3z5zJ1v80VEKLan1X30iauF6X9YRGb1RbV9NIldm6stdVxa9ztunvGIVecrpcMEK6M5Ia9yCkUtnpiJ6LfgXD3hHhUuugdBzdSxp2Jws6Qzn56dFElJzwQKD25ON5vOSxUM9PCepmSx%2F0aFT11fBu%2Fj%2FIyMJLnRALeR3YdBkiXuvba2gvLayznMY0kLIIX4fQ64fQFbmAaHuh9XpK00O6cfXxww78L%2FpQY6iAIYrIydxsFr2jWq7zVF7grysdRtyp3oNCH8PjLpmkP1flccKG1503uy16Ddr0ewzfe1hWAg%2BDQ8lUvougQ20CtHYMSsnIuFp3MQwLzQg6rOLKAmyMczgUxJOIMcRknfJmbvjEb2gn3udm6lTMYUp%2FUH6OCfvRuLQAwuQtIVZttAaPFv80XO4TcS56gwe3%2BHurGqJsqBvcLFW5NO1otR9pV6nsWVTpvUwIZYcJujjjSn4N4a2hSQlmqpwvwN4V71sogKCn3A1BiAFxGdnMcKEESKEX8oowc5ipdI5vKMBzTrxVR4hvL9oDZNKMtLVziml9HPoj%2FRG5klC6c1BHnELgkvrPIER34cQ%2BQ%3D&X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20230726T005858Z&X-Amz-SignedHeaders=host&X-Amz-Expires=43200&X-Amz-Credential=ASIARZPMCQ7PPD5UT76Q%2F20230726%2Fus-west-1%2Fs3%2Faws4_request&X-Amz-Signature=14b8db8cc5b8449dc767a175eb70b76d20f8b9026b7ef5740b5b52dc1fdd54c7)

## AWS App Mesh - Virtual Gateway 

Legal! Chegamos até aqui, nossos pods estão sendo mapeados de forma automática graças a integração do AWS CloudMap com o AWS App Mesh e nós não precisamos mais de um Load Balancer como serviço para nos comunicar com nossos pods. 
Isso é excelente, mas você ja se perguntou porque mesmo sem o Load Balancer esse sistema continua funcionando? 
A resposta é simples, todo o ecossistema está dentro do mesmo cluster que está sendo tratado pelo AWS App Mesh, então ele tem controle do tráfego e toda comunicação flui. 
Mas e se outro recurso que não está no AWS App Mesh precisar realizar uma chamada para os Pods? 
Para esse cenário, nós temos o **Virtual Gateway**!

Primeiro passo vamos incluir uma label na nossa namespace do kubernetes, essa label será utilizada pelo Virtual Gateway. 

```
kubectl label namespace appmesh-workshop-ns gateway=ingress-gw
```


Vamos agora criar os serviços necessários para criar a exposição dos nossos serviços para que clientes fora do AWS App Mesh possam nos consultar.


```
---
apiVersion: v1
kind: Service
metadata:
  name: ingress-gw
  namespace: appmesh-workshop-ns
  annotations:
    service.beta.kubernetes.io/aws-load-balancer-type: "nlb"
    service.beta.kubernetes.io/aws-load-balancer-internal: (http://service.beta.kubernetes.io/aws-load-balancer-internal:) "true"
spec:
  type: LoadBalancer
  ports:
    - port: 80
      targetPort: 8088
      name: http
  selector:
    app: ingress-gw
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: ingress-gw
  namespace: appmesh-workshop-ns
spec:
  replicas: 2
  selector:
    matchLabels:
      app: ingress-gw
  template:
    metadata:
      labels:
        app: ingress-gw
    spec:
      containers:
        - name: envoy
          image: 840364872350.dkr.ecr.us-west-1.amazonaws.com/aws-appmesh-envoy:v1.16.1.1-prod
          ports:
            - containerPort: 8088
---
apiVersion: appmesh.k8s.aws/v1beta2
kind: VirtualGateway
metadata:
  name: ingress-gw
  namespace: appmesh-workshop-ns
spec:
  namespaceSelector:
    matchLabels:
      gateway: ingress-gw
  podSelector:
    matchLabels:
      app: ingress-gw
  listeners:
    - portMapping:
        port: 8088
        protocol: http
---
apiVersion: appmesh.k8s.aws/v1beta2
kind: GatewayRoute
metadata:
  name: gateway-route-eks
  namespace: appmesh-workshop-ns
spec:
  httpRoute:
    match:
      prefix: "/eks/"
    action:
      target:
        virtualService:
          virtualServiceRef:
            name: nodejs
---

```

Podemos realiar o teste, por exemplo utilizando o cloud9. O cloud9 irá subir uma VM fora do contexto do AWS App Mesh e com isso podemos realizar uma simples chamada curl para validar. 

```
curl http://${LB_DNS}/eks/
```

![AppMesh with virtual gateway](https://images-blackbelt.s3.us-west-1.amazonaws.com/appmesh5.png?response-content-disposition=inline&X-Amz-Security-Token=IQoJb3JpZ2luX2VjEJz%2F%2F%2F%2F%2F%2F%2F%2F%2F%2FwEaCXNhLWVhc3QtMSJGMEQCIFa1w866vhUbG%2F%2FdpnAAA4PfrKRpcpll7YdXm8ANYx7IAiBWlHJL9j4u0mIxhFRlBm6EbF%2BICfPDrz5LVeDkkhl6MyrmAgg1EAAaDDEyMzQzODUzMjU3NCIMx2lrtNw%2FL7cK8ZJ0KsMCwhcdGm3dDYXG6hqiZb9NtmmogCHc%2FmKYYxdoXAKTSJCK0SGQdN3ZpWf7blH7PwQFI%2FOdf59QA98BvPAUmjoTChOz%2BRUxVGwirmCXf%2FnxYmFT4CRlXlBR5jSORUvVvo53EkKTp%2FDQi%2BYEgSu8DhCuhelDi0Et8gluCBg2tJ%2B0gCNKmNYoz0GG5wu8rGPQI4FU580a8iEzf7rQJ0J8wxW4H3z5zJ1v80VEKLan1X30iauF6X9YRGb1RbV9NIldm6stdVxa9ztunvGIVecrpcMEK6M5Ia9yCkUtnpiJ6LfgXD3hHhUuugdBzdSxp2Jws6Qzn56dFElJzwQKD25ON5vOSxUM9PCepmSx%2F0aFT11fBu%2Fj%2FIyMJLnRALeR3YdBkiXuvba2gvLayznMY0kLIIX4fQ64fQFbmAaHuh9XpK00O6cfXxww78L%2FpQY6iAIYrIydxsFr2jWq7zVF7grysdRtyp3oNCH8PjLpmkP1flccKG1503uy16Ddr0ewzfe1hWAg%2BDQ8lUvougQ20CtHYMSsnIuFp3MQwLzQg6rOLKAmyMczgUxJOIMcRknfJmbvjEb2gn3udm6lTMYUp%2FUH6OCfvRuLQAwuQtIVZttAaPFv80XO4TcS56gwe3%2BHurGqJsqBvcLFW5NO1otR9pV6nsWVTpvUwIZYcJujjjSn4N4a2hSQlmqpwvwN4V71sogKCn3A1BiAFxGdnMcKEESKEX8oowc5ipdI5vKMBzTrxVR4hvL9oDZNKMtLVziml9HPoj%2FRG5klC6c1BHnELgkvrPIER34cQ%2BQ%3D&X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20230726T010230Z&X-Amz-SignedHeaders=host&X-Amz-Expires=43200&X-Amz-Credential=ASIARZPMCQ7PPD5UT76Q%2F20230726%2Fus-west-1%2Fs3%2Faws4_request&X-Amz-Signature=2076d466ad56f7cbf8b52a93bc5461b0251dedc954eecc3efba9f5629df254b0)

## Estratégias de deploy

Existem uma séries de estratégias de deploy que podem nos auxiliar no dia a dia para mitigar o impacto de alterações que são realizadas em nossos códigos produtivos e também testar essas alterações em uma escala controlada. 
Vamos avaliar como o AWS App Mesh pode facilitar e nos auxiliar na utilização de uma dessas estratégias. 

**Canary Deploy**

O canary deployment é um modelo de deploy onde os releases são feitos de modo parcial. Primariamente, um novo release é disponibilizado para uma pequena parcela de usuários para que essas pessoas possam testar as novidades e dar feedbacks sobre o que mudou. E, caso essas mudanças sejam estáveis e aceitas por essas pessoas, a atualização é realizada para as demais pessoas que utilizam o sistema.

Realizamos a alteração em uma das aplicações incluindo apenas um retorno diferente no texto. 
Esse retorno tem a palavra Canary para identificarmos a diferença. 

criação de um novo deployment para a nova versão do projeto:
Pegando uma imagem diferente

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nodejsv2-app
  labels:
    app: nodejsv2-app
  namespace: appmesh-workshop-ns
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nodejsv2-app
  strategy:
    rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 25%
    type: RollingUpdate
  template:
    metadata:
      labels:
        app: nodejsv2-app
    spec:
      containers:
      - image: 123438532574.dkr.ecr.us-west-1.amazonaws.com/nodejs-v2
        imagePullPolicy: Always
        name: nodejsvs-app
        ports:
        - containerPort: 3000
          protocol: TCP
```

Criação do virtual node apontando para o deployment criado e apresentando esse deployment para o mesh.

```
apiVersion: appmesh.k8s.aws/v1beta2
kind: VirtualNode
metadata:
  name: nodejsv2-app
  namespace: appmesh-workshop-ns
spec:
  podSelector:
    matchLabels:
      app: nodejsv2-app
  listeners:
    - portMapping:
        port: 3000
        protocol: http
  serviceDiscovery:
    awsCloudMap:
     namespaceName: "appmesh.pvt.local"
     serviceName: nodejsv2
```

Inclusão do novo virtual node no tráfego do virtual router já criado anteriormente, nesse momento precisamos apenas informar para o router que ele precisa distirbuir essa carga.

```
apiVersion: appmesh.k8s.aws/v1beta2
kind: VirtualRouter
metadata:
  name: nodejs-router
  namespace: appmesh-workshop-ns
spec:
  listeners:
    - portMapping:
        port: 3000
        protocol: http
  routes:
    - name: route-to-nodejs-app
      httpRoute:
        match:
          prefix: /
        action:
          weightedTargets:
            - virtualNodeRef:
                name: nodejs-app
              weight: 2
            - virtualNodeRef:
                name: nodejsv2-app
              weight: 1
```

Podemos olhar na parte de observabilidade que um novo serviço foi incluído e está sendo chamado. 

Vamos olhar também no painel (AWS App Mesh) como as configurações ficaram.

![AppMesh view router](https://images-blackbelt.s3.us-west-1.amazonaws.com/appmesh6.png?response-content-disposition=inline&X-Amz-Security-Token=IQoJb3JpZ2luX2VjEJz%2F%2F%2F%2F%2F%2F%2F%2F%2F%2FwEaCXNhLWVhc3QtMSJGMEQCIFa1w866vhUbG%2F%2FdpnAAA4PfrKRpcpll7YdXm8ANYx7IAiBWlHJL9j4u0mIxhFRlBm6EbF%2BICfPDrz5LVeDkkhl6MyrmAgg1EAAaDDEyMzQzODUzMjU3NCIMx2lrtNw%2FL7cK8ZJ0KsMCwhcdGm3dDYXG6hqiZb9NtmmogCHc%2FmKYYxdoXAKTSJCK0SGQdN3ZpWf7blH7PwQFI%2FOdf59QA98BvPAUmjoTChOz%2BRUxVGwirmCXf%2FnxYmFT4CRlXlBR5jSORUvVvo53EkKTp%2FDQi%2BYEgSu8DhCuhelDi0Et8gluCBg2tJ%2B0gCNKmNYoz0GG5wu8rGPQI4FU580a8iEzf7rQJ0J8wxW4H3z5zJ1v80VEKLan1X30iauF6X9YRGb1RbV9NIldm6stdVxa9ztunvGIVecrpcMEK6M5Ia9yCkUtnpiJ6LfgXD3hHhUuugdBzdSxp2Jws6Qzn56dFElJzwQKD25ON5vOSxUM9PCepmSx%2F0aFT11fBu%2Fj%2FIyMJLnRALeR3YdBkiXuvba2gvLayznMY0kLIIX4fQ64fQFbmAaHuh9XpK00O6cfXxww78L%2FpQY6iAIYrIydxsFr2jWq7zVF7grysdRtyp3oNCH8PjLpmkP1flccKG1503uy16Ddr0ewzfe1hWAg%2BDQ8lUvougQ20CtHYMSsnIuFp3MQwLzQg6rOLKAmyMczgUxJOIMcRknfJmbvjEb2gn3udm6lTMYUp%2FUH6OCfvRuLQAwuQtIVZttAaPFv80XO4TcS56gwe3%2BHurGqJsqBvcLFW5NO1otR9pV6nsWVTpvUwIZYcJujjjSn4N4a2hSQlmqpwvwN4V71sogKCn3A1BiAFxGdnMcKEESKEX8oowc5ipdI5vKMBzTrxVR4hvL9oDZNKMtLVziml9HPoj%2FRG5klC6c1BHnELgkvrPIER34cQ%2BQ%3D&X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20230726T011708Z&X-Amz-SignedHeaders=host&X-Amz-Expires=43200&X-Amz-Credential=ASIARZPMCQ7PPD5UT76Q%2F20230726%2Fus-west-1%2Fs3%2Faws4_request&X-Amz-Signature=646d8a21512a73c7cc57a40b20445f19eeba6cea1ef6606af2b51afe8a873f5e)


Vale ressaltar:

## AWS App Mesh mTLS

O mutual TLS (mTLS) oferece uma maneira de aplicar a identidade da aplicação na camada de transporte e permitir ou rejeitar conexões de cliente com base no certificado apresentado. O AWS App Mesh é compatível com a imposição de identidade de aplicação cliente usando certificados X.509, chamada de segurança mútua de camada de transporte, do inglês mutual transport layer security (mTLS). Para configurar o mTLS, é necessário configurar o cliente para fornecer um certificado ao serviço do servidor como parte da negociação de sessão TLS durante a inicialização da solicitação. Esse certificado é usado pelo servidor para identificar e autenticar o cliente, verificando se o certificado é válido e foi emitido por uma Certificate Authority (CA – Autoridade certificadora) confiável, e identificando o cliente usando o Subject Alternative Name (SAN – Nome alternativo do assunto) no certificado.


![App Mesh + AWS Services](https://images-blackbelt.s3.us-west-1.amazonaws.com/appmesh7.png?response-content-disposition=inline&X-Amz-Security-Token=IQoJb3JpZ2luX2VjEJz%2F%2F%2F%2F%2F%2F%2F%2F%2F%2FwEaCXNhLWVhc3QtMSJGMEQCIFa1w866vhUbG%2F%2FdpnAAA4PfrKRpcpll7YdXm8ANYx7IAiBWlHJL9j4u0mIxhFRlBm6EbF%2BICfPDrz5LVeDkkhl6MyrmAgg1EAAaDDEyMzQzODUzMjU3NCIMx2lrtNw%2FL7cK8ZJ0KsMCwhcdGm3dDYXG6hqiZb9NtmmogCHc%2FmKYYxdoXAKTSJCK0SGQdN3ZpWf7blH7PwQFI%2FOdf59QA98BvPAUmjoTChOz%2BRUxVGwirmCXf%2FnxYmFT4CRlXlBR5jSORUvVvo53EkKTp%2FDQi%2BYEgSu8DhCuhelDi0Et8gluCBg2tJ%2B0gCNKmNYoz0GG5wu8rGPQI4FU580a8iEzf7rQJ0J8wxW4H3z5zJ1v80VEKLan1X30iauF6X9YRGb1RbV9NIldm6stdVxa9ztunvGIVecrpcMEK6M5Ia9yCkUtnpiJ6LfgXD3hHhUuugdBzdSxp2Jws6Qzn56dFElJzwQKD25ON5vOSxUM9PCepmSx%2F0aFT11fBu%2Fj%2FIyMJLnRALeR3YdBkiXuvba2gvLayznMY0kLIIX4fQ64fQFbmAaHuh9XpK00O6cfXxww78L%2FpQY6iAIYrIydxsFr2jWq7zVF7grysdRtyp3oNCH8PjLpmkP1flccKG1503uy16Ddr0ewzfe1hWAg%2BDQ8lUvougQ20CtHYMSsnIuFp3MQwLzQg6rOLKAmyMczgUxJOIMcRknfJmbvjEb2gn3udm6lTMYUp%2FUH6OCfvRuLQAwuQtIVZttAaPFv80XO4TcS56gwe3%2BHurGqJsqBvcLFW5NO1otR9pV6nsWVTpvUwIZYcJujjjSn4N4a2hSQlmqpwvwN4V71sogKCn3A1BiAFxGdnMcKEESKEX8oowc5ipdI5vKMBzTrxVR4hvL9oDZNKMtLVziml9HPoj%2FRG5klC6c1BHnELgkvrPIER34cQ%2BQ%3D&X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20230726T012349Z&X-Amz-SignedHeaders=host&X-Amz-Expires=43200&X-Amz-Credential=ASIARZPMCQ7PPD5UT76Q%2F20230726%2Fus-west-1%2Fs3%2Faws4_request&X-Amz-Signature=228155552868ddc6ed3c9f90411675477fcffe08dacc7be327b2ef7c2f2a285f)

