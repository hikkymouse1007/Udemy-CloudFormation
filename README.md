# Udemy-CloudFormation
https://www.udemy.com/course/aws-cloudformation-master-class/learn/lecture/8138960?start=0#overview

<<<<<<< HEAD
ユーザガイド
https://docs.aws.amazon.com/cloudformation/index.html

CLI_Doc
https://docs.aws.amazon.com/ja_jp/cli/latest/reference/cloudformation/create-stack.html

# 1.introduction
CloudFormationの基本
- yamlがS3にアップロードされると、CFは S3を参照してスタックを展開する
- スタックの更新は、"新しいyamlをアップロードする”ことで実現する。
- スタックは一意な名前で識別される。
- スタック削除時には、yamlに基づくリソースを全て削除する。

テンプレートの選択>テンプレートファイルの選択からyamlファイルを作成

```
// 
Resources:
  MyInstance:
    Type: AWS::EC2::Instance
    Properties:
      AvailabilityZone: us-east-1a　 // アベイラビリティーゾーン(a, b, c)も指定すること
      ImageId: ami-04bf6dcdc9ab498ca
      InstanceType: t2.micro
```

スタックは編集できないので、
"スタック>スタック名>更新する"から
新しいテンプレートを追加する。

Refについて
https://docs.aws.amazon.com/ja_jp/AWSCloudFormation/latest

組み込み関数
https://qiita.com/bisque33/items/51147a65c2ba40417d01

```
Resources:
  MyInstance:
    Type: AWS::EC2::Instance
    Properties:
      AvailabilityZone: us-east-1a
      ImageId: ami-04bf6dcdc9ab498ca
      InstanceType: t2.micro
      SecurityGroups:
        - !Ref SSHSecurityGroup //以下のリソースを参照する
        - !Ref ServerSecurityGroup

  # an elastic IP for our instance
  MyEIP:
    Type: AWS::EC2::EIP
    Properties:
      InstanceId: !Ref MyInstance

  # our EC2 security group
  SSHSecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: Enable SSH access via port 22
      SecurityGroupIngress:
      - CidrIp: 0.0.0.0/0
        FromPort: 22
        IpProtocol: tcp
        ToPort: 22

  # our second EC2 security group
  ServerSecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: allow connections from specified CIDR ranges
      SecurityGroupIngress:
      - IpProtocol: tcp
        FromPort: 80
        ToPort: 80
        CidrIp: 0.0.0.0/0
      - IpProtocol: tcp
        FromPort: 22
        ToPort: 22
        CidrIp: 192.168.1.1/32

```

EC2の設定に変更があると、(SG, EIPのアタッチなど)既存のEC2は更新時に削除され、再度作成される
<img width="1200" alt="スクリーンショット 2020-11-23 11 54 45" src="https://user-images.githubusercontent.com/54907440/99925954-55e57980-2d83-11eb-82d9-735c8ced8b39.png">
SG
<img width="1142" alt="スクリーンショット 2020-11-23 11 55 02" src="https://user-images.githubusercontent.com/54907440/99925955-567e1000-2d83-11eb-834f-036e9e7566d8.png">
EIP
<img width="1188" alt="スクリーンショット 2020-11-23 11 55 20" src="https://user-images.githubusercontent.com/54907440/99925957-5716a680-2d83-11eb-9033-6fa4e02cfeb6.png">
登録したテンプレートのyamlファイル
<img width="1413" alt="スクリーンショット 2020-11-23 11 56 36" src="https://user-images.githubusercontent.com/54907440/99925959-5847d380-2d83-11eb-9288-61387b767283.png">

スタックを削除すると自動で全て削除される
<img width="996" alt="スクリーンショット 2020-11-23 11 57 25" src="https://user-images.githubusercontent.com/54907440/99925960-5847d380-2d83-11eb-85f5-4ccd61a0d60f.png">
<img width="1011" alt="スクリーンショット 2020-11-23 11 57 47" src="https://user-images.githubusercontent.com/54907440/99925962-58e06a00-2d83-11eb-98b6-1b0a9139dbb1.png">
<img width="1197" alt="スクリーンショット 2020-11-23 11 58 18" src="https://user-images.githubusercontent.com/54907440/99925963-59790080-2d83-11eb-8c17-1ed5b9d23e50.png">

# 2. hands-on
##  S3 
ユーザガイド
https://docs.aws.amazon.com/ja_jp/AWSCloudFormation/latest/UserGuide/aws-properties-s3-bucket.html

S3のバケット名は、
"stack_name-resource_name-unique_id"の形式

"mys3stack-hikkys3bucket-xxxxxx"
<img width="1153" alt="スクリーンショット 2020-11-23 12 45 14" src="https://user-images.githubusercontent.com/54907440/99927761-e921ad80-2d89-11eb-859e-896640adfdc5.png">


```
---
Resources:
  HikkyS3Bucket: //resource_name
    Type: "AWS::S3::Bucket"
    Properties: {}

```
アクセスコントロールの設定

```
---
Resources:
  HikkyS3Bucket:
    Type: "AWS::S3::Bucket"
    Properties:
      AccessControl: PublicRead

```

バケット名の更新
# 注意
既存のスタックの更新でバケットの名前を変更すると名前変更前のバケットは**削除される**ので注意

```
---
Resources:
  HikkyS3Bucket:
    Type: "AWS::S3::Bucket"
    Properties:
      AccessControl: PublicRead
      BucketName: "hikky-brand-new-bucket" //新しいバケット名

```
置換がTrueの場合、旧バケットの削除が実行され、新しいバケットが作成される。
<img width="825" alt="スクリーンショット 2020-11-23 12 57 01" src="https://user-images.githubusercontent.com/54907440/99928336-ede76100-2d8b-11eb-8483-55d8d330e6c7.png">
<img width="832" alt="スクリーンショット 2020-11-23 12 56 14" src="https://user-images.githubusercontent.com/54907440/99928334-ecb63400-2d8b-11eb-9b69-951d84c4fd1f.png">

<img width="692" alt="スクリーンショット 2020-11-23 12 58 28" src="https://user-images.githubusercontent.com/54907440/99928337-ee7ff780-2d8b-11eb-8871-3a5a0efa2bc3.png">


## CloudFormation Template
デザイナーでテンプレートを作成

それぞれの点には役割があるので注意
![スクリーンショット 2020-11-24 23 17 37](https://user-images.githubusercontent.com/54907440/100106674-2d0cd380-2eac-11eb-87ef-2ec2d9259572.png)

既存のyamlからインフラを出力することも可能
![スクリーンショット 2020-11-24 23 22 27](https://user-images.githubusercontent.com/54907440/100106680-30a05a80-2eac-11eb-8ad9-18af98dad622.png)

# CloudFormation Parameter
パラメータとは
- テンプレートのリユース
- タイプを利用し、エラーを避ける
- パラメータの変更にはテンプレートの再アップロードは不要

# 2-ハンズオン

```
Parameters:
  SecurityGroupDescription:
    Description: Security Group Description (Simple parameter)
    Type: String
  SecurityGroupPort:
    Description: Simple Description of a Number Parameter, with MinValue and MaxValue
    Type: Number
    MinValue: 1150
    MaxValue: 65535
  InstanceType:
    Description: WebServer EC2 instance type (has default, AllowedValues)
    Type: String
    Default: t2.small
    AllowedValues:
      - t1.micro
      - t2.nano
      - t2.micro
      - t2.small
    ConstraintDescription: must be a valid EC2 instance type.
  DBPwd:
    NoEcho: true //パスワードの秘匿化
    Description: The database admin account password (won't be echoed)
    Type: String
  KeyName: //SSHキー
    Description: Name of an existing EC2 KeyPair to enable SSH access to the instances. Linked to AWS Parameter
    Type: AWS::EC2::KeyPair::KeyName
    ConstraintDescription: must be the name of an existing EC2 KeyPair.
  SecurityGroupIngressCIDR:
    Description: The IP address range that can be used to communicate to the EC2 instances
    Type: String
    MinLength: '9'
    MaxLength: '18'
    Default: 0.0.0.0/0
    AllowedPattern: (\d{1,3})\.(\d{1,3})\.(\d{1,3})\.(\d{1,3})/(\d{1,2}) //RegExp
    ConstraintDescription: must be a valid IP CIDR range of the form x.x.x.x/x.
  MyVPC:
    Description: VPC to operate in
    Type: AWS::EC2::VPC::Id
  MySubnetIDs:
    Description: Subnet IDs that is a List of Subnet Id
    Type: "List<AWS::EC2::Subnet::Id>"
  DbSubnetIpBlocks:
    Description: "Comma-delimited list of three CIDR blocks"
    Type: CommaDelimitedList
    Default: "10.0.48.0/24, 10.0.112.0/24, 10.0.176.0/24"

Resources:
  MyEC2Instance:
    Type: "AWS::EC2::Instance"
    Properties:
      #we reference the InstanceType parameter
      InstanceType: !Ref InstanceType
      KeyName: !Ref KeyName
      ImageId: "ami-a4c7edb2"
      # here we reference an internal CloudFormation resource
      SubnetId: !Ref DbSubnet1

  MySecurityGroup:
    Type: "AWS::EC2::SecurityGroup"
    Properties:
      GroupDescription: !Ref SecurityGroupDescription
      SecurityGroupIngress:
        - CidrIp: !Ref SecurityGroupIngressCIDR
          FromPort: !Ref SecurityGroupPort
          ToPort: !Ref SecurityGroupPort
          IpProtocol: tcp
      VpcId: !Ref MyVPC

  DbSubnet1:
    Type: AWS::EC2::Subnet
    Properties:
      VpcId: !Ref MyVPC
      # the select function allows us to select across a list
      CidrBlock: !Select [0, !Ref DbSubnetIpBlocks]
  DbSubnet2:
    Type: AWS::EC2::Subnet
    Properties:
      VpcId: !Ref MyVPC
      # the select function allows us to select across a list
      CidrBlock: !Select [1, !Ref DbSubnetIpBlocks]
  DbSubnet3:
    Type: AWS::EC2::Subnet
    Properties:
      VpcId: !Ref MyVPC
      # the select function allows us to select across a list
      CidrBlock: !Select [2, !Ref DbSubnetIpBlocks]

```

パラメータで指定した値はテンプレで反映される

![スクリーンショット 2020-11-26 0 02 44](https://user-images.githubusercontent.com/54907440/100245516-7415ca00-2f7b-11eb-9a37-79233d09126c.png)

##　参照するには
Fn:Ref関数を使う
YAMLでは!Refを使う
![スクリーンショット 2020-11-26 0 15 41](https://user-images.githubusercontent.com/54907440/100246580-ac69d800-2f7c-11eb-868f-4a5c809cf46b.png)

# 4-Resources
```
---
Resources:
  MyInstance:
    # http://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-properties-ec2-instance.html
    Type: AWS::EC2::Instance
    Properties:
      AvailabilityZone: us-east-1a
      ImageId: ami-a4c7edb2
      InstanceType: t2.micro
      SecurityGroups:
        - !Ref SSHSecurityGroup
        - !Ref ServerSecurityGroup

  MyEIP:
    # http://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-properties-ec2-eip.html
    Type: AWS::EC2::EIP
    Properties:
      InstanceId: !Ref MyInstance

  SSHSecurityGroup:
    # http://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-properties-ec2-security-group.html
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: Enable SSH access via port 22
      SecurityGroupIngress:
      - CidrIp: 0.0.0.0/0
        FromPort: 22
        IpProtocol: tcp
        ToPort: 22

  ServerSecurityGroup:
    # http://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-properties-ec2-security-group.html
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: allow connections from specified CIDR ranges
      SecurityGroupIngress:
      - IpProtocol: tcp
        FromPort: 80
        ToPort: 80
        CidrIp: 0.0.0.0/0
      - IpProtocol: tcp
        FromPort: 22
        ToPort: 22
        CidrIp: 192.168.1.1/32

```
リソースは以下のように、参照先の順から作成される
左から) SG > EC2 > EIP
![スクリーンショット 2020-11-26 0 25 15](https://user-images.githubusercontent.com/54907440/100247713-f99a7980-2f7d-11eb-8295-f4c1087c988b.png)

## Resourceのオプショナルアトリビュート
- DependOn:リソースの作成順の指定(dockerと一緒)
- DeletionPolicy: 削除保護設定。RDSなど
- CreationPolicy: のちの章にて
- Metadata:　同じく

# 5-Mappings
- Fn::FindMAp
- !FindInMap [MapName, TopLevelKey, SecondLevelKey]

```
Parameters:
  EnvironmentName:
    Description: Environment Name
    Type: String
    AllowedValues: [development, production]
    ConstraintDescription: must be development or production

Mappings:
  AWSRegionArch2AMI:
    us-east-1:
      HVM64: ami-6869aa05
    us-west-2:
      HVM64: ami-7172b611
    us-west-1:
      HVM64: ami-31490d51
    eu-west-1:
      HVM64: ami-f9dd458a
    eu-central-1:
      HVM64: ami-ea26ce85
    ap-northeast-1:
      HVM64: ami-374db956
    ap-northeast-2:
      HVM64: ami-2b408b45
    ap-southeast-1:
      HVM64: ami-a59b49c6
    ap-southeast-2:
      HVM64: ami-dc361ebf
    ap-south-1:
      HVM64: ami-ffbdd790
    us-east-2:
      HVM64: ami-f6035893
    sa-east-1:
      HVM64: ami-6dd04501
    cn-north-1:
      HVM64: ami-8e6aa0e3
  EnvironmentToInstanceType:
    development:
      instanceType: t2.micro
    # we want a bigger instance type in production
    production:
      instanceType: t2.small

Resources:
  EC2Instance:
    Type: AWS::EC2::Instance
    Properties:
      InstanceType: !FindInMap [EnvironmentToInstanceType, !Ref 'EnvironmentName', instanceType]
      # Note we use the pseudo parameter AWS::Region
      ImageId: !FindInMap [AWSRegionArch2AMI, !Ref 'AWS::Region', HVM64]
```

マッピングで指定したパラメータは選択肢に反映される
![スクリーンショット 2020-11-26 0 42 56](https://user-images.githubusercontent.com/54907440/100250223-c73e4b80-2f80-11eb-9dd4-df31029c7978.png)

AMIはマッピングの情報から自分のいるリージョンを自動で判断し、選択してくれる
![スクリーンショット 2020-11-26 0 45 40](https://user-images.githubusercontent.com/54907440/100250231-cad1d280-2f80-11eb-970f-746eb8fc5556.png)

# 6-Output
 ネットワークの設定に便利な設定
 ```
 // example.yaml
 Outputs:
  Logical ID:
    Description: Information about the value
    Value: Value to return 
    Export:
      Name: Value to export
 ```

ssh-output.yaml
 ```
 Resources:
  # here we define a SSH security group that will be used in the entire company
  MyCompanyWideSSHSecurityGroup:
    # http://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-properties-ec2-security-group.html
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: Enable SSH access via port 22
      SecurityGroupIngress:
        # we have a lot of rules because it's a perfect security group
        # finance team network
      - CidrIp: 10.0.48.0/24
        FromPort: 22
        IpProtocol: tcp
        ToPort: 22
        # marketing team network
      - CidrIp: 10.0.112.0/24
        FromPort: 22
        IpProtocol: tcp
        ToPort: 22
        # application team support network
      - CidrIp: 10.0.176.0/24
        FromPort: 22
        IpProtocol: tcp
        ToPort: 22

Outputs:
  StackSSHSecurityGroup:
    Description: The SSH Security Group for our Company
    Value: !Ref MyCompanyWideSSHSecurityGroup
    Export:
      Name: SSHSecurityGroup

 ```

![スクリーンショット 2020-11-28 23 27 39](https://user-images.githubusercontent.com/54907440/100517910-9012a900-31d1-11eb-9dd5-88205032319d.png)
 アウトプット名がCFのコンソールに表示される
![スクリーンショット 2020-11-28 23 27 57](https://user-images.githubusercontent.com/54907440/100517912-92750300-31d1-11eb-8690-53bd590f1283.png)

### Outputのimport

```
Resources:
  MySecureInstance:
    # http://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-properties-ec2-instance.html
    Type: AWS::EC2::Instance
    Properties:
      AvailabilityZone: us-east-1a
      ImageId: ami-a4c7edb2
      InstanceType: t2.micro
      SecurityGroups:
        # we reference the output here, using the Fn::ImportValue function
        - !ImportValue SSHSecurityGroup //アウトプットのimport

```

作成したアプトプットのSGが適用されている
![スクリーンショット 2020-11-28 23 27 57](https://user-images.githubusercontent.com/54907440/100520315-847aae80-31e0-11eb-888f-3e86f70847b9.png)

### おまけ
SSMパラメータで適切なAMIを取得する実装

```
Parameters:
  Ec2ImageId:
    Type: AWS::SSM::Parameter::Value<String>
    Default: /aws/service/ami-amazon-linux-latest/amzn2-ami-hvm-x86_64-gp2
  Ec2InstanceType:
    Type: String
    Default: t2.micro

Resources:
  MySecureInstance:
    # http://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-properties-ec2-instance.html
    Type: AWS::EC2::Instance
    Properties:
      AvailabilityZone: ap-northeast-1a
      ImageId: !Ref Ec2ImageId
      InstanceType: !Ref Ec2InstanceType
      SecurityGroups:
        # we reference the output here, using the Fn::ImportValue function
        - !ImportValue SSHSecurityGroup

```

SSMパラメータで適切なAMIを取得する
https://dev.classmethod.jp/articles/get-the-latest-amazon-linux-2-ami-id-with-cloudformation/


## 7-Conditions
- 環境の設定(dev/test/prod)
- リージョンの設定
などに使える

環境変数で作成されるリソースを分けることも可能
![スクリーンショット 2020-11-29 1 38 16](https://user-images.githubusercontent.com/54907440/100521055-a413d600-31e4-11eb-89db-1fedcb612255.png)
![スクリーンショット 2020-11-29 1 51 35](https://user-images.githubusercontent.com/54907440/100521194-70857b80-31e5-11eb-8959-f6bd7e044dfa.png)
![スクリーンショット 2020-11-29 1 45 25](https://user-images.githubusercontent.com/54907440/100521049-99594100-31e4-11eb-87f6-065701cb6d9b.png)

condition function
https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/intrinsic-function-reference-conditions.html

## 8-meta-data
yamlに基づいてインターフェースをカスタマイズすることができる

```
---
Parameters:
  KeyName:
    Description: Name of an existing EC2 key pair for SSH access to the EC2 instance.
    Type: AWS::EC2::KeyPair::KeyName
  InstanceType:
    Description: EC2 instance type.
    Type: String
    Default: t2.micro
    AllowedValues:
    - t2.micro
    - t2.small
    - t2.medium
    - m3.medium
    - m3.large
    - m3.xlarge
    - m3.2xlarge
  SSHLocation:
    Description: The IP address range that can SSH to the EC2 instance.
    Type: String
    MinLength: '9'
    MaxLength: '18'
    Default: 0.0.0.0/0
    AllowedPattern: "(\\d{1,3})\\.(\\d{1,3})\\.(\\d{1,3})\\.(\\d{1,3})/(\\d{1,2})"
    ConstraintDescription: Must be a valid IP CIDR range of the form x.x.x.x/x.
  VPCID:
    Description: VPC to operate in
    Type: AWS::EC2::VPC::Id
  SubnetID:
    Description: Subnet ID
    Type: AWS::EC2::Subnet::Id
  SecurityGroupID:
    Description: Security Group
    Type: AWS::EC2::SecurityGroup::Id

Resources:
  MyEC2Instance:
    Type: "AWS::EC2::Instance"
    Properties:
      AvailabilityZone: us-east-1a
      ImageId: ami-a4c7edb2
      InstanceType: !Ref InstanceType
      SecurityGroups:
        - !Ref SecurityGroupID
      SubnetID: !Ref SubnetID

Metadata:
  AWS::CloudFormation::Interface:
    ParameterGroups:
      - Label:
          default: "Network Configuration"
        Parameters:
          - VPCID
          - SubnetID
          - SecurityGroupID
      - Label:
          default: "Amazon EC2 Configuration"
        Parameters:
          - InstanceType
          - KeyName
    ParameterLabels:
      VPCID:
        default: "Which VPC should this be deployed to?"
```

![スクリーンショット 2020-11-29 2 05 56](https://user-images.githubusercontent.com/54907440/100521520-9dd32900-31e7-11eb-9af5-c69acff5b892.png)

## 9-EC2 User Data
Linuxコマンドを実行させる

```
UserData:
        Fn::Base64: |
           #!/bin/bash
           yum update -y
           yum install -y httpd24 php56 mysql55-server php56-mysqlnd
           service httpd start
           chkconfig httpd on
           groupadd www
           usermod -a -G www ec2-user
           chown -R root:www /var/www
           chmod 2775 /var/www
           find /var/www -type d -exec chmod 2775 {} +
           find /var/www -type f -exec chmod 0664 {} +
           echo "<?php phpinfo(); ?>" > /var/www/html/phpinfo.php
```

###　注意
本セクションはLinux2ではなくLinux1を使う
https://docs.aws.amazon.com/ja_jp/AWSEC2/latest/UserGuide/amazon-linux-ami-basics.html#amazon-linux-image-id

```
You have to make sure that you are using the right ec2 image id for your region, if you see in the template this part: "ImageId: ami-a4c7edb2", well for your region the image id is different, you can go and create manually the ec2 instance, and be careful to see image id used for Amazon Linux AMI (this is right on step 1) and not Amazon Linux 2 AMI, as Linux 2 does not support these packages, or use a different repo to pull packages.

Ec2ImageId:
    Type: AWS::SSM::Parameter::Value<String>
    Default: /aws/service/ami-amazon-linux-latest/amzn-ami-hvm-x86_64-gp2

```

## CloudFormation Init
パッケージのバージョンを指定しない場合はlatestがデフォルトで
インストールされる
- !Sub Function(substitute:代替)
環境変数のアサイン

Cfn-hup
https://www.udemy.com/course/aws-cloudformation-master-class/learn/lecture/8162188#questions/10622236
メタデータの変更を15分ごとに監視・更新してくれる

>AWS::CloudFormation::Init:の内容に変更があった場合、UpdateStack時にcfn-hupサービスが検知して、cfn-auto-reloader.confの内容(cfn-init)を実行してくれる...ハズなんですが、実行してくれる時としてくれない時があって良くわかりませんでした
>2018-11-02追記
/var/log/cfn-hup.logを眺めていたら、どうやら変更監視は15分間隔で実施されているようでした。
UpdateStack後に次回のチェックを待つとちゃんと実行されました。

https://qiita.com/algi_nao/items/8898afed7ce723ea7fbb

```
User Data vs CloudFormation::Init vs Helper Scripts
In summary, what's the difference between EC2 User Data, CloudFormation::Init, and CF Helper scripts?

=============================
- User-data is an imperative way to provision/bootstrap the EC2 instance using SHELL syntax .

- AWS::CloudFormation::Init is a declarative way to provision/bootstrap the EC2 instance using YAML or JSON syntax.

- AWS::CloudFormation::Init is useless if it is NOT triggered by UserData.

=> Triggering AWS::CloudFormation::Init inside UserData is done by one of helper scripts (cfn-init).
```

## CloudFormation Drift
GUIからインフラ変更を加えた場合に、CFのスタックから
Drift(変更した差分)を確認することができる。
差分をyamlに反映させることで整合性を保ったり、テンプレートの
バージョン管理を実現することができる。
![スクリーンショット 2020-11-29 22 24 24](https://user-images.githubusercontent.com/54907440/100543138-bef05400-3291-11eb-8d0a-bd249b83cf87.png)


全てのリソースを網羅しているわけではないので注意。
https://docs.aws.amazon.com/ja_jp/AWSCloudFormation/latest/UserGuide/detect-drift-stack.html

# 11-Nested-Stacks
複数のStackを他のStackから呼び出す
1. S3を作り、CFのyamlを起き、URLをコピーする
![スクリーンショット 2020-12-01 22 11 18](https://user-images.githubusercontent.com/54907440/100745251-571d4300-3422-11eb-9df8-f94b91a06f2a.png)

2. StackのURLをyamlから読み込む

```
#ec2.yaml

Parameters:
  VPCId:
    Description: VPC to create the security group and EC2 instance into
    Type: AWS::EC2::VPC::Id

Mappings:
  AWSRegionArch2AMI:
    us-east-1:
      HVM64: ami-6869aa05
    us-west-2:
      HVM64: ami-7172b611
    us-west-1:
      HVM64: ami-31490d51
    eu-west-1:
      HVM64: ami-f9dd458a
    eu-central-1:
      HVM64: ami-ea26ce85
    ap-northeast-1:
      HVM64: ami-374db956
    ap-northeast-2:
      HVM64: ami-2b408b45
    ap-southeast-1:
      HVM64: ami-a59b49c6
    ap-southeast-2:
      HVM64: ami-dc361ebf
    ap-south-1:
      HVM64: ami-ffbdd790
    us-east-2:
      HVM64: ami-f6035893
    sa-east-1:
      HVM64: ami-6dd04501
    cn-north-1:
      HVM64: ami-8e6aa0e3

Resources:

  SSHSecurityGroupStack:
    Type: AWS::CloudFormation::Stack　// Nested-Stack　S3からyamlを呼び出す
    Properties:
      TemplateURL: https://mashimo-cloudformation-stack.s3-ap-northeast-1.amazonaws.com/ssh-sg.yaml //S3のyamlURL
      Parameters:
        ApplicationName: !Ref AWS::StackName
        VPCId: !Ref VPCId
      TimeoutInMinutes: 5


  EC2Instance:
    Type: AWS::EC2::Instance
    Properties:
      InstanceType: t2.micro
      # Note we use the pseudo parameter AWS::Region
      ImageId: !FindInMap [AWSRegionArch2AMI, !Ref 'AWS::Region', HVM64]
      AvailabilityZone: !Sub ${AWS::Region}a
      SecurityGroupIds:
        - !GetAtt SSHSecurityGroupStack.Outputs.SSHGroupId //S3にあるスタックを取得

```
SSHSecurityGroupStackはS3のスタックが反映されている。Outputs.SSHGroupIdは下のS3に上げたスタック
の内容を読み込む。

S3に上げたスタック
```
// ssh-sg.yaml
Parameters:
  ApplicationName:
    Description: The application name
    Type: String
  VPCId:
    Description: VPC to create the security group into
    Type: AWS::EC2::VPC::Id
  
Resources:
  SSHSecurityGroup:
    Type: "AWS::EC2::SecurityGroup"
    Properties:
      GroupDescription: !Sub Security group for ${ApplicationName}
      SecurityGroupIngress:
        - CidrIp: "10.0.0.0/25"
          FromPort: 22
          ToPort: 22
          IpProtocol: tcp
          Description: SSH for Engineering department
        - CidrIp: "192.168.0.0/25"
          FromPort: 22
          ToPort: 22
          IpProtocol: tcp
          Description: SSH for HR department
      VpcId: !Ref VPCId

Outputs:
  SSHGroupId:
    Value: !Ref SSHSecurityGroup
    Description: Id for the SSH Security Group
```

3. スタックを作成する
![スクリーンショット 2020-12-01 22 27 32](https://user-images.githubusercontent.com/54907440/100746968-c72cc880-3424-11eb-83a4-4bd146c83b85.png)
![スクリーンショット 2020-12-01 22 29 34](https://user-images.githubusercontent.com/54907440/100746981-cac04f80-3424-11eb-83d9-0f6f20e09970.png)

4. スタックの更新
- 同名のyamlをS3に上げる(yamlが更新される)
- rootスタックを更新する(同一ファイルの上げ直し)
![スクリーンショット 2020-12-01 22 39 48](https://user-images.githubusercontent.com/54907440/100748194-60101380-3426-11eb-9686-6e89e5ca1e0d.png)
![スクリーンショット 2020-12-01 22 40 52](https://user-images.githubusercontent.com/54907440/100748205-630b0400-3426-11eb-81c6-fcc573d31ab2.png)

ネストされたスタックは、CF上では操作しない。
S3でyamlを更新→CFでrootスタックを更新
の流れで変更を加える。


5. ネストされたスタックの削除
必ずrootスタックのみを削除する。
![スクリーンショット 2020-12-01 22 44 43](https://user-images.githubusercontent.com/54907440/100748726-078d4600-3427-11eb-8f90-d6341b10438d.png)
![スクリーンショット 2020-12-01 22 45 58](https://user-images.githubusercontent.com/54907440/100748734-09efa000-3427-11eb-9dc6-2d1182b3106a.png)

# 12-Advanced
## AWS CLI
1. CLIのダウンロード
2. アクセスキーの発行
マイセキュリティ資格情報>セキュリティ認証情報
3. CLIの登録
pip install awscli --upgrade --user
aws configure --profile cf-course

```
// 0-parameters.json
[
  {
    "ParameterKey": "InstanceType",
    "ParameterValue": "t2.micro"
  },
  {
    "ParameterKey": "KeyName",
    "ParameterValue": "udemy" //EC2インスタンスの自分のリージョンに存在するSSHキーの名前を指定する
  },
  {
    "ParameterKey": "SSHLocation",
    "ParameterValue": "0.0.0.0/0"
  }
]

```

上のjsonは0-sample-template.yamlの
Parameters:のkey-valueを配列として
上から順番に定義している。

```
// 0-sample-template.yaml
Metadata:
  License: Apache-2.0
AWSTemplateFormatVersion: '2010-09-09'
Description: 'AWS CloudFormation Sample Template Sample template EIP_With_Association:
  This template shows how to associate an Elastic IP address with an Amazon EC2 instance
  - you can use this same technique to associate an EC2 instance with an Elastic IP
  Address that is not created inside the template by replacing the EIP reference in
  the AWS::EC2::EIPAssoication resource type with the IP address of the external EIP.
  **WARNING** This template creates an Amazon EC2 instance and an Elastic IP Address.
  You will be billed for the AWS resources used if you create a stack from this template.'
Parameters:
  InstanceType:
    Description: WebServer EC2 instance type
    Type: String
    Default: t2.small
    AllowedValues: [t1.micro, t2.nano, t2.micro, t2.small, t2.medium, t2.large, m1.small,
      m1.medium, m1.large, m1.xlarge, m2.xlarge, m2.2xlarge, m2.4xlarge, m3.medium,
      m3.large, m3.xlarge, m3.2xlarge, m4.large, m4.xlarge, m4.2xlarge, m4.4xlarge,
      m4.10xlarge, c1.medium, c1.xlarge, c3.large, c3.xlarge, c3.2xlarge, c3.4xlarge,
      c3.8xlarge, c4.large, c4.xlarge, c4.2xlarge, c4.4xlarge, c4.8xlarge, g2.2xlarge,
      g2.8xlarge, r3.large, r3.xlarge, r3.2xlarge, r3.4xlarge, r3.8xlarge, i2.xlarge,
      i2.2xlarge, i2.4xlarge, i2.8xlarge, d2.xlarge, d2.2xlarge, d2.4xlarge, d2.8xlarge,
      hi1.4xlarge, hs1.8xlarge, cr1.8xlarge, cc2.8xlarge, cg1.4xlarge]
    ConstraintDescription: must be a valid EC2 instance type.
  KeyName:
    Description: Name of an existing EC2 KeyPair to enable SSH access to the instances
    Type: AWS::EC2::KeyPair::KeyName
    ConstraintDescription: must be the name of an existing EC2 KeyPair.
  SSHLocation:
    Description: The IP address range that can be used to SSH to the EC2 instances
    Type: String
    MinLength: '9'
    MaxLength: '18'
    Default: 0.0.0.0/0
    AllowedPattern: (\d{1,3})\.(\d{1,3})\.(\d{1,3})\.(\d{1,3})/(\d{1,2})
    ConstraintDescription: must be a valid IP CIDR range of the form x.x.x.x/x.
Mappings:
  AWSInstanceType2Arch:
    t1.micro:
      Arch: PV64
    t2.nano:
      Arch: HVM64
    t2.micro:
      Arch: HVM64
    t2.small:
      Arch: HVM64
    t2.medium:
      Arch: HVM64
    t2.large:
      Arch: HVM64
    m1.small:
      Arch: PV64
    m1.medium:
      Arch: PV64
    m1.large:
      Arch: PV64
    m1.xlarge:
      Arch: PV64
    m2.xlarge:
      Arch: PV64
    m2.2xlarge:
      Arch: PV64
    m2.4xlarge:
      Arch: PV64
    m3.medium:
      Arch: HVM64
    m3.large:
      Arch: HVM64
    m3.xlarge:
      Arch: HVM64
    m3.2xlarge:
      Arch: HVM64
    m4.large:
      Arch: HVM64
    m4.xlarge:
      Arch: HVM64
    m4.2xlarge:
      Arch: HVM64
    m4.4xlarge:
      Arch: HVM64
    m4.10xlarge:
      Arch: HVM64
    c1.medium:
      Arch: PV64
    c1.xlarge:
      Arch: PV64
    c3.large:
      Arch: HVM64
    c3.xlarge:
      Arch: HVM64
    c3.2xlarge:
      Arch: HVM64
    c3.4xlarge:
      Arch: HVM64
    c3.8xlarge:
      Arch: HVM64
    c4.large:
      Arch: HVM64
    c4.xlarge:
      Arch: HVM64
    c4.2xlarge:
      Arch: HVM64
    c4.4xlarge:
      Arch: HVM64
    c4.8xlarge:
      Arch: HVM64
    g2.2xlarge:
      Arch: HVMG2
    g2.8xlarge:
      Arch: HVMG2
    r3.large:
      Arch: HVM64
    r3.xlarge:
      Arch: HVM64
    r3.2xlarge:
      Arch: HVM64
    r3.4xlarge:
      Arch: HVM64
    r3.8xlarge:
      Arch: HVM64
    i2.xlarge:
      Arch: HVM64
    i2.2xlarge:
      Arch: HVM64
    i2.4xlarge:
      Arch: HVM64
    i2.8xlarge:
      Arch: HVM64
    d2.xlarge:
      Arch: HVM64
    d2.2xlarge:
      Arch: HVM64
    d2.4xlarge:
      Arch: HVM64
    d2.8xlarge:
      Arch: HVM64
    hi1.4xlarge:
      Arch: HVM64
    hs1.8xlarge:
      Arch: HVM64
    cr1.8xlarge:
      Arch: HVM64
    cc2.8xlarge:
      Arch: HVM64
  AWSInstanceType2NATArch:
    t1.micro:
      Arch: NATPV64
    t2.nano:
      Arch: NATHVM64
    t2.micro:
      Arch: NATHVM64
    t2.small:
      Arch: NATHVM64
    t2.medium:
      Arch: NATHVM64
    t2.large:
      Arch: NATHVM64
    m1.small:
      Arch: NATPV64
    m1.medium:
      Arch: NATPV64
    m1.large:
      Arch: NATPV64
    m1.xlarge:
      Arch: NATPV64
    m2.xlarge:
      Arch: NATPV64
    m2.2xlarge:
      Arch: NATPV64
    m2.4xlarge:
      Arch: NATPV64
    m3.medium:
      Arch: NATHVM64
    m3.large:
      Arch: NATHVM64
    m3.xlarge:
      Arch: NATHVM64
    m3.2xlarge:
      Arch: NATHVM64
    m4.large:
      Arch: NATHVM64
    m4.xlarge:
      Arch: NATHVM64
    m4.2xlarge:
      Arch: NATHVM64
    m4.4xlarge:
      Arch: NATHVM64
    m4.10xlarge:
      Arch: NATHVM64
    c1.medium:
      Arch: NATPV64
    c1.xlarge:
      Arch: NATPV64
    c3.large:
      Arch: NATHVM64
    c3.xlarge:
      Arch: NATHVM64
    c3.2xlarge:
      Arch: NATHVM64
    c3.4xlarge:
      Arch: NATHVM64
    c3.8xlarge:
      Arch: NATHVM64
    c4.large:
      Arch: NATHVM64
    c4.xlarge:
      Arch: NATHVM64
    c4.2xlarge:
      Arch: NATHVM64
    c4.4xlarge:
      Arch: NATHVM64
    c4.8xlarge:
      Arch: NATHVM64
    g2.2xlarge:
      Arch: NATHVMG2
    g2.8xlarge:
      Arch: NATHVMG2
    r3.large:
      Arch: NATHVM64
    r3.xlarge:
      Arch: NATHVM64
    r3.2xlarge:
      Arch: NATHVM64
    r3.4xlarge:
      Arch: NATHVM64
    r3.8xlarge:
      Arch: NATHVM64
    i2.xlarge:
      Arch: NATHVM64
    i2.2xlarge:
      Arch: NATHVM64
    i2.4xlarge:
      Arch: NATHVM64
    i2.8xlarge:
      Arch: NATHVM64
    d2.xlarge:
      Arch: NATHVM64
    d2.2xlarge:
      Arch: NATHVM64
    d2.4xlarge:
      Arch: NATHVM64
    d2.8xlarge:
      Arch: NATHVM64
    hi1.4xlarge:
      Arch: NATHVM64
    hs1.8xlarge:
      Arch: NATHVM64
    cr1.8xlarge:
      Arch: NATHVM64
    cc2.8xlarge:
      Arch: NATHVM64
  AWSRegionArch2AMI:
    us-east-1:
      PV64: ami-2a69aa47
      HVM64: ami-6869aa05
      HVMG2: ami-50b4f047
    us-west-2:
      PV64: ami-7f77b31f
      HVM64: ami-7172b611
      HVMG2: ami-002bf460
    us-west-1:
      PV64: ami-a2490dc2
      HVM64: ami-31490d51
      HVMG2: ami-699ad409
    eu-west-1:
      PV64: ami-4cdd453f
      HVM64: ami-f9dd458a
      HVMG2: ami-f0e0a483
    eu-central-1:
      PV64: ami-6527cf0a
      HVM64: ami-ea26ce85
      HVMG2: ami-d9d62ab6
    ap-northeast-1:
      PV64: ami-3e42b65f
      HVM64: ami-374db956
      HVMG2: ami-78ba6619
    ap-northeast-2:
      PV64: NOT_SUPPORTED
      HVM64: ami-2b408b45
      HVMG2: NOT_SUPPORTED
    ap-southeast-1:
      PV64: ami-df9e4cbc
      HVM64: ami-a59b49c6
      HVMG2: ami-56e84c35
    ap-southeast-2:
      PV64: ami-63351d00
      HVM64: ami-dc361ebf
      HVMG2: ami-2589b946
    ap-south-1:
      PV64: NOT_SUPPORTED
      HVM64: ami-ffbdd790
      HVMG2: ami-f7354198
    us-east-2:
      PV64: NOT_SUPPORTED
      HVM64: ami-f6035893
      HVMG2: NOT_SUPPORTED
    sa-east-1:
      PV64: ami-1ad34676
      HVM64: ami-6dd04501
      HVMG2: NOT_SUPPORTED
    cn-north-1:
      PV64: ami-77559f1a
      HVM64: ami-8e6aa0e3
      HVMG2: NOT_SUPPORTED
Resources:
  EC2Instance:
    Type: AWS::EC2::Instance
    Properties:
      UserData: !Base64
        Fn::Join:
        - ''
        - [IPAddress=, !Ref 'IPAddress']
      InstanceType: !Ref 'InstanceType'
      SecurityGroups: [!Ref 'InstanceSecurityGroup']
      KeyName: !Ref 'KeyName'
      ImageId: !FindInMap [AWSRegionArch2AMI, !Ref 'AWS::Region', !FindInMap [AWSInstanceType2Arch,
          !Ref 'InstanceType', Arch]]
  InstanceSecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: Enable SSH access
      SecurityGroupIngress:
      - IpProtocol: tcp
        FromPort: '22'
        ToPort: '22'
        CidrIp: !Ref 'SSHLocation'
  IPAddress:
    Type: AWS::EC2::EIP
  IPAssoc:
    Type: AWS::EC2::EIPAssociation
    Properties:
      InstanceId: !Ref 'EC2Instance'
      EIP: !Ref 'IPAddress'
Outputs:
  InstanceId:
    Description: InstanceId of the newly created EC2 instance
    Value: !Ref 'EC2Instance'
  InstanceIPAddress:
    Description: IP address of the newly created EC2 instance
    Value: !Ref 'IPAddress'

```

![key](https://user-images.githubusercontent.com/54907440/100754571-95206400-342e-11eb-9526-bc0914565a98.png)
![cl](https://user-images.githubusercontent.com/54907440/100754495-82a62a80-342e-11eb-91cf-f8367e573f3e.png)
<img width="1439" alt="スクリーンショット 2020-12-01 23 17 06" src="https://user-images.githubusercontent.com/54907440/100754358-5b4f5d80-342e-11eb-84da-391a4da5c791.png">
![スクリーンショット 2020-12-01 23 23 30](https://user-images.githubusercontent.com/54907440/100754503-846fee00-342e-11eb-980f-666ce36b36f8.png)

##　その他便利ツール系
- troposphere
https://github.com/cloudtools/troposphere
- former2
https://github.com/iann0036/former2

## Deletion Policy
スタックを削除しても、リソースを残したいときに定義する
=>　手動削除を実行したい時

```
Resources:
  myS3Bucket:
    Type: AWS::S3::Bucket
    DeletionPolicy: Retain
```
# SREになるためのおすすめ書籍
- Google:Site Reliability Engineering
https://sre.google/sre-book/table-of-contents/
- Qiitaより
https://qiita.com/tmknom/items/67dbfcf5194aee5c6e61
- LT資料
https://codezine.jp/article/detail/11307

