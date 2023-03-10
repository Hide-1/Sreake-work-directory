# Kuberentesの運用効率化をChatGPTで実現する

# 1. はじめに
  はじめまして、Sreake事業部インターン生の井上です。

  Sreake事業部はSRE関連技術に強みを持つエンジニアによるコンサルテーションサービスを提供する事業部であり、私たちもSRE技術の調査と研究を行う目的で2023年3月6日から長期インターン生として参加しています。

  本記事では、Kuberentesの運用効率化をChatGPTで実現方法について調査・評価した事をまとめました。

# 2. ChatGPT とは？

## 概要
  - [公式](https://openai.com/blog/chatgpt)からの引用
    - 会話形式でやり取りする ChatGPT というモデルをトレーニングしました。対話形式により、ChatGPT はフォローアップの質問に答えたり、間違いを認めたり、間違った前提に異議を唱えたり、不適切な要求を拒否したりできます。
  - 要約すると、対話形式で会話ができるチャットボットです。
  - 特徴
    - 対話形式で、タスクを投げる事ができる。
    - タスク例
      - テキストの生成
      - 質問への回答
      - テキストの翻訳
      - テキストの要約
    - 自然な問答を実現する一方で、不正確な回答を出す事があります。
## ChatGPTの学習データについて
  - 主に、インターネット上のテキスト、書籍、ニュース記事、ウィキペディアの記事、映画の字幕、会話文等が含まれています。
  - IT系の技術も、ウェブサイトの記事、技術ブログ、オンラインの学術論文、コンファレンスのプレゼンテーション、またはオンラインの教育プラットフォームから収集されています。下記はChatGPT自身が学習データに含んでいる技術です。
    - プログラミング言語
    - フレームワークとライブラリ
    - データベース
    - クラウドコンピューティング
    - ビッグデータ技術
    - セキュリティとネットワーク
    - ソフトウェア開発プロセス
    - Web開発
    - モバイル開発
    - 人工知能（AI）と機械学習

## Kuberentes × ChatGPT = 運用効率化
  - ChatGPTは、学習データにIT系の技術を多く含む事から、Kuberentesについても対応ができる事が予想される。ChatGPTでKuberentesに関する問題を相談できるので、Kuberentesから出てくるエラー解釈し、運用チームに解説付きで、通知する事が期待できる。また、エラーが出た瞬間に、直し方やデバッグの指針まで、ChatGPTが提案する事が期待できる。

# 3. 検証方法
Kuberentesの運用効率化をChatGPTで実現する方法の検証として、実際に起きたエラーをChatGPTに質問、分析させ、任意のフォーマットで回答の出力を行わせる。
## 質問フォーマット
  ```markdown
    # Kubernetesの運用中に以下のエラーが出ました。以下の情報を元に、解決の支援をお願いします。
      
    # 現在のKubernetesの構成は以下の通りです。
    - masterノード : 2
    - workerノード : 3

    # 解答は以下のフォーマットでお願いします。ChatGPTの回答は<ChatGPT_Answer>に入力して下さい。
    1. 障害箇所切り分け(Kubernetesを構成するコンポーネント、それ以外の場合でも、どこに問題があるか一言で入力して下さい。)
      - 障害箇所 : <ChatGPT_Answer>
        - 追加情報 : <ChatGPT_Answer>
    2. エラー内容の説明
      - <ChatGPT_Answer>
    3. なぜそのエラーになったか？原因として予測される内容(複数個ある場合は箇条書きで入力して下さい。)
      - <ChatGPT_Answer>
    4. エラーのデバッグ方針(複数個ある場合は箇条書きで入力して下さい。)
      - <ChatGPT_Answer>
    5. エラーの解決方法(複数個ある場合は箇条書きで入力して下さい。)
      - <ChatGPT_Answer>

    # エラー文は以下です。
    ```
    <エラー文を挿入>
    ```
  ```

# 4. 検証環境
オンプレミス環境上にVMを5つ用意して、2つをMasterノード、3つをWorkerノードのクラスタを構築した。リソースは下記の通り。
<!-- #! 図を挿入する -->
- 各VMのスペック
  - CPU数: 2
  - メモリ： 4GB
# 5. 実際に検証してみる
## A. Podのコンテナエラー
最初は意図的にエラーを発生させるため、以下のdeployment.yamlを検証用のK8sクラスタにデプロイする。このdeployment.yamlのコンテナは、実行時の引数が指定されていないため、Podがクラッシュする。
- deployment.yaml
  ```yaml
  # deployment.yaml
  apiVersion: apps/v1
  kind: Deployment
  metadata:
    name: test-deployment
  spec:
    selector:
      matchLabels:
        app: ubuntu
    replicas: 3
    template:
      metadata:
        labels:
          app: ubuntu
      spec:
        containers:
        - name: ubuntu
          image: ubuntu:20.04
  ```

- ChatGPTへ与える情報
  - Pod名を取得
    ```
    $ kubectl get pods

      NAME                              READY   STATUS             RESTARTS         AGE
      test-deployment-f4955dc4d-95wrl   0/1     CrashLoopBackOff   29 (2m15s ago)   125m
      test-deployment-f4955dc4d-ss6nq   0/1     CrashLoopBackOff   29 (114s ago)    125m
      test-deployment-f4955dc4d-tdkxc   0/1     CrashLoopBackOff   29 (2m31s ago)   125m
    ```
  - Pod名からdescribe情報を取得して、ChatGPTに与える
    ```
    $ kubectl describe pods <クラッシュしたPod名>
    
      Name:             test-deployment-f4955dc4d-95wrl
      Namespace:        k8s-and-chatgpt
      Priority:         0

      ...
      <長いので省略>
      ...

      Events:
        Type     Reason     Age                    From               Message
        ----     ------     ----                   ----               -------
        Normal   Scheduled  50m                    default-scheduler  Successfully assigned k8s-and-chatgpt/test-deployment-f4955dc4d-95wrl to 3shake-k8s-node-001
        Normal   Pulled     48m (x5 over 50m)      kubelet            Container image "ubuntu:20.04" already present on machine
        Normal   Created    48m (x5 over 50m)      kubelet            Created container ubuntu
        Normal   Started    48m (x5 over 50m)      kubelet            Started container ubuntu
        Warning  BackOff    4m59s (x209 over 50m)  kubelet            Back-off restarting failed container ubuntu in pod test-deployment-f4955dc4d-95wrl_k8s-and-chatgpt(b3d79768-1bbd-470f-bb02-ebfaea2c53f9)
    ```

- ChatGPTの回答
  ```markdown
    # 障害箇所切り分け
      - 障害箇所 : Pod(test-deployment-f4955dc4d-95wrl)内のubuntuコンテナ
        - 追加情報 : コンテナがCrashLoopBackOff状態であり、直前にTerminatedされたことがある。
    # エラー内容の説明
      - Podのうち、ubuntuコンテナがCrashLoopBackOff状態にあり、起動できなくなっている。
    # なぜそのエラーになったか？原因として予測される内容
      - ubuntuコンテナの起動に失敗し、続けて起動に失敗した場合に、PodがCrashLoopBackOff状態になります。
      - 具体的なエラーの原因は、Pod内のコンテナで発生した問題によるものである可能性があります。したがって、Podのログを確認することが重要です。
    # エラーのデバッグ方針
      - kubectl logsコマンドを使用してPodのログを取得し、問題の原因を確認する
      - kubectl describe podコマンドを使用してPodの状態を確認する
      - もし、コンテナが何らかの理由ですぐに終了する場合は、開始時にログを表示させるなどして、コンテナ内の問題をデバッグする。
    # エラーの解決方法
      - コンテナのログを確認して、エラーを特定し、それに応じて対処する。
      - Podの状態を確認し、必要に応じてPodを再作成する。
  ```

- 評価
  - describe情報から、障害箇所や障害原因を要約できている。
  - コンテナ内のエラーという事を読み取れている。
  - エラー内容の説明、原因予測は正解である。
  - デバッグ方針等、実際に起きた場合の対処法と概ね一致していると考えられる。

## B. Prometheus導入中に起きたエラー
オンプレミスKubernetes環境にて、HelmでPrometheusを導入したが、以下の様にPending状態になる問題に遭遇した。ここでは、KubernetesのEvent情報をChatGPTに与え、解決を試みる。
```
root@3shake-main:~# kubectl get pods -o=wide
NAME                                                READY   STATUS    RESTARTS   AGE    IP              NODE                  NOMINATED NODE   READINESS GATES
prometheus-alertmanager-0                           0/1     Pending   0          119s   <none>          <none>                <none>           <none>
prometheus-kube-state-metrics-7f6769f7c6-8gl5h      1/1     Running   0          119s   10.244.2.81     3shake-k8s-node-001   <none>           <none>
prometheus-prometheus-node-exporter-8ggvh           1/1     Running   0          119s   192.168.50.64   3shake-k8s-node-003   <none>           <none>
prometheus-prometheus-node-exporter-kggrn           1/1     Running   0          119s   192.168.50.62   3shake-k8s-node-001   <none>           <none>
prometheus-prometheus-node-exporter-tc82p           1/1     Running   0          119s   192.168.50.63   3shake-k8s-node-002   <none>           <none>
prometheus-prometheus-pushgateway-684dc6674-xd2qj   1/1     Running   0          119s   10.244.2.80     3shake-k8s-node-001   <none>           <none>
prometheus-server-55db4c8c5d-vm5ds                  0/2     Pending   0          119s   <none>          <none>                <none>           <none>
```

- ChatGPTへ与える情報
  - kubectl get eventでイベント情報を取得する。ここで、hatGPTは、時に長い入力を受け付ける事が出来ないので、必要な情報のみを抽出してChatGPTへ与える。
    ```
    $ kubectl get event | grep -e Warning -e Failed

      LAST SEEN   TYPE      REASON                   OBJECT                                                    MESSAGE
      49m         Warning   FailedScheduling         pod/prometheus-alertmanager-0                             0/3 nodes are available: pod has unbound immediate PersistentVolumeClaims. preemption: 0/3 nodes are available: 3 No preemption victims found for incoming pod..
      44m         Warning   FailedScheduling         pod/prometheus-alertmanager-0                             0/3 nodes are available: pod has unbound immediate PersistentVolumeClaims. preemption: 0/3 nodes are available: 3 No preemption victims found for incoming pod..

      ...
      略
      ...

      19m         Normal    FailedBinding            persistentvolumeclaim/storage-prometheus-alertmanager-0   no persistent volumes available for this claim and no storage class is set
      2m11s       Normal    FailedBinding            persistentvolumeclaim/storage-prometheus-alertmanager-0   no persistent volumes available for this claim and no storage class is set

    ```

- ChatGPTの回答
  ```
  1. 障害箇所切り分け
    - 障害箇所 : PersistentVolumeClaimsがbindされておらず、スケジューリングに失敗しているPod
    - 追加情報 : Prometheus Alertmanager Pod
  2. エラー内容の説明
    - Podがスケジュールできず、一部のPodが削除されました。
  3. なぜそのエラーになったか？原因として予測される内容
    - 予約された永続ボリュームが利用可能でないため、PodのPersistentVolumeClaimsがbindできなかった。
    - また、スケジューリングできるノードが存在しなかった可能性があります。
  4. エラーのデバッグ方針
    - PersistentVolumeClaimsの状態を確認して、予約された永続ボリュームが利用可能であることを確認する。
    - スケジューリングできるノードが存在することを確認する。
    - ノードのリソース使用状況を確認し、Podがリソース不足によってスケジュールされなかった可能性があるかどうかを確認する。
  5. エラーの解決方法
    - PersistentVolumeClaimsが正常にbindできるように永続ボリュームの予約を確認し、利用可能であることを確認する。
    - スケジューリングできるノードが存在することを確認する。
    - ノードのリソース不足によってスケジュールされなかった場合は、リソースを増やすか、Podのリソース要求を減らすことで解決できるかもしれません。
  ```
- 評価WIP
  - 実際にPVCを見直してみる？

## C. ノード系のエラー？
