## この記事で伝えたいこと

- SDK/CDK/CLIが利用できるDev Containersの作り方を伝えるんだぜ
- 実際にAWS CLI/AWS SDK/AWS CDKをDev Containersで使ってみるよ

## はじめに

この記事ではDev Containersを使ってAWS向けに快適な開発環境を構築する方法を書いています。
なお、前提としては環境はmacOSまたはLinux上としています。
Windows版については後ほど追記予定です。（ディスクマウントの方法が若干異なる以外はすべて同じ設定で対応可能です。）

## AWSで利用できるコマンドやツールをすぐに使えるようにしたい

AWSマネジメントコンソールで操作してインフラを構築することはもっとも簡単な方法ですが、昨今ではさまざまな方法でAWSを操作できます。

たとえば

- AWS CLIで設定を変更する
- AWS SDKを使ってリソースを操作する
- AWS CDKでインフラのテンプレートを展開する

などです。それぞれのツールはもちろん、自分のデバイスにインストールする必要があります。
ではどのようにセットアップすれば良いのでしょうか。

## 各種ツールのセットアップに重要なこと

### いずれにしても必要なこと

AWSのツールはいずれにしても認証プロファイルが必要です。認証プロファイルは通常、ホームディレクトリに保存されています。

保存先は`~/.aws`となっているはずです。以下のコマンドで確認します。

```bash
ls -la ~/.aws
```

### AWS CLI

まずはAWS CLIです。マネジメントコンソールが操作できないときは有力な選択肢になりえます。
そんなAWS CLIにはさまざまなインストール方法がありますが、とくに理由がない限り個人的にはzip形式をおすすめしています。

[参考 - AWS CLIの最新バージョンへのインストールまたは更新 - AWS Command Line Interface](https://docs.aws.amazon.com/ja_jp/cli/latest/userguide/getting-started-install.html)

理由としてはLinux環境の動作を前提としているためです。
これがmsi（Windows）やbrew（macOS）だったりするとインストール方法が大きく変わってしまい、とても扱いにくいためです。

また、dockerfileとして落とし込んだ際にコマンドをそのまま実行できる場合がほとんどです。
なお、注意点としてはCPUアーキテクチャに気をつける必要があります。

上記を踏まえて、インストール方法をコマンドで表現すると下記のとおりになります。

```bash
arch=$(uname -m) && \
    if [ "$arch" != "x86_64" ]; then \
        arch="aarch64"; \
    fi && \
    curl "https://awscli.amazonaws.com/awscli-exe-linux-${arch}.zip" -o "aws-cli.zip" && \
    unzip aws-cli.zip && \
    sudo ./aws/install && \
    rm -rf aws aws-cli.zip
```

### AWS SDK

次にAWS SDKです。Pythonで使うことが多いため、Pythonを前提に説明します。
AWS CLIより簡単にセットアップが可能です。

結論から先に説明するとパッケージマネージャで`python3`と`python-pip`がインストールできたのであれば、すぐにインストールができます。

ただし、`--user`オプションの使用や`--break-system-packages`の利用が必要になる場合があります。くわえて、`--index-url`でパッケージレジストリを指定する場合もあるでしょう。

前提を踏まえて、AWS SDKをインストールするコマンドは以下のとおりになります。
※Linuxのパッケージマネージャーを使うものとしてaptを前提にインストールしています。

```bash
sudo apt install -y python3 python3-pip
pip install --index-url https://pypi.org/simple --user boto3 --break-system-packages
```

### AWS CDK

最後にAWS CDKです。TypeScriptを使って開発されている方が多いこととAWSでもドキュメントを見るとTypeScriptが多いため、TypeScriptを前提に説明します。

先に結論を述べるとMicrosoftが提供しているレジストリを使うと幸せになれます。

TypeScriptを使う場合の考慮点はとても多いです。たとえば

- Node.jsのバージョンをどうするか
- Node.jsをどのようにインストールするか

などです。

主にNode.jsがインストールできれば、あとはよしなになんとかなる側面があります。
ただ、そのNode.jsをどうインストールするかというところでさまざまな手段があります。

いくつか例を挙げると

- voltaを使ってインストールする
- nodebrewを使ってインストールする
- nvmを使ってインストールする

などです。個人的にはvoltaが多いですが、そうなるとNode.jsの前にvoltaをインストールしないといけません。

ではどうするかというとコンテナイメージでNode.jsのバージョンを管理してしまうというところに着地するかなと思います。もちろん、ローカルでNode.jsを実行する場合は前述のバージョン管理ツールを使うと良いです。

ということでMicrosoftが提供しているレジストリ（mcr）からコンテナイメージを取得して開発環境として利用しましょう。dockerfileでユーザを`node`にすることを忘れないようにしてください。

```dockerfile
FROM mcr.microsoft.com/vscode/devcontainers/typescript-node:18
USER node
```

## Dev Containersのコンテナイメージを作る

以上の方法を前提にDev Containersのコンテナイメージを作ります。（作りました。）

実際にgit cloneして試してみましょう。

```bash
git clone https://github.com/ymd65536/aws_devcontainer.git
```

## Dev Containersの準備

まずはDev Containersを開きましょう。git cloneしたリポジトリをVisual Studio Codeで開きます。

```bash
code aws_devcontainer
```

次にDev Containersを起動します。`Ctrl+Shift+P`または`Command+Shift+P`を実行して`reopen`と入力し、`Rebuild and Reopen in Container`を選択します。

![reopen.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/527543/5aeace1d-103d-bf88-19f8-6abef43ef4c5.png)

最後にターミナルを起動します。

![terminal.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/527543/76ed6f92-a8a7-646a-e070-dd26982a15bd.png)

## 動作確認

AWS CLI/AWS SDK/AWS CDKの動作確認をしていきます。

### AWS CLIの動作確認

動作確認の項目としては2点

- インストールされているか
- AWS環境と疎通が取れるか

まずはインストールされているかを確認するためにバージョンを確認します。

```bash
aws --version
```

実行結果

```text
aws-cli/2.19.1 Python/3.12.6 Linux/6.6.51-0-virt exe/aarch64.debian.12
```

AWS環境と疎通が取れるか、つまりは認証ができてコマンドが実行できるかを確認します。
認証することになるので実行方法が異なります。
具体的にはIAM Identity CenterでAWSにログインしているかまたはアクセスキーでログインしているかで変わります。

アクセスキーを使われている方は以下のコマンドで動作確認が可能です。重要なポイントとしてはプロファイル名です。

```bash
aws s3 ls --profile profile名
```

IAM Identity CenterでAWSにログインしている場合はブラウザで認証を実行します。

```bash
aws sso login --profile profile名
```

認証完了後に以下のコマンドを実行してください。

```bash
aws s3 ls --profile profile名
```

### AWS SDKの動作確認

次にboto3(AWS SDK for Python)がインポートできるかを確認します。
pythonを対話モードで起動するために以下のコマンドを実行します。

```bash
python3
```

以下のコードを入力して実行します。

```python
import boto3
f"boto3 version: {boto3.__version__}"
```

実行結果

```text
Python 3.11.2 (main, Aug 26 2024, 07:20:54) [GCC 12.2.0] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>> import boto3
>>> f"boto3 version: {boto3.__version__}"
'boto3 version: 1.35.54'
```

認証についてはAWS SDKのデフォルト認証情報を使っているため、確認は割愛します。
なお、興味のある方は以下のコードを実行してみるとAIと対話するための準備が整います。

```python
# us-east-1の基盤モデルを取得するサンプルコード
import boto3

bedrock_client = boto3.client("bedrock",region_name="us-east-1")
bedrock_found_list = bedrock_client.list_foundation_models()
model_summaries = bedrock_found_list['modelSummaries']

for model_summary in model_summaries:
    print(model_summary['modelArn'])

```

[引用：【AWS】GitHub CodespacesとAWS SDKでClaude 3.5 Sonnetを実行してみる](https://qiita.com/ymd65536/items/4c9f1ab9ff18ceec0928)

### AWS CDKの動作確認

最後にAWS CDKの動作確認です。以下のコマンドを実行してください。

```bash
cdk --help && cdk --version
```

実行結果

```text
Usage: cdk -a <cdk-app> COMMAND

Commands:
  cdk list [STACKS..]             Lists all stacks in the app      [aliases: ls]
  cdk synthesize [STACKS..]       Synthesizes and prints the CloudFormation
                                  template for this stack       [aliases: synth]
  cdk bootstrap [ENVIRONMENTS..]  Deploys the CDK toolkit stack into an AWS
                                  environment
  cdk deploy [STACKS..]           Deploys the stack(s) named STACKS into your
                                  AWS account
  cdk import [STACK]              Import existing resource(s) into the given
                                  STACK
  cdk watch [STACKS..]            Shortcut for 'deploy --watch'
  cdk destroy [STACKS..]          Destroy the stack(s) named STACKS
  cdk diff [STACKS..]             Compares the specified stack with the deployed
                                  stack or a local template file, and returns
                                  with status 1 if any difference is found
  cdk metadata [STACK]            Returns all metadata associated with this
                                  stack
  cdk acknowledge [ID]            Acknowledge a notice so that it does not show
                                  up anymore                      [aliases: ack]
  cdk notices                     Returns a list of relevant notices
  cdk init [TEMPLATE]             Create a new, empty CDK project from a
                                  template.
  cdk context                     Manage cached context values
  cdk docs                        Opens the reference documentation in a browser
                                                                  [aliases: doc]
  cdk doctor                      Check your set-up for potential problems

Options:
  -a, --app                REQUIRED WHEN RUNNING APP: command-line for executing
                           your app or a cloud assembly directory (e.g. "node
                           bin/my-app.js"). Can also be specified in cdk.json or
                           ~/.cdk.json                                  [string]
      --build              Command-line for a pre-synth build           [string]
  -c, --context            Add contextual string parameter (KEY=VALUE)   [array]
  -p, --plugin             Name or path of a node package that extend the CDK
                           features. Can be specified multiple times     [array]
      --trace              Print trace for stack warnings              [boolean]
      --strict             Do not construct stacks with warnings       [boolean]
      --lookups            Perform context lookups (synthesis fails if this is
                           disabled and context lookups need to be performed)
                                                       [boolean] [default: true]
      --ignore-errors      Ignores synthesis errors, which will likely produce
                           an invalid output          [boolean] [default: false]
  -j, --json               Use JSON output instead of YAML when templates are
                           printed to STDOUT          [boolean] [default: false]
  -v, --verbose            Show debug logs (specify multiple times to increase
                           verbosity)                   [count] [default: false]
      --debug              Enable emission of additional debugging information,
                           such as creation stack traces of tokens
                                                      [boolean] [default: false]
      --profile            Use the indicated AWS profile as the default
                           environment                                  [string]
      --proxy              Use the indicated proxy. Will read from HTTPS_PROXY
                           environment variable if not specified        [string]
      --ca-bundle-path     Path to CA certificate to use when validating HTTPS
                           requests. Will read from AWS_CA_BUNDLE environment
                           variable if not specified                    [string]
  -i, --ec2creds           Force trying to fetch EC2 instance credentials.
                           Default: guess EC2 instance status          [boolean]
      --version-reporting  Include the "AWS::CDK::Metadata" resource in
                           synthesized templates (enabled by default)  [boolean]
      --path-metadata      Include "aws:cdk:path" CloudFormation metadata for
                           each resource (enabled by default)          [boolean]
      --asset-metadata     Include "aws:asset:*" CloudFormation metadata for
                           resources that uses assets (enabled by default)
                                                                       [boolean]
  -r, --role-arn           ARN of Role to use when invoking CloudFormation
                                                                        [string]
      --staging            Copy assets to the output directory (use --no-staging
                           to disable the copy of assets which allows local
                           debugging via the SAM CLI to reference the original
                           source files)               [boolean] [default: true]
  -o, --output             Emits the synthesized cloud assembly into a directory
                           (default: cdk.out)                           [string]
      --notices            Show relevant notices                       [boolean]
      --no-color           Removes colors and other style from console output
                                                      [boolean] [default: false]
      --ci                 Force CI detection. If CI=true then logs will be sent
                           to stdout instead of stderr[boolean] [default: false]
      --version            Show version number                         [boolean]
  -h, --help               Show help                                   [boolean]

If your app has a single stack, there is no need to specify the stack name

If one of cdk.json or ~/.cdk.json exists, options specified there will be used
as defaults. Settings in cdk.json take precedence.
2.153.0 (build 2bccd85)
```

## おまけ：AWSの環境を使ったパッケージ開発もやりたい

ここまでくるともうひと押しやっておきたいことがあります。そう、パッケージ開発もやっていきたいですよね。準備は簡単！`build`と`twine`を`postCreateCommand.sh`でインストールします。

```bash
# CodeArtifactを使って独自のパッケージを開発する場合はbuildとtwineもインストールします。
pip install --index-url https://pypi.org/simple --user build twine --break-system-packages
```

なお、パッケージの開発方法についてはまた別の記事で説明したいと思います。

## まとめ


Dev Containersを使うと快適な開発環境を構築できることがわかりました。
また、Dev Containersで使うコンテナイメージではmcrを使っていくといくらか作業が短縮できます。開発環境をみんなで共有する場合にはDev Containersを積極的に活用しましょう。
