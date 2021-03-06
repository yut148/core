﻿# RoboticBase Coreインストールマニュアル #3


## 構築環境(2019年3月6日現在)


# 3. minikubeでモニターリング＆ロギングの開始

## 環境変数設定

1. 環境変数の設定

    ```
    $ export CORE_ROOT="${HOME}/core"
    $ cd ${CORE_ROOT};pwd
    ```

    - 実行結果（例）

        ```
        /home/fiware/core
        ```

1. 環境設定の読み込み

    ```
    $ source ${CORE_ROOT}/docs/minikube/env
    ```

## cygnus-elasticsearchの設定

1. cygnus-elasticsearch-serviceの作成

    ```
    $ kubectl apply -f cygnus/cygnus-elasticsearch-service.yaml
    ```

    - 実行結果（例）

        ```
        service/cygnus-elasticsearch created
        ```

1. cygnus-elasticsearch-deploymentの作成

    ```
    $ kubectl apply -f cygnus/cygnus-elasticsearch-deployment.yaml
    ```

    - 実行結果（例）

        ```
        deployment.apps/cygnus-elasticsearch created
        ```

1. cygnus-elasticsearchのpods状態確認

    ```
    $ kubectl get pods -l app=cygnus-elasticsearch
    ```

    - 実行結果（例）

        ```
        NAME                                    READY   STATUS    RESTARTS   AGE
        cygnus-elasticsearch-8575567db7-bx8w7   1/1     Running   0          95s
        cygnus-elasticsearch-8575567db7-c2xv7   1/1     Running   0          95s
        cygnus-elasticsearch-8575567db7-gjpth   1/1     Running   0          95s
        ```

1. cygnus-elasticsearchのservices状態確認

    ```
    $ kubectl get services -l app=cygnus-elasticsearch
    ```

    - 実行結果（例）

        ```
        NAME                   TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)             AGE
        cygnus-elasticsearch   ClusterIP   10.108.153.95   <none>        5050/TCP,8081/TCP   2m12s
        ```


## prometheusとgrafanaの設定

1. リポジトリにcoreosの追加

    ```
    $ helm repo add coreos https://s3-eu-west-1.amazonaws.com/coreos-charts/stable/
    ```

    - 実行結果（例）

        ```
        "coreos" has been added to your repositories
        ```

1. prometheus-operatorのインストール

    ```
    $ helm install coreos/prometheus-operator --name po --namespace monitoring
    ```

    - 実行結果（例）

        ```
        NAME:   po
        LAST DEPLOYED: Wed Mar  6 14:24:11 2019
        NAMESPACE: monitoring
        STATUS: DEPLOYED

        RESOURCES:
        ==> v1beta1/ClusterRole
        NAME                        AGE
        po-prometheus-operator      2m
        psp-po-prometheus-operator  2m

        ==> v1beta1/ClusterRoleBinding
        NAME                        AGE
        po-prometheus-operator      2m
        psp-po-prometheus-operator  2m

        ==> v1beta1/Deployment
        NAME                    DESIRED  CURRENT  UP-TO-DATE  AVAILABLE  AGE
        po-prometheus-operator  1        1        1           1          2m

        ==> v1/Pod(related)
        NAME                                    READY  STATUS   RESTARTS  AGE
        po-prometheus-operator-56956994f-hssjz  1/1    Running  0         2m

        ==> v1beta1/PodSecurityPolicy
        NAME                    DATA   CAPS      SELINUX   RUNASUSER  FSGROUP    SUPGROUP  READONLYROOTFS  VOLUMES
        po-prometheus-operator  false  RunAsAny  RunAsAny  MustRunAs  MustRunAs  false     configMap,emptyDir,projected,secret,downwardAPI,persistentVolumeClaim

        ==> v1/ConfigMap
        NAME                    DATA  AGE
        po-prometheus-operator  1     2m

        ==> v1/ServiceAccount
        NAME                    SECRETS  AGE
        po-prometheus-operator  1        2m

        NOTES:
        The Prometheus Operator has been installed. Check its status by running:
        kubectl --namespace monitoring get pods -l "app=prometheus-operator,release=po"

        Visit https://github.com/coreos/prometheus-operator for instructions on how
        to create & configure Alertmanager and Prometheus instances using the Operator.
        ```

1. prometheus-operatorのpods状態確認

    ```
    $ kubectl --namespace monitoring get pods -l "app=prometheus-operator,release=po"
    ```

    - 実行結果（例）

        ```
        NAME                                     READY   STATUS    RESTARTS   AGE
        po-prometheus-operator-56956994f-hssjz   1/1     Running   0          4m29s
        ```

1. kube-prometheusのインストール

    ```
    $ helm install coreos/kube-prometheus --name kp --namespace monitoring -f monitoring/kube-prometheus-minikube.yaml
    ```

    - 実行結果（例）

        ```
        NAME:   kp
        LAST DEPLOYED: Wed Mar  6 14:29:26 2019
        NAMESPACE: monitoring
        STATUS: DEPLOYED

        RESOURCES:
        ==> v1beta1/ClusterRole
        NAME                        AGE
        psp-kp-alertmanager         37s
        kp-exporter-kube-state      36s
        psp-kp-exporter-kube-state  36s
        psp-kp-exporter-node        36s
        psp-kp-grafana              35s
        kp-prometheus               35s
        psp-kp-prometheus           35s

        ==> v1beta1/Role
        kp-exporter-kube-state  32s

        ==> v1/Service
        NAME                                 TYPE       CLUSTER-IP      EXTERNAL-IP  PORT(S)    AGE
        kp-alertmanager                      ClusterIP  10.101.214.36   <none>       9093/TCP   32s
        kp-exporter-coredns                  ClusterIP  None            <none>       9153/TCP   32s
        kp-exporter-kube-controller-manager  ClusterIP  None            <none>       10252/TCP  32s
        kp-exporter-kube-etcd                ClusterIP  None            <none>       4001/TCP   31s
        kp-exporter-kube-scheduler           ClusterIP  None            <none>       10251/TCP  30s
        kp-exporter-kube-state               ClusterIP  10.98.211.64    <none>       80/TCP     29s
        kp-exporter-node                     ClusterIP  10.110.53.144   <none>       9100/TCP   26s
        kp-grafana                           ClusterIP  10.105.106.169  <none>       80/TCP     25s
        kp-prometheus                        ClusterIP  10.96.150.232   <none>       9090/TCP   24s

        ==> v1/Prometheus
        NAME           AGE
        kp-prometheus  22s

        ==> v1/ServiceMonitor
        kp-alertmanager                      12s
        kp-exporter-coredns                  12s
        kp-exporter-kube-controller-manager  11s
        kp-exporter-kube-etcd                11s
        kp-exporter-kube-scheduler           10s
        kp-exporter-kube-state               10s
        kp-exporter-kubelets                 9s
        kp-exporter-kubernetes               9s
        kp-exporter-node                     9s
        kp-grafana                           9s
        kp-prometheus                        8s

        ==> v1beta1/ClusterRoleBinding
        NAME                        AGE
        psp-kp-alertmanager         35s
        kp-exporter-kube-state      34s
        psp-kp-exporter-kube-state  34s
        psp-kp-exporter-node        33s
        psp-kp-grafana              33s
        kp-prometheus               33s
        psp-kp-prometheus           32s

        ==> v1beta1/RoleBinding
        NAME                    AGE
        kp-exporter-kube-state  32s

        ==> v1beta1/DaemonSet
        NAME              DESIRED  CURRENT  READY  UP-TO-DATE  AVAILABLE  NODE SELECTOR  AGE
        kp-exporter-node  1        1        0      1           0          <none>         24s

        ==> v1beta1/Deployment
        NAME                    DESIRED  CURRENT  UP-TO-DATE  AVAILABLE  AGE
        kp-exporter-kube-state  1        1        1           0          23s
        kp-grafana              1        1        1           0          23s

        ==> v1/PrometheusRule
        NAME                                 AGE
        kp-alertmanager                      21s
        kp-exporter-kube-controller-manager  21s
        kp-exporter-kube-etcd                21s
        kp-exporter-kube-scheduler           17s
        kp-exporter-kube-state               16s
        kp-exporter-kubelets                 15s
        kp-exporter-kubernetes               15s
        kp-exporter-node                     14s
        kp-prometheus-rules                  13s
        kp-kube-prometheus                   13s

        ==> v1/Pod(related)
        NAME                                    READY  STATUS             RESTARTS  AGE
        kp-exporter-node-k24np                  0/1    ContainerCreating  0         21s
        kp-exporter-kube-state-574bf889c-m2jzd  0/2    ContainerCreating  0         21s
        kp-grafana-6df99fd77f-c276j             0/2    ContainerCreating  0         21s

        ==> v1beta1/PodSecurityPolicy
        NAME                    DATA   CAPS      SELINUX   RUNASUSER  FSGROUP    SUPGROUP  READONLYROOTFS  VOLUMES
        kp-alertmanager         false  RunAsAny  RunAsAny  MustRunAs  MustRunAs  false     configMap,emptyDir,projected,secret,downwardAPI,persistentVolumeClaim
        kp-exporter-kube-state  false  RunAsAny  RunAsAny  MustRunAs  MustRunAs  false     configMap,emptyDir,projected,secret,downwardAPI,persistentVolumeClaim
        kp-exporter-node        false  RunAsAny  RunAsAny  MustRunAs  MustRunAs  false     configMap,emptyDir,projected,secret,downwardAPI,persistentVolumeClaim,hostPath
        kp-grafana              false  RunAsAny  RunAsAny  MustRunAs  MustRunAs  false     configMap,emptyDir,projected,secret,downwardAPI,persistentVolumeClaim,hostPath
        kp-prometheus           false  RunAsAny  RunAsAny  MustRunAs  MustRunAs  false     configMap,emptyDir,projected,secret,downwardAPI,persistentVolumeClaim

        ==> v1/Secret
        NAME             TYPE    DATA  AGE
        alertmanager-kp  Opaque  1     39s
        kp-grafana       Opaque  2     39s

        ==> v1/ConfigMap
        NAME        DATA  AGE
        kp-grafana  10    39s

        ==> v1/ServiceAccount
        NAME                    SECRETS  AGE
        kp-exporter-kube-state  1        38s
        kp-exporter-node        1        38s
        kp-grafana              1        37s
        kp-prometheus           1        37s

        ==> v1/Alertmanager
        NAME  AGE
        kp    22s

        NOTES:
        DEPRECATION NOTICE:

        - alertmanager.ingress.fqdn is not used anymore, use alertmanager.ingress.hosts []
        - prometheus.ingress.fqdn is not used anymore, use prometheus.ingress.hosts []
        - grafana.ingress.fqdn is not used anymore, use prometheus.grafana.hosts []

        - additionalRulesConfigMapLabels is not used anymore, use additionalRulesLabels
        - prometheus.additionalRulesConfigMapLabels is not used anymore, use additionalRulesLabels
        - alertmanager.additionalRulesConfigMapLabels is not used anymore, use additionalRulesLabels
        - exporter-kube-controller-manager.additionalRulesConfigMapLabels is not used anymore, use additionalRulesLabels
        - exporter-kube-etcd.additionalRulesConfigMapLabels is not used anymore, use additionalRulesLabels
        - exporter-kube-scheduler.additionalRulesConfigMapLabels is not used anymore, use additionalRulesLabels
        - exporter-kubelets.additionalRulesConfigMapLabels is not used anymore, use additionalRulesLabels
        - exporter-kubernetes.additionalRulesConfigMapLabels is not used anymore, use additionalRulesLabels
        ```

1. monitoringネームスペースのdaemonsets状態確認

    ```
    $ kubectl get daemonsets --namespace monitoring
    ```

    - 実行結果（例）

        ```
        NAME               DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR   AGE
        kp-exporter-node   1         1         1       1            1           <none>          7m12s
        ```

1. monitoringネームスペースのdeployments状態確認

    ```
    $ kubectl get deployments --namespace monitoring
    ```

    - 実行結果（例）

        ```
        NAME                     DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
        kp-exporter-kube-state   1         1         1            1           7m21s
        kp-grafana               1         1         1            1           7m21s
        po-prometheus-operator   1         1         1            1           12m
        ```

1. monitoringネームスペースのstatefulsets状態確認

    ```
    $ kubectl get statefulsets --namespace monitoring
    ```

    - 実行結果（例）

        ```
        NAME                       DESIRED   CURRENT   AGE
        alertmanager-kp            1         1         7m27s
        prometheus-kp-prometheus   1         1         7m18s
        ```

1. monitoringネームスペースのpods状態確認

    ```
    $ kubectl get pods --namespace monitoring
    ```

    - 実行結果（例）

        ```
        NAME                                      READY   STATUS    RESTARTS   AGE
        alertmanager-kp-0                         2/2     Running   0          7m31s
        kp-exporter-kube-state-7db7c95dd5-pzf9g   2/2     Running   0          4m58s
        kp-exporter-node-k24np                    1/1     Running   0          7m35s
        kp-grafana-6df99fd77f-c276j               2/2     Running   0          7m35s
        po-prometheus-operator-56956994f-hssjz    1/1     Running   0          13m
        prometheus-kp-prometheus-0                3/3     Running   1          7m25s
        ```

1. monitoringネームスペースのservices状態確認

    ```
    kubectl get services --namespace monitoring
    ```

    - 実行結果（例）

        ```
        NAME                     TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)             AGE
        alertmanager-operated    ClusterIP   None             <none>        9093/TCP,6783/TCP   7m42s
        kp-alertmanager          ClusterIP   10.101.214.36    <none>        9093/TCP            7m53s
        kp-exporter-kube-state   ClusterIP   10.98.211.64     <none>        80/TCP              7m50s
        kp-exporter-node         ClusterIP   10.110.53.144    <none>        9100/TCP            7m47s
        kp-grafana               ClusterIP   10.105.106.169   <none>        80/TCP              7m46s
        kp-prometheus            ClusterIP   10.96.150.232    <none>        9090/TCP            7m45s
        prometheus-operated      ClusterIP   None             <none>        9090/TCP            7m34s
        ```

## prometheusルールの編集

1. kp-kube-prometheusの編集

    ```
    $ kubectl edit prometheusrules --namespace monitoring kp-kube-prometheus
    ```

    ※先頭に-の付いている部分を削除してください

    ```
      for: 10m
      labels:
        severity: warning
    -   - alert: DeadMansSwitch
    -     annotations:
    -       description: This is a DeadMansSwitch meant to ensure that the entire Alerting
    -         pipeline is functional.
    -       summary: Alerting DeadMansSwitch
    -     expr: vector(1)
    -     labels:
    -       severity: none
        - expr: process_open_fds / process_max_fds
          record: fd_utilization
        - alert: FdExhaustionClose
    ```

1. kp-exporter-kube-controller-managerの編集

    ```
    $ kubectl edit prometheusrules --namespace monitoring kp-exporter-kube-controller-manager
    ```

    ※先頭に-の付いている部分を削除、先頭に+の付いている部分を追加してください

    ```
    spec:
        groups:
        - name: kube-controller-manager.rules
    -     rules:
    -      - alert: K8SControllerManagerDown
    -        annotations:
    -          description: There is no running K8S controller manager. Deployments and replication
    -            controllers are not making progress.
    -          runbook: https://coreos.com/tectonic/docs/latest/troubleshooting/controller-recovery.html#recovering-a-controller-manager
    -          summary: Controller manager is down
    -        expr: absent(up{job="kube-controller-manager"} == 1)
    -        for: 5m
    -        labels:
    -          severity: critical
    +     rules: []
    ```

1. kp-exporter-kube-schedulerの編集

    ```
    $ kubectl edit prometheusrules --namespace monitoring kp-exporter-kube-scheduler
    ```

    ※先頭に-の付いている部分を削除してください

    ```
        labels:
        quantile: "0.5"
        record: cluster:scheduler_binding_latency_seconds:quantile
    -  - alert: K8SSchedulerDown
    -    annotations:
    -      description: There is no running K8S scheduler. New pods are not being assigned
    -        to nodes.
    -      runbook: https://coreos.com/tectonic/docs/latest/troubleshooting/controller-recovery.html#recovering-a-scheduler
    -      summary: Scheduler is down
    -    expr: absent(up{job="kube-scheduler"} == 1)
    -    for: 5m
    -    labels:
    -      severity: critical
    ```


## prometheusの確認

1. コマンドの作成

    ```
    $ echo 'kubectl --namespace monitoring port-forward $(kubectl get pod --namespace monitoring -l prometheus=kube-prometheus -l app=prometheus -o template --template "{{(index .items 0).metadata.name}}") 9090:9090'
    ```

1. 別ターミナルでprometheusのポートフォワーディングを開始

    ```
    $ kubectl --namespace monitoring port-forward $(kubectl get pod --namespace monitoring -l prometheus=kube-prometheus -l app=prometheus -o template --template "{{(index .items 0).metadata.name}}") 9090:9090
    ```

    - 実行結果（例）

      ```
      Forwarding from 127.0.0.1:9090 -> 9090
      Forwarding from [::1]:9090 -> 9090
      ```

1. ブラウザでprometheusにアクセス

    ```
    xdg-open http://localhost:9090
    ```

1. prometheusのWEB管理画面が表示されたことを確認

    ![prometheus001](images/prometheus/prometheus001.png)

1. 「Alerts」を選択

   ![prometheus002](images/prometheus/prometheus002.png)

1. すべてのアラートが「0 active」と表示されることを確認

    ![prometheus003](images/prometheus/prometheus003.png)

1. メニューから「Status」「Targets」をクリック

    ![prometheus004](images/prometheus/prometheus004.png)

1. すべての「State」が「UP」になっていることを確認

    ![prometheus005](images/prometheus/prometheus005.png)

1. ブラウザを終了

1. port-forwardingを閉じる


## grafanaのData Sources追加

1. コマンドの作成

    ```
    $ echo 'kubectl --namespace monitoring port-forward $(kubectl get pod --namespace monitoring -l app=kp-grafana -o template --template "{{(index .items 0).metadata.name}}") 3000:3000'
    ```

1. 別ターミナルでgrafanaのポートフォワーディングを開始

    ```
    $ kubectl --namespace monitoring port-forward $(kubectl get pod --namespace monitoring -l app=kp-grafana -o template --template "{{(index .items 0).metadata.name}}") 3000:3000
    ```

    - 実行結果（例）

      ```
      Forwarding from 127.0.0.1:3000 -> 3000
      Forwarding from [::1]:3000 -> 3000
      ```

1. ブラウザでgrafanaにアクセス

    ```
    xdg-open http://localhost:3000
    ```

1. grafanaのログイン画面が表示されたことを確認

   ![grafana001](images/grafana/grafana001.png)

1. 「email or username」に「admin」、「password」に「admin」を入力し「Log In」をクリック

    ![grafana002](images/grafana/grafana002.png)

1. パスワードの変更画面が表示されるので、新規パスワードを入力し「Save」をクリック

    ![grafana003](images/grafana/grafana003.png)

1. ホーム画面が表示されることを確認

    ![grafana004](images/grafana/grafana004.png)

1. 「歯車」「Data Sources」をクリック

    ![grafana005](images/grafana/grafana005.png)

1. 「prometheus」をクリック

    ![grafana006](images/grafana/grafana006.png)

1. 「URL」のテキストボックスに「http://kp-prometheus:9090」を入力

    ![grafana007](images/grafana/grafana007.png)

1. 最後部の「Save & Test」をクリック

    ![grafana008](images/grafana/grafana008.png)

1. 「Data source is working」が表示されたことを確認

    ![grafana009](images/grafana/grafana009.png)

1. ブラウザを終了

1. port-forwardingを閉じる


## Elasticsearchの設定

1. elasticsearch-minikube-serviceの作成

    ```
    $ kubectl apply -f logging/elasticsearch-minikube-service.yaml
    ```

    - 実行結果（例）

        ```
        service/elasticsearch-logging created
        ```

1. elasticsearch-minikube-deploymentの作成

    ```
    $ kubectl apply -f logging/elasticsearch-minikube-deployment.yaml
    ```

    - 実行結果（例）

        ```
        serviceaccount/elasticsearch-logging created
        clusterrole.rbac.authorization.k8s.io/elasticsearch-logging created
        clusterrolebinding.rbac.authorization.k8s.io/elasticsearch-logging created
        statefulset.apps/elasticsearch-logging created
        ```

1. elasticsearch-loggingのstatefulsets状態確認

    ```
    $ kubectl get statefulsets --namespace monitoring -l k8s-app=elasticsearch-logging
    ```

    - 実行結果（例）

        ```
        NAME                    DESIRED   CURRENT   AGE
        elasticsearch-logging   2         2         3m47s
        ```

1. elasticsearch-loggingのpods状態確認

    ```
    $ kubectl get pods --namespace monitoring -l k8s-app=elasticsearch-logging
    ```

    - 実行結果（例）

        ```
        NAME                      READY   STATUS    RESTARTS   AGE
        elasticsearch-logging-0   1/1     Running   0          3m49s
        elasticsearch-logging-1   1/1     Running   0          59s
        ```

1. elasticsearch-loggingのservices状態確認

    ```
    $ kubectl get services --namespace monitoring -l k8s-app=elasticsearch-logging
    ```

    - 実行結果（例）

        ```
        NAME                    TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)    AGE
        elasticsearch-logging   ClusterIP   10.96.35.140   <none>        9200/TCP   4m23s
        ```

1. elasticsearch-loggingの接続確認

    ```
    $ kubectl exec -it elasticsearch-logging-0 --namespace monitoring -- curl -H "Content-Type: application/json" -X PUT http://elasticsearch-logging:9200/_cluster/settings -d '{"transient": {"cluster.routing.allocation.enable":"all"}}'
    ```

    - 実行結果（例）

        ```
        {"acknowledged":true,"persistent":{},"transient":{"cluster":{"routing":{"allocation":{"enable":"all"}}}}}
        ```


## fluentdの設定

1. fluentd-es-configmapの作成

    ```
    $ kubectl apply -f logging/fluentd-es-configmap.yaml
    ```

    - 実行結果（例）

        ```
        configmap/fluentd-es-config-v0.2.0 created
        ```

1. fluentd-es-dsの作成

    ```
    $ kubectl apply -f logging/fluentd-es-ds.yaml
    ```

    - 実行結果（例）

        ```
        serviceaccount/fluentd-es created
        clusterrole.rbac.authorization.k8s.io/fluentd-es created
        clusterrolebinding.rbac.authorization.k8s.io/fluentd-es created
        daemonset.apps/fluentd-es-v2.4.0 created
        ```

1. fluentd-esのdaemonsets状態確認

    ```
    $ kubectl get daemonsets --namespace monitoring -l k8s-app=fluentd-es
    ```

    - 実行結果（例）

        ```
        NAME                DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR   AGE
        fluentd-es-v2.4.0   1         1         1       1            1           <none>          1d
        ```

1. fluentd-esのpods状態確認

    ```
    $ kubectl get pods --namespace monitoring -l k8s-app=fluentd-es
    ```

    - 実行結果（例）

        ```
        NAME                      READY   STATUS    RESTARTS   AGE
        fluentd-es-v2.4.0-p28rf   1/1     Running   0          69s
        ```


## kibanaの設定

1. kibana-serviceの作成

    ```
    $ kubectl apply -f logging/kibana-service.yaml
    ```

    - 実行結果（例）

        ```
        service/kibana-logging created
        ```

1. kibana-deploymentの作成

    ```
    $ kubectl apply -f logging/kibana-deployment.yaml
    ```

    - 実行結果（例）

        ```
        deployment.apps/kibana-logging created
        ```

1. kubana-loggingのpods状態確認

    ```
    $ kubectl get pods --namespace monitoring -l k8s-app=kibana-logging
    ```

    - 実行結果（例）

        ```
        NAME                              READY   STATUS    RESTARTS   AGE
        kibana-logging-76ff7dbb49-lc46n   1/1     Running   0          2m20s
        ```


## curatorの設定

1. curator-configmapの作成

    ```
    $ kubectl apply -f logging/curator-configmap.yaml
    ```

    - 実行結果（例）

        ```
        configmap/curator-config created
        ```

1. curator-cronjobの作成

    ```
    $ kubectl apply -f logging/curator-cronjob.yaml
    ```

    - 実行結果（例）

        ```
        cronjob.batch/elasticsearch-curator created
        ```

1. elasticsearch-curatorのcronjobs状態確認

    ```
    $ kubectl get cronjobs --namespace monitoring -l k8s-app=elasticsearch-curator
    ```

    - 実行結果（例）

        ```
        NAME                    SCHEDULE     SUSPEND   ACTIVE   LAST SCHEDULE   AGE
        elasticsearch-curator   0 18 * * *   False     0        <none>          12s
        ```

## KibanaにIndex Patternsの設定

1. コマンドの作成

    ```
    $ echo 'kubectl --namespace monitoring port-forward $(kubectl get pod -l k8s-app=kibana-logging --namespace monitoring -o template --template "{{(index .items 0).metadata.name}}") 5601:5601'
    ```

1. 別ターミナルでKibanaのポートフォワーディングを開始

    ```
    $ kubectl --namespace monitoring port-forward $(kubectl get pod -l k8s-app=kibana-logging --namespace monitoring -o template --template "{{(index .items 0).metadata.name}}") 5601:5601
    ```

    - 実行結果（例）

      ```
      Forwarding from 127.0.0.1:5601 -> 5601
      Forwarding from [::1]:5601 -> 5601
      ```

1. ブラウザでkibanaにアクセス

    ```
    $ xdg-open http://localhost:5601/
    ```

1. KibanaのWEB管理画面が表示されたことを確認

    ![kibana001](images/kibana/kibana001.png)

1. 「Management」をクリック

    ![kibana002](images/kibana/kibana002.png)

1. 「Index Patterns」をクリック

    ![kibana003](images/kibana/kibana003.png)

1. 「Index pattern」のテキストボックスに「logstash-*」を入力

    ![kibana004](images/kibana/kibana004.png)

1. 「Success!」のメッセージが表示されたことを確認し「Next step」をクリック

    ![kibana005](images/kibana/kibana005.png)

1. 「Time Filter field name」のテキストボックスで「@timestamp」を選択し「Create Index pattern」をクリック

    ![kibana006](images/kibana/kibana006.png)

1. 「logstash-*」が作成されたことを確認

    ![kibana007](images/kibana/kibana007.png)

1. ブラウザを終了

1. port-forwardingを閉じる


## grafanaにelasticsearch dashboardの追加

1. コマンドの作成

    ```
    $ echo 'kubectl --namespace monitoring port-forward $(kubectl get pod --namespace monitoring -l app=kp-grafana -o template --template "{{(index .items 0).metadata.name}}") 3000:3000'
    ```

1. 別ターミナルでgrafanaのポートフォワーディングを開始

    ```
    $ kubectl --namespace monitoring port-forward $(kubectl get pod --namespace monitoring -l app=kp-grafana -o template --template "{{(index .items 0).metadata.name}}") 3000:3000
    ```

    - 実行結果（例）

      ```
      Forwarding from 127.0.0.1:3000 -> 3000
      Forwarding from [::1]:3000 -> 3000
      ```

1. ブラウザでgrafanaにアクセス

    ```
    xdg-open http://localhost:3000
    ```

1. grafanaのWEB管理画面が表示されたことを確認

    ![grafana2-001](images/grafana2/grafana2-001.png)

1. 「歯車」「Data Sources」をクリック

    ![grafana2-002](images/grafana2/grafana2-002.png)

1. 「Add data source」をクリック

    ![grafana2-003](images/grafana2/grafana2-003.png)

1. 「ElasticSearch」をクリック

    ![grafana2-004](images/grafana2/grafana2-004.png)

1. 下記の設定値を入力し、「Save & Test」をクリック

    Name : cygnus-fiwaredemo-deployer  
    URL : http://elasticsearch-logging:9200  
    Access : Server(Default)  
    Index name : logstash-* 
    Time field name : @timestamp  
    Version : 6.0+

    ![grafana2-005](images/grafana2/grafana2-005.png)

    ![grafana2-006](images/grafana2/grafana2-006.png)

1. 「Datasource Updated」と表示されることを確認

    ![grafana2-007](images/grafana2/grafana2-007.png)

1. 「+」「Import」をクリック

    ![grafana2-008](images/grafana2/grafana2-008.png)

1. 「Upload .json File」をクリック

     ![grafana2-009](images/grafana2/grafana2-009.png)

1. 「monitoring/dashboard_elasticsearch.json」を選択し「開く」をクリック

     ![grafana2-010](images/grafana2/grafana2-010.png)

1. 「Import」をクリック

    ![grafana2-011](images/grafana2/grafana2-011.png)

1. ElasticSeartchのダッシュボード画面が表示されることを確認

    ![grafana2-012](images/grafana2/grafana2-012.png)

1. ブラウザを終了

1. port-forwardingを閉じる
