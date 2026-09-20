# AWS-CloudFormation---DIO
AWS CloudFormation: Automação e Gerenciamento de Infraestrutura em Nuvem

O AWS CloudFormation é o serviço de Infrastructure as Code da Amazon Web Services (AWS). Ele permite que o desenvolvedor ou administrador de infraestrutura descreva os recursos necessários para uma aplicação em um arquivo de configuração, geralmente utilizando YAML ou JSON.

Por meio desse arquivo, é possível criar e gerenciar recursos como:

máquinas virtuais EC2;
redes VPC;
sub-redes;
Internet Gateway;
Security Groups;
Load Balancers;
bancos de dados RDS;
buckets S3;
funções Lambda;
regras de IAM;
filas SQS;
tabelas DynamoDB;
entre diversos outros serviços da AWS.

Dessa forma, o CloudFormation permite transformar uma infraestrutura que normalmente seria criada manualmente em um processo automatizado, versionável e reproduzível.

O objetivo deste trabalho é apresentar o funcionamento do AWS CloudFormation, demonstrando seus principais conceitos e sua utilização na criação e gerenciamento de infraestrutura na AWS.

Como exemplo prático, será apresentada a criação de uma máquina virtual Amazon EC2, incluindo:

definição da imagem do sistema operacional;
escolha do tipo da instância;
criação de um Security Group;
configuração de regras de acesso;
criação de uma VPC;
criação de uma subnet;
associação da EC2 à rede;
utilização de parâmetros;
utilização de UserData;
disponibilização de informações através de Outputs.

Também será apresentada uma arquitetura que poderia ser utilizada em uma aplicação desenvolvida em .NET, Angular e PostgreSQL.

Templates são arquivos escritos principalmente em YAML e JSON

Estruturas dos templates podem conter varis seções:
AWSTemplateFormatVersion: Define a versão do formato do template
Description: Descreve a finalidade do template
Parameters: Permite receber valores externos
Mappings: Permite criar mapas de valores
Conditions: Permite criar recursos condicionalmente
Resources: Define os recursos AWS
Outputs: Retorna informações da infraestrutura

Exemplo simples de criação de uma EC2: 
AWSTemplateFormatVersion: '202026-09-09'
Description: >
  Exemplo de infraestrutura contendo
  VPC, Subnet, Security Group e EC2.
Parameters:
  TipoInstancia:
    Type: String
    Default: t3.micro
    AllowedValues:
      - t3.micro
      - t3.small
      - t3.medium
      
Resources:
  MinhaVPC:
    Type: AWS::EC2::VPC
    Properties:
      CidrBlock: 10.0.0.0/16
      Tags:
        - Key: Name
          Value: MinhaVPC


  MinhaSubnet:
    Type: AWS::EC2::Subnet
    Properties:
      VpcId: !Ref MinhaVPC
      CidrBlock: 10.0.1.0/24
      AvailabilityZone: sa-east-1a
      Tags:
        - Key: Name
          Value: MinhaSubnet

  MeuSecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: Acesso da aplicacao
      VpcId: !Ref MinhaVPC
      SecurityGroupIngress:
        - IpProtocol: tcp
          FromPort: 22
          ToPort: 22
          CidrIp: 0.0.0.0/0
        - IpProtocol: tcp
          FromPort: 80
          ToPort: 80
          CidrIp: 0.0.0.0/0
          
  MinhaInstancia:
    Type: AWS::EC2::Instance
    Properties:
      ImageId: ami-xxxxxxxx
      InstanceType: !Ref TipoInstancia
      SubnetId: !Ref MinhaSubnet
      SecurityGroupIds:
        - !GetAtt MeuSecurityGroup.GroupId
      Tags:
        - Key: Name
          Value: ServidorAplicacao
      UserData:
        Fn::Base64: |
          #!/bin/bash
          echo "Servidor iniciado" > /tmp/status.txt

Outputs:
  InstanceId:
    Description: ID da instancia
    Value: !Ref MinhaInstancia
