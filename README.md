AWS Cloud Formation é um processo que auxilia na automação de criação de recursos na aws por meio de templates json ou yaml.

Podemos utilizar os templates quantas vezes quisermos e pagarmos apenas pela stacks criadas( conjunto de recursos , ex EC2, RDS, S3, etc

Alem de ser um processo automatizado, conseguimos versionar estes templates. Com ele podemos criar destes um recurso simples como um Ec2 até um arquitetura robusta com varios recursos

Aula Prática — AWS CloudFormation

1. Objetivo da aula

Nesta aula vamos aprender, de forma prática, como utilizar o AWS CloudFormation para criar infraestrutura na AWS através de código.

Ao final, teremos uma infraestrutura semelhante a:

                    AWS CloudFormation
                           |
                           v
                         Stack
                           |
             +-------------+-------------+
             |                           |
             v                           v
            VPC                    Security Group
             |
             v
           Subnet
             |
             v
            EC2
             |
             v
      Aplicação / Servidor


2. Pré-requisitos

Antes de começar, é necessário ter:

Uma conta AWS;
Acesso ao AWS Management Console;
Permissão para utilizar CloudFormation;
Permissão para criar EC2, VPC e Security Groups;
AWS CLI instalada, caso queira executar os comandos pelo terminal.
Atenção: a criação de recursos AWS pode gerar custos. Para uma aula prática, utilize recursos de baixo custo e exclua a Stack ao final quando os recursos não forem mais necessários.

3. O que é Infrastructure as Code?
Infrastructure as Code, ou IaC, significa definir a infraestrutura através de código.
Sem IaC, poderíamos fazer:
Console AWS
    |
    +-- Criar VPC
    +-- Criar Subnet
    +-- Criar Security Group
    +-- Criar EC2
    +-- Configurar regras
    +-- Configurar servidor

Com CloudFormation:

template.yaml
      |
      v
CloudFormation
      |
      v
Infraestrutura AWS

A grande vantagem é que o mesmo arquivo pode ser utilizado novamente para reproduzir a infraestrutura.

4. O que é CloudFormation?

O AWS CloudFormation é o serviço da AWS utilizado para definir, criar e gerenciar recursos de infraestrutura através de templates.
Os templates podem ser escritos em: YAML; JSON.

Nesta aula vamos utilizar YAML, porque ele é mais simples de ler.
Exemplo mínimo:
AWSTemplateFormatVersion: '2026-09-09'
Resources:
  MinhaInstancia:
    Type: AWS::EC2::Instance
    Properties:
      ImageId: ami-xxxxxxxx
      InstanceType: t3.micro

Esse código descreve uma EC2.

5. Conceito de Stack
Quando executamos um template no CloudFormation, criamos uma Stack.
Podemos pensar assim:

template.yaml
      |
      v
CloudFormation
      |
      v
Stack
      |
      +-- VPC
      +-- Subnet
      +-- Security Group
      +-- EC2

A Stack permite que os recursos sejam tratados como um conjunto.

6. Preparando o projeto
Crie uma pasta:
cloudformation-aula/
Dentro dela:
cloudformation-aula/
|
+-- template.yaml
|
+-- README.md

O arquivo principal será:
template.yaml

7. Primeiro template

Abra o template.yaml e coloque:
AWSTemplateFormatVersion: '2026-09-09'

Description: Aula pratica de AWS CloudFormation
Resources:
  MinhaInstancia:
    Type: AWS::EC2::Instance
    Properties:
      ImageId: ami-xxxxxxxx
      InstanceType: t3.micro

Entendendo o código

AWSTemplateFormatVersion

AWSTemplateFormatVersion: '2026-09-09'
Define a versão do formato do template.

Description: Aula pratica de AWS CloudFormation
É uma descrição do template.
Resources: É a seção onde declaramos os recursos AWS.
MinhaInstancia: É o nome lógico do recurso.
Type: AWS::EC2::Instance
Define o tipo do recurso.
Properties: Define as configurações do recurso.

8. Escolhendo a AMI
Na EC2 precisamos definir uma AMI:
ImageId: ami-xxxxxxxx
O valor precisa corresponder a uma AMI válida na região escolhida.
Por exemplo:
Região: sa-east-1
A AMI de uma região não deve ser presumida como válida em outra região.
Para descobrir uma AMI adequada:
Abra o serviço EC2;
Escolha a região desejada;
Procure por AMIs;
Identifique a imagem do sistema operacional;
Copie o AMI ID;
Substitua no template.
Exemplo: ImageId: ami-1234567890abcdef
O valor acima é apenas ilustrativo. Não copie esse ID como se fosse uma AMI real.

9. Escolhendo o tipo da instância
Podemos definir:
InstanceType: t3.micro
Exemplos de tipos:
t3.micro
t3.small
t3.medium
Para uma aula simples, uma instância pequena pode ser suficiente, desde que esteja disponível e seja adequada à conta/região.

10. Criando um Security Group
Agora vamos melhorar o template.
Adicione:
MeuSecurityGroup:
  Type: AWS::EC2::SecurityGroup
  Properties:
    GroupDescription: Security Group da aula
    SecurityGroupIngress:
      - IpProtocol: tcp
        FromPort: 80
        ToPort: 80
        CidrIp: 0.0.0.0/0
Esse Security Group permite tráfego HTTP pela porta 80.

A estrutura agora é:

CloudFormation
      |
      +-- Security Group
      |      |
      |      +-- TCP 80
      |
      +-- EC2

11. Associando o Security Group à EC2
Agora precisamos informar à EC2 qual Security Group utilizar.
SecurityGroupIds:
  - !GetAtt MeuSecurityGroup.GroupId

A EC2 ficará:
MinhaInstancia:
  Type: AWS::EC2::Instance
  Properties:
    ImageId: ami-xxxxxxxx
    InstanceType: t3.micro
    SecurityGroupIds:
      - !GetAtt MeuSecurityGroup.GroupId

O que é !GetAtt?
!GetAtt MeuSecurityGroup.GroupId
Significa:
Obtenha o atributo GroupId do recurso MeuSecurityGroup.

12. Criando uma VPC

Agora vamos controlar a rede da nossa aplicação.
Adicione:
MinhaVPC:
  Type: AWS::EC2::VPC
  Properties:
    CidrBlock: 10.0.0.0/16
    Tags:
      - Key: Name
        Value: AulaCloudFormation

A estrutura passa a ser:
VPC
10.0.0.0/16

13. Criando uma Subnet
Dentro da VPC vamos criar uma subnet:
MinhaSubnet:
  Type: AWS::EC2::Subnet
  Properties:
    VpcId: !Ref MinhaVPC
    CidrBlock: 10.0.1.0/24
    Tags:
      - Key: Name
        Value: SubnetAula

Agora:
VPC
10.0.0.0/16
    |
    +-- Subnet
        10.0.1.0/24

14. O que é !Ref?
No código:
VpcId: !Ref MinhaVPC
!Ref referencia outro recurso.
Nesse caso:
MinhaSubnet
      |
      +-- precisa da MinhaVPC

O CloudFormation consegue entender essa dependência.

15. Colocando a EC2 na Subnet
Agora vamos alterar a EC2:
MinhaInstancia:
  Type: AWS::EC2::Instance
  Properties:
    ImageId: ami-xxxxxxxx
    InstanceType: t3.micro
    SubnetId: !Ref MinhaSubnet
    SecurityGroupIds:
      - !GetAtt MeuSecurityGroup.GroupId

Agora temos:

VPC
 |
 +-- Subnet
      |
      +-- EC2
           |
           +-- Security Group

16. Template completo da aula
Neste momento, nosso template.yaml pode ficar assim:
AWSTemplateFormatVersion: '2026-09-09'
Description: Aula pratica de AWS CloudFormation
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
          Value: AulaCloudFormation

  MinhaSubnet:
    Type: AWS::EC2::Subnet
    Properties:
      VpcId: !Ref MinhaVPC
      CidrBlock: 10.0.1.0/24
      Tags:
        - Key: Name
          Value: SubnetAula

  MeuSecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: Security Group da aula
      VpcId: !Ref MinhaVPC
      SecurityGroupIngress:
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
          Value: EC2-AulaCloudFormation

Substitua ami-xxxxxxxx por uma AMI válida para a região escolhida.

17. Parameters

Observe:
Parameters:
  TipoInstancia:
    Type: String
    Default: t3.micro
    AllowedValues:
      - t3.micro
      - t3.small
      - t3.medium

Isso permite escolher o tamanho da EC2 sem alterar o código.
Depois utilizamos:
InstanceType: !Ref TipoInstancia
Fluxo:

Usuário
   |
   v
TipoInstancia = t3.micro
   |
   v
CloudFormation
   |
   v
EC2 t3.micro

18. Adicionando UserData
Agora vamos configurar automaticamente o servidor.
Podemos adicionar:
UserData:
  Fn::Base64: |
    #!/bin/bash
    echo "Servidor criado pelo CloudFormation" > /tmp/cloudformation.txt

A EC2 executará o script na inicialização.
Em uma aplicação real, o UserData poderia instalar dependências, configurar serviços e iniciar a aplicação.

19. Instalando um servidor web

Por exemplo, em uma distribuição compatível com o script:
UserData:
  Fn::Base64: |
    #!/bin/bash
    yum update -y
    yum install -y nginx
    systemctl enable nginx
    systemctl start nginx

O fluxo será:

CloudFormation
      |
      v
Cria EC2
      |
      v
EC2 inicia
      |
      v
UserData
      |
      +-- Atualiza sistema
      +-- Instala Nginx
      +-- Inicia Nginx

O comando de instalação depende da distribuição Linux utilizada. Sempre ajuste o UserData ao sistema operacional da AMI escolhida.

20. Outputs
Vamos adicionar:
Outputs:
  InstanceId:
    Description: ID da instancia EC2
    Value: !Ref MinhaInstancia

Depois da criação da Stack, teremos algo semelhante a:

Outputs
InstanceId
i-0123456789abcdef

21. Criando a Stack pelo Console
Agora vamos executar a aula.
Passo 1
Abra:
AWS Console
Depois:
CloudFormation
Passo 2
Selecione:
Create stack
Passo 3
Escolha:
With new resources (standard)

Passo 4
Escolha:
Upload a template file

Passo 5
Selecione:
template.yaml

Passo 6
Defina o nome:
AulaCloudFormation

Passo 7
No parâmetro:
TipoInstancia
escolha:
t3.micro

Passo 8
Avance até a criação da Stack.

22. Acompanhando a criação

O CloudFormation apresentará eventos semelhantes a:

CREATE_IN_PROGRESS
CREATE_COMPLETE

Podemos acompanhar:
MinhaVPC
    CREATE_COMPLETE

MinhaSubnet
    CREATE_COMPLETE

MeuSecurityGroup
    CREATE_COMPLETE

MinhaInstancia
    CREATE_COMPLETE

Quando todos estiverem como:
CREATE_COMPLETE

a Stack foi criada com sucesso.

23. Verificando a EC2

Agora abra:
EC2
   |
   +-- Instances

Você deverá encontrar uma instância com a tag:

EC2-AulaCloudFormation
A partir daí podemos verificar:
Instance ID;
estado;
tipo;
VPC;
subnet;
Security Group;
endereço IP;
AMI.

24. Verificando a VPC
Abra:
VPC
Procure a VPC:
AulaCloudFormation
Ela deverá possuir:
CIDR:
10.0.0.0/16

25. Verificando a Subnet
Na área de VPC:
Subnets
procure:
SubnetAula
CIDR:
10.0.1.0/24

26. Entendendo a ordem de criação

O CloudFormation identifica as dependências.
Temos:

MinhaVPC
   |
   +---- MinhaSubnet
   |
   +---- MeuSecurityGroup
              |
              v
        MinhaInstancia

A EC2 depende da Subnet e do Security Group.
A Subnet e o Security Group dependem da VPC.
Portanto, o CloudFormation consegue organizar a criação.

27. Criando a Stack pela AWS CLI
Depois de entender o console, podemos utilizar a CLI.
Primeiro valide sua configuração:
aws sts get-caller-identity

Depois:
aws cloudformation create-stack   --stack-name AulaCloudFormation   --template-body file://template.yaml   --parameters ParameterKey=TipoInstancia,ParameterValue=t3.micro

28. Consultando a Stack

Podemos executar:
aws cloudformation describe-stacks   --stack-name AulaCloudFormation

29. Consultando os eventos
Para acompanhar os recursos:
aws cloudformation describe-stack-events   --stack-name AulaCloudFormation

30. Atualizando a Stack
Imagine que queremos mudar:
t3.micro
para:
t3.small

Podemos atualizar a Stack.
aws cloudformation update-stack   --stack-name AulaCloudFormation   --template-body file://template.yaml   --parameters ParameterKey=TipoInstancia,ParameterValue=t3.small

O CloudFormation identificará a alteração.

31. Excluindo a Stack

Quando a aula terminar, podemos excluir a Stack:
aws cloudformation delete-stack   --stack-name AulaCloudFormation
O CloudFormation excluirá os recursos gerenciados pela Stack, conforme as políticas de retenção e as características de cada recurso.
Antes de excluir, verifique se não existe nenhum recurso ou dado que você deseja preservar.
