# nodePort

Um NodePort é uma porta aberta no nó de seu cluster Kubernetes (K8S). 

O K8S roteia de maneira transparente o tráfego de entrada no NodePort para o seu serviço, mesmo se o aplicativo estiver sendo executado em um nó distinto do chamado pela URL.

Para funcionar é obrigatório editar as regras do Ingress Rules da Subnet, dentro do Oracle Cloud.

Um serviço de NodePort é atribuído a partir de um pool de intervalos NodePort configurados para cluster (normalmente 30000–32767). 


## Criar um cluster K8S no OKE - Oracle Container Engine for Kubernetes 

Criar um cluster no Kubernetes, desde a Console do OCI com o seguinte:
- kubernetes v. 1.20.11
- atribuir os worker numa subnet píblica
- 3 worker nodes (E4.Flex, com 1 oCPU e 2 Gb de RAM)

<img src='https://objectstorage.us-ashburn-1.oraclecloud.com/n/idsvh8rxij5e/b/imagens_git/o/create_oke_.png'/>

### Configurar o acesso ao cluster Kubernetes

Dada a criação do cluster, configurar o acesso localmente, conforme descrito no menu "Quick Start", localizado no Detalhe do cluster.

<img src='https://objectstorage.us-ashburn-1.oraclecloud.com/n/idsvh8rxij5e/b/imagens_git/o/create_oke_detail.png'/>

Em resumo são dois comandos que devem ser executados e apresentados acima no Quick Start:

- mkdir -p $HOME/.kube
- oci ce cluster create-kubeconfig .... 

### Testar o acesso ao cluster via Console

```
$ alias k=kubectl
$ k get nodes
```
<img src='https://objectstorage.us-ashburn-1.oraclecloud.com/n/idsvh8rxij5e/b/imagens_git/o/get_node.png'/>

## Deploy do Apache

- Atribuir label a dois nós de sua escolha. Veja o exemplo abaixo:

```
$ k label nodes 10.0.10.52 destino=exemplo
$ k label nodes 10.0.10.94 destino=exemplo
```

- Executar deploy do Apache que destinará as réplicas aos nós com o label acima

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: apache
  labels:
      name: apache
spec:
  selector:
    matchLabels:
      app: apache
  replicas: 4
  template:
    metadata:
      labels:
        app: apache
    spec:
      containers:
      - name: apache
        image: httpd:latest
        ports:
        - name: http 
          containerPort: 80
        imagePullPolicy: IfNotPresent
      nodeSelector:
        destino: exemplo
```

<img src='https://objectstorage.us-ashburn-1.oraclecloud.com/n/idsvh8rxij5e/b/imagens_git/o/get_po_np.png'>

- Criar o serviço de tipo NodePort que aponta a aplicação apache

```
apiVersion: v1
kind: Service
metadata:
  name: apache
  labels:
    name: apache
spec:
  type: NodePort
  ports:
  - name: http
    port: 80
    targetPort: http
  selector:
    app: apache
```

<img src='https://objectstorage.us-ashburn-1.oraclecloud.com/n/idsvh8rxij5e/b/imagens_git/o/get_svc_np.png'/>


## Liberar Acesso

- Dos três nós do cluster k8s, somente 1 está sem as réplicas do Apache e sem o label chamado destino. Neste meu caso o node 10.0.10.37.
- Via console do Oracle OCI e detalhes da instância 10.0.10.37, acessar a subnet atrelada.
- Acessar o menu Ingress Rules.
- Criar uma nova regra desde qualquer origem a porta de range 3XXXX apresentada no serviço apache verificado na imagem acima (32274).

<img src='https://objectstorage.us-ashburn-1.oraclecloud.com/n/idsvh8rxij5e/b/imagens_git/o/ingress_nodeport.png'/>


## Teste

Na console do OCI é apresentado o IP público na lista de instâncias e no detalhe da instância

<img src='https://objectstorage.us-ashburn-1.oraclecloud.com/n/idsvh8rxij5e/b/imagens_git/o/list_nodes.png'/>

Desde o navegador chamar a URL do IP_EXTERNO:<NodePort> do nó que não tem o label destino, logo não tem réplicas do Apache.
  
<img src='https://objectstorage.us-ashburn-1.oraclecloud.com/n/idsvh8rxij5e/b/imagens_git/o/apache.png'/>
  
<hr>
:smiling_face_with_three_hearts:
