# 🧩 CISO エージェントでのベンチマーク実行ガイド

このガイドでは、**CISO エージェント**を使用して ITBench ベンチマークを起動する方法を説明します。  
評価中は 2 つの Docker コンテナ（**Agent Harness** と **Bench Runner**）を起動し、実行したままにしておきます。

> ⚠️ **注意:** 複数の Agent Harness または Bench Runner を同時に実行しないでください。

---

## 🚀 オプション 1：CISO CAA エージェント（事前ビルド済み）を使用する

公式の [CISO CAA エージェント](https://github.com/itbench-hub/itbench-ciso-caa-agent) を使用してベンチマークを実行する場合は、以下の手順に従ってください。

### 1. `.env` ファイルを作成する

以下の内容で `.env` ファイルを作成します：

```
OPENAI_API_KEY = <YOUR OPENAI API KEY>
OPENAI_MODEL_NAME = gpt-4o-mini
CODE_GEN_MODEL = gpt-4o-mini
```

別のモデルを使用したい場合は、[こちら](https://github.com/itbench-hub/itbench-ciso-caa-agent?tab=readme-ov-file#3-create-env-file-and-set-llm-api-credentials)を参照してください。

---

### 2. CISO Agent Harness コンテナを起動する

以下のコマンドを実行してコンテナを起動します。  
`<ABSOLUTE_PATH/TO/AGENT_MANIFEST>` および `<ABSOLUTE_PATH/TO/ENVFILE>` をそれぞれ自分の環境に置き換えてください。

```
docker run --rm -it --name ciso-agent-harness \
    --mount type=bind,src=<ABSOLUTE_PATH/TO/AGENT_MANIFEST>,dst=/tmp/agent-manifest.json \
    --mount type=bind,src=<ABSOLUTE_PATH/TO/ENVFILE>,dst=/etc/ciso-agent/.env \
    quay.io/it-bench/ciso-agent-harness:latest \
    --host itbench.apps.prod.itbench.res.ibm.com \
    --benchmark_timeout 3600
```

<img width="614" alt="image" src="https://github.com/itbench-hub/ITBench-Tutorials/blob/gh-pages/docs/images/1.2.png">

---

### 3. CISO Bench Runner コンテナを起動する

新しいターミナルを開き、以下のコマンドを実行します。  
`<ABSOLUTE_PATH/TO/AGENT_MANIFEST>` と `<ABSOLUTE_PATH/TO/KUBECONFIG_FILE>` を自分のパスに置き換えてください。

（RHEL シナリオを実行する場合は、[ベンチランナーの完全仕様](#ベンチランナーの完全仕様)を参照してください）

```
docker run --rm -it --name ciso-bench-runner \
    --mount type=bind,src=<ABSOLUTE_PATH/TO/AGENT_MANIFEST>,dst=/tmp/agent-manifest.json \
    --mount type=bind,src=<ABSOLUTE_PATH/TO/KUBECONFIG_FILE>,dst=/tmp/kubeconfig.yaml \
    quay.io/it-bench/ciso-bench-runner:latest \
    --host itbench.apps.prod.itbench.res.ibm.com \
    --runner_id my-ciso-runner-1
```

<img width="614" alt="image" src="https://github.com/itbench-hub/ITBench-Tutorials/blob/gh-pages/docs/images/1.3.png">

---

### 4. ベンチマークの進行とステータス確認

- ベンチマークは起動後、自動的に進行します。  
  約 1 時間ほどで完了し、両方のコンテナが自動停止します。  
  完了後はターミナルを閉じても問題ありません。
- 実行中は、登録済みの Issue が 10 分ごとに更新され、各シナリオの進捗テーブルが表示されます。

<img width="614" alt="image" src="https://github.com/itbench-hub/ITBench-Tutorials/blob/gh-pages/docs/images/1.4.png">

| フィールド名 | 説明 |
|:---------------|:--------------------------|
| Scenario Name | シナリオ名 |
| Description | 評価対象のコントロール説明 |
| Passed | シナリオの成功可否（True/False） |
| Time To Resolve | 実行完了までの時間 |
| Error | 発生したエラー内容 |
| Message | 追加情報やステータス |
| Date | 実行完了日時 |

---

### 5. ベンチマーク完了後

- すべてのシナリオが完了すると、Docker コマンドは自動的に停止します。  

    <img width="614" alt="image" src="https://github.com/itbench-hub/ITBench-Tutorials/blob/gh-pages/docs/images/1.5.1.png">

- Issue のコメントが **Finished** に更新され、自動的にクローズされます。  

    <img width="614" alt="image" src="https://github.com/itbench-hub/ITBench-Tutorials/blob/gh-pages/docs/images/1.5.2.png">

---

### 6. トラブルシューティング

- **ベンチマークが開始しない場合**
  - Issue に `abort` とコメントを追加してください。
  - 必要に応じて状況説明を追記してください。
- **コンテナが終了しない場合**
  - テーブルの「Date」が更新されていない場合、停止している可能性があります。
  - `Ctrl + C` でコンテナを終了し、Issue に `abort` とコメントしてください。

---

### 7. リーダーボードの更新

- ベンチマーク結果は数日以内にリーダーボードへ反映されます。
    <img width="614" alt="image" src="https://github.com/itbench-hub/ITBench-Tutorials/blob/gh-pages/docs/images/1.7.png">
- 数日経っても反映されない場合は、[サポート連絡先](#サポート連絡)までお問い合わせください。

---

## 🧠 オプション 2：独自エージェントを使用する

独自のカスタムエージェントを提出する場合は、次の手順に従います。

### 1. Agent Harness 設定ファイルの作成

```yaml
# このフィールドは、シナリオ環境データが保存されるパスを定義します。
path_to_data_provided_by_scenario: /tmp/agent/scenario_data.json 

# このフィールドは、エージェントの出力結果を保存するパスを定義します。
path_to_data_pushed_to_scenario: /tmp/agent/agent_data.txt 

# Agent Harness により実行されるコマンド
run:
  command: ["/bin/bash"]
  args:
  - -c
  - |
    <あなたのエージェント実行コマンドをここに記述>
```

詳しくは [公式サンプル](https://github.com/itbench-hub/ITBench-CISO-CAA-Agent/blob/main/agent-harness.yaml) を参照してください。
たとえば、以下は サンプル CISO CAA エージェントの Agent Harness 設定ファイルの例です。
この設定はエラー処理などを含むため少し複雑に見えますが、自分で作成する際はここまで複雑にする必要はありません。
ただし、無限ループを防ぐために適切な終了処理を含めることは必須です。

```yaml
path_to_data_provided_by_scenario: /tmp/agent/scenario_data.json
path_to_data_pushed_to_scenario: /tmp/agent/agent_data.tar
run:
command: ["/bin/bash"]
args:
- -c
- |

    timestamp=$(date +%Y%m%d%H%M%S)
    tmpdir=/tmp/agent/${timestamp}
    mkdir -p ${tmpdir}

    cat /tmp/agent/scenario_data.json > ${tmpdir}/scenario_data.json

    jq -r .goal_template ${tmpdir}/scenario_data.json > ${tmpdir}/goal_template.txt
    jq -r .vars.kubeconfig ${tmpdir}/scenario_data.json > ${tmpdir}/kubeconfig.yaml
    jq -r .vars.ansible_ini ${tmpdir}/scenario_data.json > ${tmpdir}/ansible.ini
    jq -r .vars.ansible_user_key ${tmpdir}/scenario_data.json > ${tmpdir}/user_key
    chmod 600 ${tmpdir}/user_key
    sed -i.bak -E "s|(ansible_ssh_private_key_file=\")[^\"]*|\1${tmpdir}/user_key|" ${tmpdir}/ansible.ini
    
    sed "s|{{ kubeconfig }}|${tmpdir}/kubeconfig.yaml|g" ${tmpdir}/goal_template.txt > ${tmpdir}/goal.txt
    sed -i.bak -E "s|\{\{ path_to_inventory \}\}|${tmpdir}/ansible.ini|g" ${tmpdir}/goal.txt

    echo "You can use \`${tmpdir}\` as your workdir." >> ${tmpdir}/goal.txt
    
    source .venv/bin/activate
    timeout 200 python src/ciso_agent/main.py --goal "`cat ${tmpdir}/goal.txt`" --auto-approve -o ${tmpdir}/agent-result.json || true

    tar -C ${tmpdir} -cf /tmp/agent/agent_data.tar .
```
---

### 2. Docker イメージを作成する

Agent Harness のベースイメージを使い、あなたのエージェントを含む Docker イメージをビルドします。  
例として、CISO Agent 用の Dockerfile は以下の通りです：

```
FROM icr.io/agent-bench/ciso-agent-harness-base:0.0.3 AS base
RUN ln -sf /bin/bash /bin/sh
RUN apt update -y && apt install -y curl gnupg2 unzip ssh
COPY itbench-ciso-caa-agent /etc/ciso-agent
WORKDIR /etc/ciso-agent
RUN python -m venv .venv && source .venv/bin/activate && pip install -r requirements-dev.txt --no-cache-dir
RUN pip install --upgrade ansible-core jmespath kubernetes==31.0.0 setuptools==70.0.0 --no-cache-dir
RUN ansible-galaxy collection install kubernetes.core community.crypto
RUN echo "StrictHostKeyChecking no" >> /etc/ssh/ssh_config
RUN apt update -y && apt install -y jq
RUN curl -LO https://dl.k8s.io/release/v1.31.0/bin/linux/$(dpkg --print-architecture)/kubectl && chmod +x ./kubectl && mv ./kubectl /usr/local/bin/kubectl
RUN curl "https://awscli.amazonaws.com/awscli-exe-linux-$(uname -m).zip" -o "awscliv2.zip" && unzip awscliv2.zip && ./aws/install
RUN curl -L -o opa https://github.com/open-policy-agent/opa/releases/download/v1.0.0/opa_linux_$(dpkg --print-architecture)_static && chmod +x ./opa && mv ./opa /usr/local/bin/opa
RUN python -m venv .venv && source .venv/bin/activate && pip install -e /etc/ciso-agent --no-cache-dir
COPY agent-bench-automation.wiki/.gist/agent-harness/entrypoint.sh /etc/entrypoint.sh
RUN chmod +x /etc/entrypoint.sh
WORKDIR /etc/agent-benchmark
ENTRYPOINT ["/etc/entrypoint.sh"]
```

---

## 🎉 まとめ

これで ITBench の CISO ベンチマーク実行手順が完了しました。  
実行結果は数日以内にリーダーボードへ反映されます。

---

## 🆘 サポート連絡

数日経っても応答がない場合は、登録 Issue にコメントを追加し、以下のメンションを行ってください：

- メンション先: `@yana`, `@rohanarora`  
- ラベル: `need help`

**コメント例:**

```
@yana, @rohanarora
登録リクエストに関する返信がまだありません。
"need help" ラベルを追加しました。
```

---

## 📄 ベンチランナーの完全仕様

```
docker run --rm -it --name ciso-bench-runner \
    --mount type=bind,src=<ABSOLUTE_PATH/TO/AGENT_MANIFEST>,dst=/tmp/agent-manifest.json \
    --mount type=bind,src=<ABSOLUTE_PATH/TO/KUBECONFIG_FILE>,dst=/tmp/kubeconfig.yaml \
    --mount type=bind,src=<PATH/TO/RHEL_MACHINE_SSHKEY>,dst=/tmp/rhel-bundle-config/ssh_key \
    quay.io/it-bench/ciso-bench-runner:latest \
    --host itbench.apps.prod.itbench.res.ibm.com \
    --runner_id my-ciso-runner-1 \
    --rhel_address <IP Address or Hostname of the RHEL9 machine used for RHEL Scenario> \
    --rhel_username <Username of the RHEL9 machine used for RHEL Scenario>
```
