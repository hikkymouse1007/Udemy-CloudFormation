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

##　ハンズオン

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

# Resources
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

# SREになるためのおすすめ書籍
- Google:Site Reliability Engineering
https://sre.google/sre-book/table-of-contents/
- Qiitaより
https://qiita.com/tmknom/items/67dbfcf5194aee5c6e61
- LT資料
https://codezine.jp/article/detail/11307

