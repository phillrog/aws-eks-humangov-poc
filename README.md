# SaaS na AWS EKS

Neste projeto abrangente, enfrentei o desafio de implantar a aplicação HumanGov SaaS usando a plataforma Amazon Web Services (AWS), focando especificamente no Elastic Kubernetes Service (EKS) para orquestração, juntamente com o Route 53 para gerenciamento de domínio, Application Load Balancer (ALB) para ingress e o AWS Certificate Manager para criptografia de endpoint SSL.

Minha jornada começou com o trabalho fundamental de configurar a VPC da AWS e o cluster EKS. Procedi com a instalação do kubectl e eksctl no Cloud 9, aproveitando essas ferramentas para interagir com o cluster EKS.

A aplicação foi containerizada usando o Docker, o que envolveu a criação de Dockerfiles e a otimização do processo de compilação para o aplicativo HumanGov. Uma vez containerizadas, as imagens foram enviadas para o Amazon Elastic Container Registry (ECR), garantindo que estivessem prontas para implantação.

Com o ECR configurado, elaborei manifestos de implantação e serviços do Kubernetes. Isso me permitiu definir o estado desejado da aplicação, incluindo o número de pods, limites de recursos e as variáveis de ambiente necessárias.

Para o gerenciamento de domínio e roteamento de tráfego, configurei uma zona hospedada no Route 53 e registros. Isso foi fundamental para a resolução do nome de domínio e fornecimento de uma URL amigável para o usuário acessar a aplicação.

Para gerenciar o tráfego de entrada, configurei um ALB e recursos ingress. Essa etapa foi crucial para direcionar o tráfego para os serviços corretos dentro do cluster EKS e permitir a terminação SSL no nível do ALB.

Utilizei o AWS Certificate Manager para provisionar e gerenciar o certificado SSL, garantindo comunicação criptografada e segura com a aplicação.

O resultado foi uma aplicação HumanGov SaaS altamente disponível, segura e escalável na AWS EKS, acessível por meio de um domínio seguro. Essa implantação não apenas demonstrou a eficiência do Kubernetes na AWS, mas também a robustez dos serviços de rede e segurança da AWS.

Os principais aprendizados deste projeto incluem um entendimento aprofundado do Kubernetes na AWS, gerenciamento de domínio e tráfego, criptografia SSL, containerização, Route 53, DNS provider e muito mais.

# Parte 1: EKS Cluster Setup

### Pré-requisitos

Certifique-se de ter as seguintes ferramentas instaladas:

* **EC2** (IMPORTANTE): crie uma EC2 e instale os requisistos abaixo.


* **Docker**:
    * [Instalar Docker](https://docs.docker.com/engine/install/ubuntu/)
* **Terraform**:
    * [Instalar Terraform](https://developer.hashicorp.com/terraform/install)

1. Execute o comando AWS version para verificar a versão da sua AWS CLI. Certifique-se de estar utilizando a AWS CLI 2. Caso contrário, você precisa atualizá-la antes de continuar.
    
    ```bash
    aws --version
    ```
    
    Output:
    
    ```bash
    ec2-user:~/environment $ aws --version
    **aws-cli/1.19.112** Python/2.7.18 Linux/4.14.328-248.540.amzn2.x86_64 botocore/1.20.112
    ```
    
    **Se a AWS CLI já estiver na versão 2 ou superior, por favor, vá para a etapa 2 (Instale a ferramenta CLI `eksctl`)**. Caso contrário, por favor, atualize a AWS CLI para a versão 2 agora. NÃO PULE ESTA ETAPA, caso contrário, você terá problemas ao criar o cluster EKS usando a ferramenta eksctl.
    
    ### AWS CLI 2 Upgrade Steps
    
    Remova a AWS CLI existente
    
    ```bash
    sudo yum remove awscli
    ```
    
    Instale a AWS CLI 2
    
    ```bash
    curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
    unzip awscliv2.zip
    sudo ./aws/install --bin-dir /usr/local/bin --install-dir /usr/local/aws-cli --update
    ```
    
    Certifique-se de ter a AWS CLI 2 instalada.
    
    ```bash
    aws --version
    ```
    
    Output:
    
    ```bash
    ec2-user:~/environment $ aws --version
    **aws-cli/2.14.5** Python/3.11.6 Linux/4.14.328-248.540.amzn2.x86_64 exe/x86_64.amzn.2 prompt/off
    ```
    
2. Instale a ferramenta CLI `eksctl` 
    
    ```bash
    curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
    sudo cp /tmp/eksctl /usr/bin
    eksctl version
    ```
    
3. Instale a ferramenta CLI `kubectl` 
    
    ```bash
    curl -o kubectl https://amazon-eks.s3.us-west-2.amazonaws.com/1.18.9/2020-11-02/bin/linux/amd64/kubectl
    chmod +x ./kubectl
    mkdir -p $HOME/bin && cp ./kubectl $HOME/bin/kubectl && export PATH=$PATH:$HOME/bin
    echo 'export PATH=$PATH:$HOME/bin' >> ~/.bashrc
    kubectl version --short --client
    ```
    
4. Instale a ferramenta CLI `helm`
    
    ```bash
    curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
    chmod 700 get_helm.sh
    ./get_helm.sh
    helm version
    ```
    
5. Crie um usuário IAM `eks-user` com AdministratorAccess policy.
6. Crie uma Access Key para o `eks-user`

### Subindo infra

1. No terminal 
2. Autentique com a chave de acesso do usuário `eks-user`
    
    ```bash
    export AWS_ACCESS_KEY_ID=XXXXXXXXXXXXXXXX
    export AWS_SECRET_ACCESS_KEY=XXXXXXXXXXXXXXXXXXXXXXXXXXX
    
    aws s3 ls
    ```
    
3. Usando o Terraform, provisione o S3 e a tabela DynamoDB a serem usados pela Aplicação HumanGov
    
    ```bash
    cd /human-gov-infrastructure/terraform
    terraform show
    terraform plan
    terraform apply
    ```
    
    💡 Anote o nome da sua tabela DynamoDB e do seu Bucket S3. Eles serão usados posteriormente.
    
4. Crie um EKS Cluster
    
    ```bash
    eksctl create cluster --name humangov-cluster --region us-east-1 --nodegroup-name standard-workers --node-type t3.medium --nodes 1
    
    ```
    
5. Conecte-se ao cluster EKS usando a configuração do `kubectl` 
    
    ```bash
    aws eks update-kubeconfig --name humangov-cluster
    
    ```
    
6. Verifique a conectividade do Cluster
    
    ```bash
    kubectl get svc
    kubectl get nodes
    ```
    

---

<br/>

<img width="1916" height="992" alt="image" src="https://gist.github.com/user-attachments/assets/e325f603-33a5-4cb6-82b0-7de5ed352c82" />

<br/>

<img width="1910" height="747" alt="image" src="https://gist.github.com/user-attachments/assets/dc1f4196-133d-49d9-bebf-70a8a19e4890" />

<br/>


# Parte 2: Installando Ingress Controller e ALB

1. Crie uma política IAM.
    1. Baixe uma política IAM para o AWS Load Balancer Controller que permita que ele faça chamadas às APIs da AWS em seu nome.
        
        ```bash
        cd ../..
        curl -O https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/refs/heads/main/docs/install/iam_policy.json
        ```
        
    2. Crie uma política IAM usando a política baixada na etapa anterior.
        
        ```bash
        aws iam create-policy \
            --policy-name AWSLoadBalancerControllerIAMPolicy \
            --policy-document file://iam_policy.json
        ```
- **Este comando da AWS CLI cria uma nova política IAM chamada AWSLoadBalancerControllerIAMPolicy. Ele lê as permissões dessa política de um arquivo JSON local chamado iam_policy.json.**        
        
2. Crie um provedor de identidade IAM OIDC para o seu cluster com o `eksctl`
    
    ```bash
    eksctl utils associate-iam-oidc-provider --cluster humangov-cluster --approve
    ```
    
3. Crie uma função IAM e uma conta de serviço Kubernetes com o nome **`aws-load-balancer-controller`** no namespace **`kube-system`** para o AWS Load Balancer Controller e adicione uma anotação à conta de serviço Kubernetes com o nome da IAM role.

Substitua `*111122223333*` pelo ID da sua conta e, em seguida, execute o comando:.

```bash
eksctl create iamserviceaccount \
  --cluster=humangov-cluster \
  --namespace=kube-system \
  --name=aws-load-balancer-controller \
  --role-name AmazonEKSLoadBalancerControllerRole \
  --attach-policy-arn=arn:aws:iam::**111122223333**:policy/AWSLoadBalancerControllerIAMPolicy \
  --approve
```

- **Permite que o controlador de Load Balancer no seu EKS se comunique de forma segura e autorizada com a AWS para gerenciar os Application Load Balancers (ALBs) que você vai configurar para a sua aplicação**

1. Instale o AWS Load Balancer Controller usando o [Helm V3](https://docs.aws.amazon.com/eks/latest/userguide/helm.html) ou posterior, ou aplicando um manifesto Kubernetes..
    1. Adicione o repositório `eks-charts` .
        
        ```bash
        helm repo add eks https://aws.github.io/eks-charts
        ```
        
    2. Atualize seu repositório local para garantir que você tenha os charts mais recentes.
        
        ```bash
        helm repo update eks
        ```
        
    3. Instale o AWS Load Balancer Controller.
    
    No comando a seguir, `aws-load-balancer-controller` é a conta de serviço Kubernetes que você criou em uma etapa anterior.
        
   ```bash
   helm install aws-load-balancer-controller eks/aws-load-balancer-controller \
          -n kube-system \
          --set clusterName=humangov-cluster \
          --set serviceAccount.create=false \
          --set serviceAccount.name=aws-load-balancer-controller
   ```
        
2. Verifique que o controller está instalado.

```bash
kubectl get deployment -n kube-system aws-load-balancer-controller
```

Um exemplo de output é o seguinte:

<img width="957" height="112" alt="image" src="https://gist.github.com/user-attachments/assets/8fa6ab5e-d189-4218-874a-be7a001ed45e" />

---

# Parte 3: Implantando o aplicativo HumanGov

### Pré-requisito

Crie uma Role & Service Account para fornecer aos pods acesso às tabelas S3 e DynamoDB.

```bash
eksctl create iamserviceaccount \
  --cluster=humangov-cluster \
  --name=humangov-pod-execution-role \
  --role-name HumanGovPodExecutionRole \
  --attach-policy-arn=arn:aws:iam::aws:policy/AmazonS3FullAccess \
  --attach-policy-arn=arn:aws:iam::aws:policy/AmazonDynamoDBFullAccess \
  --region us-east-1 \
  --approve
```

### **Coloque a aplicação HumanGov em container**

Mude para o diretório da aplicação.

```jsx
cd human-gov-application/src
```

### **Crie um Repositório ECR**

Crie um novo repositório ECR público chamado **humangov-app**, build and push a imagem Docker para o repositório ECR criado (siga os passos “View push commands” conforme mostrado no vídeo)

1. Recupere o token de autenticação e autentique o seu cliente Docker no seu registro. Use o AWS CLI:
    
    ```bash
    aws ecr-public get-login-password --region us-east-1 | docker login --username AWS --password-stdin public.ecr.aws/k5z6r9x6
    ```
    
    Observação: Se você tiver um erro ao usar o AWS CLI, certifique-se de que tem a versão mais recente do AWS CLI e do Docker instalada.
    
2. Construa sua imagem Docker usando o seguinte comando. Para obter informações sobre como construir um arquivo Docker do zero, consulte as instruções [aqui](http://docs.aws.amazon.com/AmazonECS/latest/developerguide/docker-basics.html). Você pode pular esta etapa se sua imagem já estiver construída:
    
    ```bash
    docker build -t humangov-app .
    ```
    
3. Após a conclusão do build, adicione uma tag à sua imagem, para que você possa fazer o push da imagem para este repositório:
    
    ```bash
    docker tag humangov-app:latest public.ecr.aws/k5z6r9x6/humangov-app:latest
    ```
    
4. Execute o seguinte comando para fazer o push desta imagem para o novo repositório AWS que você criou:
    
    ```bash
    docker push public.ecr.aws/k5z6r9x6/humangov-app:latest
    ```

<img width="1911" height="968" alt="Captura de tela 2025-07-19 181343" src="https://gist.github.com/user-attachments/assets/ee393e73-972a-4fc8-ba5a-a7b8c971f98e" />



### **Implemente a Aplicação para Cada Estado**

Use seus arquivos YAML para implementar seu serviço. 
Crie o arquivo `humangov-sao-paulo.yaml` no diretório `human-gov-application/src` . 

Obs: Atenção ao nome do S3.

**Substitua o caminho da imagem do ECR e o nome do Bucket da AWS** pelos nomes dos seus recursos.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: humangov-python-app-sao-paulo
spec:
  replicas: 1
  selector:
    matchLabels:
      app: humangov-python-app-sao-paulo
  template:
    metadata:
      labels:
        app: humangov-python-app-sao-paulo
    spec:
      serviceAccountName: humangov-pod-execution-role
      containers:
      - name: humangov-python-app-sao-paulo
        image: SUA_IMAGEM_NA_AWS_ECR
        env:
        - name: AWS_BUCKET
          value: "humangov-sao-paulo-s3-lgyn"
        - name: AWS_DYNAMODB_TABLE
          value: "humangov-sao-paulo-dynamodb"
        - name: AWS_REGION
          value: "us-east-1"
        - name: US_STATE
          value: "sao-paulo"

---

apiVersion: v1
kind: Service
metadata:
  name: humangov-python-app-service-sao-paulo
spec:
  type: ClusterIP
  selector:
    app: humangov-python-app-sao-paulo
  ports:
    - protocol: TCP
      port: 8000
      targetPort: 8000

---

apiVersion: apps/v1
kind: Deployment
metadata:
  name: humangov-nginx-reverse-proxy-sao-paulo
spec:
  replicas: 1
  selector:
    matchLabels:
      app: humangov-nginx-reverse-proxy-sao-paulo
  template:
    metadata:
      labels:
        app: humangov-nginx-reverse-proxy-sao-paulo
    spec:
      containers:
      - name: humangov-nginx-reverse-proxy-sao-paulo
        image: nginx:alpine
        ports:
        - containerPort: 80
        volumeMounts:
        - name: humangov-nginx-config-sao-paulo-vol
          mountPath: /etc/nginx/
      volumes:
      - name: humangov-nginx-config-sao-paulo-vol
        configMap:
          name: humangov-nginx-config-sao-paulo

---

apiVersion: v1
kind: Service
metadata:
  name: humangov-nginx-service-sao-paulo
spec:
  selector:
    app: humangov-nginx-reverse-proxy-sao-paulo
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80

---

apiVersion: v1
kind: ConfigMap
metadata:
  name: humangov-nginx-config-sao-paulo
data:
  nginx.conf: |

    events {
      worker_connections 1024;
    }

    http {

      server {
        listen 80;

        location / {
          proxy_set_header Host $host;
          proxy_set_header X-Real-IP $remote_addr;
          proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
          proxy_pass http://humangov-python-app-service-sao-paulo:8000; # App container
        }
      }
    }
  
  proxy_params: |
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
```

```bash
kubectl apply -f humangov-sao-paulo.yaml

```

<img width="1008" height="126" alt="image" src="https://gist.github.com/user-attachments/assets/4981ff61-a489-4ceb-a1f0-47139b67db72" />

<br/>

# Parte 4: Ingress e Configuração do ALB com SSL

## Introdução

Neste guia, vamos explicar o processo de implantação da aplicação HumanGov de vários estados no AWS Elastic Kubernetes Service (EKS). Estaremos usando Ingress controllers e o Application Load Balancer (ALB) da AWS para o roteamento com base em subdomínios de estado, como `rio-de-janeiro.humangov-prs.click` e `sao-paulo.humangov-prs.click`. Também configuraremos SSL para comunicação segura. Este guia será dividido em várias partes para tornar o processo mais fácil de seguir.

### Pré-requisitos

Crie um domínio .click, na AWS custa USD 3,00, na hostinger custa R$ 11,00. Caso o domínio for de provedor externo a AWS, na AWS crie o Hosted Zone com o dominio e pegue os DNS tipo NS e coloque nos DNS do provedor.

## Instruções Detalhadas

1. Crie um novo domínio no Route 53 (os domínios .click geralmente são os mais baratos)
2. Crie um Certificado para o ALB
    
    Acesse o AWS Certificate Manager e solicite um novo certificado público para `*.yourdomain.click`.
    
<br/>

<img width="1912" height="930" alt="image" src="https://gist.github.com/user-attachments/assets/6ab10d54-ecc4-4240-a59b-09e2eab8f089" />

<br/>


3. Implemente as Ingress Rules
    
    Crie um arquivo YAML de Ingress (`humangov-ingress-all.yaml`) que inclua regras para cada estado. **Certifique-se de substituir o Certificate ARN em vermelho pelo seu Certificate ARN.**
    
    ```yaml
    apiVersion: networking.k8s.io/v1
    kind: Ingress
    metadata:
      name: humangov-python-app-ingress
      annotations:
        alb.ingress.kubernetes.io/scheme: internet-facing
        alb.ingress.kubernetes.io/target-type: ip
        alb.ingress.kubernetes.io/group.name: frontend
       ** alb.ingress.kubernetes.io/certificate-arn: arn:aws:acm:us-east-1:xxxxxxxxxxx:certificate/xxxxxxxxxxx**
        alb.ingress.kubernetes.io/listen-ports: '[{"HTTP": 80}, {"HTTPS":443}]'
        alb.ingress.kubernetes.io/ssl-redirect: '443'
      labels:
        app: humangov-python-app-ingress
    spec:
      ingressClassName: alb
      rules:
        - host: sao-paulo.humangov-prs.click
          http:
            paths:
            - path: /
              pathType: Prefix
              backend:
                service:
                  name: humangov-nginx-service-sao-paulo
                  port:
                    number: 80
    ```
    
    Aplique o ingress:
    
    ```bash
    kubectl apply -f humangov-ingress-all.yaml
    
    ```
    
4. **Verifique o Ingress Controller**
    
    ```bash
    kubectl get ingress
    
    ```
    
5. Crie uma entrada de alias (Record Type: A) no Route 53 que aponte o subdomínio para o domínio do ALB.
Exemplo: `sao-paulo.humangov-prs.click` → `dualstack.k8s-frontend-ded5adda2e-875432214.us-east-1.elb.amazonaws.com` 
    

6. Teste a aplicação (por exemplo, `sao-paulo.humangov-prs.click` - **substitua `humangov-prs.click` pelo seu domínio**)
    
### **Repita o processo para implementar a aplicação do HumanGov no Rio de Janeiro no mesmo cluster**.
    
1. Provisione o DynamoDB e o bucket S3 para a aplicação no Rio de Janeiro com o Terraform e anote os nomes dos recursos. Abra o arquivo Terraform `human-gov-infrastructure/terraform/variables.tf`  usando editor e adicione `rio-de-janeiro` à lista de estados.

<br/>

```bash
variable "states" {
  description = "A list of state names"
  default     = ["sao-paulo", "rio-de-janeiro"]
}
```
    
<br/>

<img width="807" height="240" alt="image" src="https://gist.github.com/user-attachments/assets/3ce0f1f0-36d8-404f-b724-f1fc43a8c592" />

<br/>
    
Aplique a configuração do Terraform.
    
```bash
cd /human-gov-infrastructure/terraform
terraform plan
terraform apply
```
    
 2. Duplique o arquivo de implementação do Kubernetes
    
```bash
cd /human-gov-application/src
cp humangov-sao-paulo.yaml humangov-rio-de-janeiro.yaml
```
    
 3. Abra o arquivo `humangov-rio-de-janeiro.yaml` e substitua todas as entradas `sao-paulo` por `rio-de-janeiro` usando a função de busca e substituição.
 4. Atualize o nome do AWS_BUCKET para o nome do bucket da Rio de Janeiro no arquivo `humangov-rio-de-janeiro.yaml`. Salve o arquivo.
 5. Implemente a aplicação do HumanGov no Rio de Janeiro.
    
```bash
kubectl apply -f humangov-rio-de-janeiro.yaml
```

<img width="1103" height="176" alt="image" src="https://gist.github.com/user-attachments/assets/7b66ba52-0cd4-4a59-b989-300941608a07" />

    
 6. Abra o arquivo `humangov-ingress-all.yaml` no editor e adicione a Ingress rule para o estado do Rio de Janeiro. **Certifique-se de substituir o Certificate ARN em pelo seu Certificate ARN.**
    
 ```yaml
 apiVersion: networking.k8s.io/v1
 kind: Ingress
 metadata:
   name: humangov-python-app-ingress
   annotations:
     alb.ingress.kubernetes.io/scheme: internet-facing
     alb.ingress.kubernetes.io/target-type: ip
     alb.ingress.kubernetes.io/group.name: frontend
     **alb.ingress.kubernetes.io/certificate-arn: arn:aws:acm:us-east-1:xxxxxxxx:certificate/xxxxxxxxxxx**
     alb.ingress.kubernetes.io/listen-ports: '[{"HTTP": 80}, {"HTTPS":443}]'
     alb.ingress.kubernetes.io/ssl-redirect: '443'
   labels:
     app: humangov-python-app-ingress
 spec:
   ingressClassName: alb
   rules:
     - host: sao-paulo.humangov-prs.click
       http:
         paths:
         - path: /
           pathType: Prefix
           backend:
             service:
               name: humangov-nginx-service-sao-paulo
               port:
                 number: 80
     **- host: rio-de-janeiro.humangov-prs.click
       http:
         paths:
         - path: /
           pathType: Prefix
           backend:
             service:
               name: humangov-nginx-service-rio-de-janeiro
               port:
                 number: 80**
 ```
    
7. Implemente as mudanças ingress

```bash
kubectl apply -f humangov-ingress-all.yaml
```

8. Adicione a entrada DNS do Route 53 para o estado do Rio de Janeiro.

<img width="1905" height="801" alt="image" src="https://gist.github.com/user-attachments/assets/fef40ba6-7878-4a75-8e68-ff558e7f4c20" />

<img width="1913" height="858" alt="image" src="https://gist.github.com/user-attachments/assets/b61e4d92-926d-4609-8ab1-d1c92446d246" />


9. Teste a aplicação (por exemplo `[sao-paulo.humangov-prs.click](https://sao-paulo.humangov-prs.click/)` e[`rio-de-janeiro.humangov-prs.click`](https://rio-de-janeiro.humangov-prs.click/) - **substitua `humangov-prs.click` pelo seu domínio**). 

<br/>

<img width="1915" height="961" alt="image" src="https://gist.github.com/user-attachments/assets/90d8f82d-100d-47af-821d-9aa61a308acc" />

<br/>

<img width="1917" height="962" alt="image" src="https://gist.github.com/user-attachments/assets/05821142-8152-4867-89fa-9a19e1198025" />

<br/>

10. Analise os recursos no nível do Kubernetes e o ALB no nível da Console AWS.

```bash
kubectl get pods
kubectl get deployment
kubectl get svc
kubectl get ingress
```

## Passo Final: Destruindo o ambiente

1. Exclua o Kubernetes Ingress
    
    ```bash
    kubectl delete -f humangov-ingress-all.yaml
    ```
    
2. Exclua os recursos restantes da aplicação no Kubernetes.
    
    ```bash
    kubectl delete -f humangov-sao-paulo.yaml
    kubectl delete -f humangov-rio-de-janeiro.yaml
    ```
    
3. Exclua o EKS Cluster usando a CLI
    
    ```bash
    eksctl delete cluster --name humangov-cluster --region us-east-1
    
    ```
    
4. Aguarde até que o cluster do EKS tenha sido excluído. Verifique novamente na Console da AWS.

5. Termine a instância EC2.
