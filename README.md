# 🧱 O que é o AWS CloudFormation?
Implementar uma infraestrutura automatizada com AWS CloudFormation é uma prática essencial para equipes que desejam escalar com segurança, agilidade e controle
O AWS CloudFormation é um serviço que permite criar e gerenciar recursos da AWS por meio de arquivos de configuração, geralmente escritos em YAML ou JSON. Ele segue o conceito de Infraestrutura como Código (IaC), permitindo que você defina toda a arquitetura de forma declarativa.

🛠️ Exemplo do Dia a Dia: Criando uma Infraestrutura Web Automatizada
Imagine que você precisa montar uma aplicação web simples com:

Um bucket S3 para armazenar arquivos estáticos
Uma instância EC2 para rodar o backend
Um grupo de segurança para liberar a porta 80
Um banco de dados RDS MySQL

🔧 Etapa 1: Criar o Template CloudFormation (YAML)
YAMLAWSTemplateFormatVersion: '2010-09-09'Resources:  MyS3Bucket:    Type: AWS::S3::Bucket    Properties:      BucketName: meu-site-estatico  MySecurityGroup:    Type: AWS::EC2::SecurityGroup    Properties:      GroupDescription: Liberar acesso HTTP      SecurityGroupIngress:        - IpProtocol: tcp          FromPort: 80          ToPort: 80          CidrIp: 0.0.0.0/0  MyEC2Instance:    Type: AWS::EC2::Instance    Properties:      InstanceType: t2.micro      ImageId: ami-0abcdef1234567890      SecurityGroups:        - !Ref MySecurityGroup  MyRDSInstance:    Type: AWS::RDS::DBInstance    Properties:      DBInstanceClass: db.t2.micro      Engine: MySQL      MasterUsername: admin      MasterUserPassword: senhaSegura123      AllocatedStorage: 20Mostrar mais linhas

📦 Etapa 2: Implantar o Template
Você pode usar o AWS CLI:
Shellaws cloudformation create-stack \  --stack-name minha-infra-web \  --template-body file://infra.yaml \  --capabilities CAPABILITY_NAMED_IAMMostrar mais linhas

🔄 Etapa 3: Atualizar ou Reverter
Se precisar mudar algo (ex: tipo da instância EC2), basta editar o template e rodar:
Shellaws cloudformation update-stack \  --stack-name minha-infra-web \  --template-body file://infra.yamlMostrar mais linhas
Se algo der errado, você pode reverter automaticamente para o estado anterior.

✅ Vantagens no Dia a Dia

Automação total: não precisa criar recursos manualmente no console.
Reprodutibilidade: pode usar o mesmo template em diferentes ambientes (dev, staging, prod).
Controle de versão: templates podem ser versionados no Git.
Auditoria e conformidade: tudo fica documentado e rastreável.


📦 CloudFormation Template (YAML) – Microsserviços com ECS Fargate


AWSTemplateFormatVersion: '2010-09-09'
Description: Infraestrutura para microsserviços com ECS Fargate e ALB

Resources:

  VPC:
    Type: AWS::EC2::VPC
    Properties:
      CidrBlock: 10.0.0.0/16
      EnableDnsSupport: true
      EnableDnsHostnames: true
      Tags:
        - Key: Name
          Value: MicroserviceVPC

  PublicSubnet:
    Type: AWS::EC2::Subnet
    Properties:
      VpcId: !Ref VPC
      CidrBlock: 10.0.1.0/24
      MapPublicIpOnLaunch: true
      AvailabilityZone: !Select [0, !GetAZs '']
      Tags:
        - Key: Name
          Value: PublicSubnet

  PrivateSubnet:
    Type: AWS::EC2::Subnet
    Properties:
      VpcId: !Ref VPC
      CidrBlock: 10.0.2.0/24
      AvailabilityZone: !Select [1, !GetAZs '']
      Tags:
        - Key: Name
          Value: PrivateSubnet

  InternetGateway:
    Type: AWS::EC2::InternetGateway
    Properties:
      Tags:
        - Key: Name
          Value: MicroserviceIGW

  AttachGateway:
    Type: AWS::EC2::VPCGatewayAttachment
    Properties:
      VpcId: !Ref VPC
      InternetGatewayId: !Ref InternetGateway

  PublicRouteTable:
    Type: AWS::EC2::RouteTable
    Properties:
      VpcId: !Ref VPC

  PublicRoute:
    Type: AWS::EC2::Route
    DependsOn: AttachGateway
    Properties:
      RouteTableId: !Ref PublicRouteTable
      DestinationCidrBlock: 0.0.0.0/0
      GatewayId: !Ref InternetGateway

  ECSCluster:
    Type: AWS::ECS::Cluster
    Properties:
      ClusterName: MicroserviceCluster

  TaskExecutionRole:
    Type: AWS::IAM::Role
    Properties:
      AssumeRolePolicyDocument:
        Version: '2012-10-17'
        Statement:
          - Effect: Allow
            Principal:
              Service: ecs-tasks.amazonaws.com
            Action: sts:AssumeRole
      ManagedPolicyArns:
        - arn:aws:iam::aws:policy/service-role/AmazonECSTaskExecutionRolePolicy

  TaskDefinition:
    Type: AWS::ECS::TaskDefinition
    Properties:
      Family: MicroserviceTask
      Cpu: '256'
      Memory: '512'
      NetworkMode: awsvpc
      RequiresCompatibilities:
        - FARGATE
      ExecutionRoleArn: !GetAtt TaskExecutionRole.Arn
      ContainerDefinitions:
        - Name: app
          Image: nginx
          PortMappings:
            - ContainerPort: 80

  Service:
    Type: AWS::ECS::Service
    Properties:
      Cluster: !Ref ECSCluster
      DesiredCount: 2
      LaunchType: FARGATE
      TaskDefinition: !Ref TaskDefinition
      NetworkConfiguration:
        AwsvpcConfiguration:
          AssignPublicIp: ENABLED
          Subnets:
            - !Ref PublicSubnet
          SecurityGroups:
            - !Ref ServiceSecurityGroup

  ServiceSecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: Allow HTTP
      VpcId: !Ref VPC
      SecurityGroupIngress:
        - IpProtocol: tcp
          FromPort: 80
          ToPort: 80
          CidrIp: 0.0.0.0/0



🧠 Boas Práticas

Use parâmetros para tornar o template reutilizável.
Separe recursos em stacks menores para facilitar manutenção.
Combine com AWS CodePipeline para CI/CD.
Use StackSets para replicar em múltiplas regiões ou contas.

SAIBA MAIS: https://docs.aws.amazon.com/pt_br/forecast/latest/dg/tutorial-cloudformation.html
