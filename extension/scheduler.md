# スケジューラ

ポッドをどのノードにデプロイする決める。

## ログレベルを変更

ログレベルを変更できるロールを作成する。

```sh
kubectl create clusterrole \
    debug-flag-admin \
    --verb=put \
    --non-resource-url=/debug/flags/v
```

```
clusterrole.rbac.authorization.k8s.io/debug-flag-admin created
```

サービスアカウントにログレベルを変更する権限を付与する。

```sh
kubectl create clusterrolebinding \
    debug-flag-admin \
    --clusterrole=debug-flag-admin \
    --serviceaccount=default:default
```

```
clusterrolebinding.rbac.authorization.k8s.io/debug-flag-admin created
```

トークンを作成する。

```sh
kubectl create token default
```

```
eyJhbGciOiJSUzI1NiIsImtpZCI6IlM...
```

ログレベルを変更する。

```sh
curl -k -H "Authorization: Bearer $TOKEN" -d 5 -X PUT  https://127.0.0.1:10259/debug/flags/v
```

```
successfully set klog.logging.verbosity to 5
```

## 起動オプション

```sh
kube-scheduler \
    --authentication-kubeconfig=/etc/kubernetes/scheduler.conf \
    --authorization-kubeconfig=/etc/kubernetes/scheduler.conf \
    --bind-address=127.0.0.1 \
    --kubeconfig=/etc/kubernetes/scheduler.conf \
    --leader-elect=true
```

## 処理確認

ポッドを作成する。

```sh
kubectl create deployment demo-sched --image=nginx --port=80
```

```
deployment.apps/demo-sched created
```

`kube-scheduler` に下記のログが出力される。

```
[eventhandlers.go:149] "Add event for unscheduled pod" pod="default/demo-sched-86675cbfff-5qr96"
[scheduling_queue.go:635] "Pod moved to an internal scheduling queue" pod="default/demo-sched-86675cbfff-5qr96" event="PodAdd" queue="Active"
[schedule_one.go:83] "About to try and schedule pod" pod="default/demo-sched-86675cbfff-5qr96"
[schedule_one.go:96] "Attempting to schedule pod" pod="default/demo-sched-86675cbfff-5qr96"
[default_binder.go:53] "Attempting to bind pod to node" logger="Bind.DefaultBinder" pod="default/demo-sched-86675cbfff-5qr96" node="worker02.home.local"
[cache.go:389] "Finished binding for pod, can be expired" podKey="1ecf3371-f271-4f54-ac31-4f73b01bbffb" pod="default/demo-sched-86675cbfff-5qr96"
[schedule_one.go:314] "Successfully bound pod to node" pod="default/demo-sched-86675cbfff-5qr96" node="worker02.home.local" evaluatedNodes=3 feasibleNodes=3
[eventhandlers.go:231] "Add event for scheduled pod" pod="default/demo-sched-86675cbfff-5qr96"
[eventhandlers.go:201] "Delete event for unscheduled pod" pod="default/demo-sched-86675cbfff-5qr96"
[eventhandlers.go:268] "Update event for scheduled pod" pod="default/demo-sched-86675cbfff-5qr96"
[eventhandlers.go:268] "Update event for scheduled pod" pod="default/demo-sched-86675cbfff-5qr96"
[eventhandlers.go:268] "Update event for scheduled pod" pod="default/demo-sched-86675cbfff-5qr96"
```

1. キューにポッドを追加する([addPodToSchedulingQueue](https://github.com/kubernetes/kubernetes/blob/v1.31.1/pkg/scheduler/eventhandlers.go#L143-L153))。
   1. アクティブ・キューに追加する([moveToActiveQ](https://github.com/kubernetes/kubernetes/blob/v1.31.1/pkg/scheduler/internal/queue/scheduling_queue.go#L606-L641))。
2. [ScheduleOne](https://github.com/kubernetes/kubernetes/blob/v1.31.1/pkg/scheduler/schedule_one.go#L65C25-L133)
   1. キューからポッドを取り出す([Pop](https://github.com/kubernetes/kubernetes/blob/v1.31.1/pkg/scheduler/internal/queue/scheduling_queue.go#L944-L977))。
   2. ポッドのスケジュール・ポリシーを取得する([frameworkForPod](https://github.com/kubernetes/kubernetes/blob/v1.31.1/pkg/scheduler/schedule_one.go#L368-L374))。
   3. 既にスケジュール済みの場合は処理を完了する。([skipPodSchedule](https://github.com/kubernetes/kubernetes/blob/v1.31.1/pkg/scheduler/schedule_one.go#L377-L395))
   4. `CycleState` を新規に作成する([NewCycleState](https://github.com/kubernetes/kubernetes/blob/v1.31.1/pkg/scheduler/framework/cycle_state.go#L60-L62))。
   5. Scheduling Cycle([schedulingCycle](https://github.com/kubernetes/kubernetes/blob/v1.31.1/pkg/scheduler/schedule_one.go#L138-L261))
      1. ワーカノードの情報を更新する([UpdateSnapshot](https://github.com/kubernetes/kubernetes/blob/v1.31.1/pkg/scheduler/internal/cache/cache.go#L185-L279))。
      2. `PreFilter` プラグインを実行する([RunPreFilterPlugins](https://github.com/kubernetes/kubernetes/blob/v1.31.1/pkg/scheduler/framework/runtime/framework.go#L698-L755))。
      3. `Filter` プラグインを実行する([RunFilterPluginsWithNominatedPods](https://github.com/kubernetes/kubernetes/blob/v1.31.1/pkg/scheduler/framework/runtime/framework.go#L973-L1018))。
      4. `Filter` Extender を実行する([findNodesThatPassExtenders](https://github.com/kubernetes/kubernetes/blob/v1.31.1/pkg/scheduler/schedule_one.go#L695-L747))。
      5. `PreScore` プラグインを実行する([RunPreScorePlugins](https://github.com/kubernetes/kubernetes/blob/v1.31.1/pkg/scheduler/framework/runtime/framework.go#L1052-L1085))。
      6. `Score` プラグインを実行する([RunScorePlugins](https://github.com/kubernetes/kubernetes/blob/v1.31.1/pkg/scheduler/framework/runtime/framework.go#L1101-L1207))。
         1. `NormalizeScore` を実行する([runScoreExtension](https://github.com/kubernetes/kubernetes/blob/v1.31.1/pkg/scheduler/framework/runtime/framework.go#L1219-L1227))。
      7. `Prioritize` Extender を実行する([prioritizeNodes](https://github.com/kubernetes/kubernetes/blob/v1.31.1/pkg/scheduler/schedule_one.go#L804-L847))。
      8. ワーカノードを選択する([selectHost](https://github.com/kubernetes/kubernetes/blob/v1.31.1/pkg/scheduler/schedule_one.go#L872-L919))。
         1. ここまでの処理でエラーが発生した場合
         2. `PostFilter` プラグインを実行する([RunPostFilterPlugins](https://github.com/kubernetes/kubernetes/blob/v1.31.1/pkg/scheduler/framework/runtime/framework.go#L908-L951))。
      9.  キャッシュを更新する([assume](https://github.com/kubernetes/kubernetes/blob/v1.31.1/pkg/scheduler/schedule_one.go#L945-L962))。
      10. `Reserve` プラグインを実行する([RunReservePluginsReserve](https://github.com/kubernetes/kubernetes/blob/v1.31.1/pkg/scheduler/framework/runtime/framework.go#L1359-L1389))。
      11. `Permit` プラグインを実行する([RunPermitPlugins](https://github.com/kubernetes/kubernetes/blob/v1.31.1/pkg/scheduler/framework/runtime/framework.go#L1443-L1490))。
   6. Binding Cycle([bindingCycle](https://github.com/kubernetes/kubernetes/blob/v1.31.1/pkg/scheduler/schedule_one.go#L264-L332)) ※ 別 goroutine で実行される。
      1. `Permit` プラグインの結果を取得する([WaitOnPermit](https://github.com/kubernetes/kubernetes/blob/v1.31.1/pkg/scheduler/framework/runtime/framework.go#L1503-L1527))。
      2. `PreBind` プラグインを実行する([RunPreBindPlugins](https://github.com/kubernetes/kubernetes/blob/v1.31.1/pkg/scheduler/framework/runtime/framework.go#L1232-L1262))。
      3. `Bind` Extender を実行する([extendersBinding](https://github.com/kubernetes/kubernetes/blob/v1.31.1/pkg/scheduler/schedule_one.go#L981-L997))。
      4. `Bind` プラグインを実行する([RunBindPlugins](https://github.com/kubernetes/kubernetes/blob/v1.31.1/pkg/scheduler/framework/runtime/framework.go#L1275-L1311))。
      5. キャッシュの期限を更新する([finishBinding](https://github.com/kubernetes/kubernetes/blob/v1.31.1/pkg/scheduler/schedule_one.go#L969-L971))。
      6. `PostBind` プラグインを実行する([RunPostBindPlugins](https://github.com/kubernetes/kubernetes/blob/v1.31.1/pkg/scheduler/framework/runtime/framework.go#L1324-L1342))。
   7. Schedule Cycle か Binding Cycle でエラーが発生した場合
      1. スケジュールできなかったポッドをキューに入れる([AddUnschedulableIfNotPresent](https://github.com/kubernetes/kubernetes/blob/v1.31.1/pkg/scheduler/internal/queue/scheduling_queue.go#L841-L887))。

## 既定のプラグイン

[既定のプラグイン](https://github.com/kubernetes/kubernetes/blob/v1.31.1/pkg/scheduler/apis/config/v1/default_plugins.go#L30-L58)は以下がある。

- [SchedulingGates](https://github.com/kubernetes/kubernetes/blob/v1.31.1/pkg/scheduler/framework/plugins/schedulinggates/scheduling_gates.go)
  - PreEnqueue プラグイン
- [PrioritySort](https://github.com/kubernetes/kubernetes/blob/v1.31.1/pkg/scheduler/framework/plugins/queuesort/priority_sort.go)
- [NodeUnschedulable](https://github.com/kubernetes/kubernetes/blob/v1.31.1/pkg/scheduler/framework/plugins/nodeunschedulable/node_unschedulable.go)
  - Filter プラグイン
- [NodeName](https://github.com/kubernetes/kubernetes/blob/v1.31.1/pkg/scheduler/framework/plugins/nodename/node_name.go)
  - Filter プラグイン
- [TaintToleration](https://github.com/kubernetes/kubernetes/blob/v1.31.1/pkg/scheduler/framework/plugins/tainttoleration/taint_toleration.go)
  - Filter プラグイン
  - PreScore プラグイン
  - Score プラグイン
  - NormalizeScore プラグイン
- [NodeAffinity](https://github.com/kubernetes/kubernetes/blob/v1.31.1/pkg/scheduler/framework/plugins/nodeaffinity/node_affinity.go)
  - PreFilter プラグイン
  - Filter プラグイン
  - PreScore プラグイン
  - Score プラグイン
  - NormalizeScore プラグイン
- [NodePort](https://github.com/kubernetes/kubernetes/blob/v1.31.1/pkg/scheduler/framework/plugins/nodeports/node_ports.go)
  - PreFilter プラグイン
  - Filter プラグイン
- [NodeResourcesFit](https://github.com/kubernetes/kubernetes/blob/v1.31.1/pkg/scheduler/framework/plugins/noderesources/fit.go)
  - PreFilter プラグイン
  - Filter プラグイン
  - PreScore プラグイン
  - Score プラグイン
- [VolumeRestrictions](https://github.com/kubernetes/kubernetes/blob/v1.31.1/pkg/scheduler/framework/plugins/volumerestrictions/volume_restrictions.go)
  - PreFilter プラグイン
  - Filter プラグイン
- [NodeVolumeLimits](https://github.com/kubernetes/kubernetes/blob/v1.31.1/pkg/scheduler/framework/plugins/nodevolumelimits/csi.go)
  - PreFilter プラグイン
  - Filter プラグイン
- [VolumeBinding](https://github.com/kubernetes/kubernetes/blob/v1.31.1/pkg/scheduler/framework/plugins/volumebinding/volume_binding.go)
  - PreFilter プラグイン
  - Filter プラグイン
  - PreScore プラグイン
  - Score プラグイン
  - Reserve プラグイン
  - PreBind プラグイン
- [VolumeZone](https://github.com/kubernetes/kubernetes/blob/v1.31.1/pkg/scheduler/framework/plugins/volumezone/volume_zone.go)
  - PreFilter プラグイン
  - Filter プラグイン
- [PodTopologySpread](https://github.com/kubernetes/kubernetes/blob/v1.31.1/pkg/scheduler/framework/plugins/podtopologyspread/plugin.go)
  - PreFilter プラグイン
  - Filter プラグイン
  - PreScore プラグイン
  - Score プラグイン
  - NormalizeScore プラグイン
- [InterPodAffinity](https://github.com/kubernetes/kubernetes/blob/v1.31.1/pkg/scheduler/framework/plugins/interpodaffinity/plugin.go)
  - PreFilter プラグイン
  - Filter プラグイン
  - PreScore プラグイン
  - Score プラグイン
  - NormalizeScore プラグイン
- [DefaultPreemption](https://github.com/kubernetes/kubernetes/blob/v1.31.1/pkg/scheduler/framework/plugins/defaultpreemption/default_preemption.go)
  - PostFilter プラグイン
    1. ポッドを優先できるかチェックする([PodEligibleToPreemptOthers](https://github.com/kubernetes/kubernetes/blob/v1.31.1/pkg/scheduler/framework/plugins/defaultpreemption/default_preemption.go#L239-L264))。
       - `preemptionPolicy=Never` なら preemption しない。
       - 他のポッドを既に preemption 中なら、それ以上 preemption しない。
    2. 候補のポッドを取得する([findCandidates](https://github.com/kubernetes/kubernetes/blob/v1.31.1/pkg/scheduler/framework/preemption/preemption.go#L210-L248))。
    3. `Preemption` Extender を実行する([callExtenders](https://github.com/kubernetes/kubernetes/blob/v1.31.1/pkg/scheduler/framework/preemption/preemption.go#L254-L310))。
    4. ポッドを選択する([SelectCandidate](https://github.com/kubernetes/kubernetes/blob/v1.31.1/pkg/scheduler/framework/preemption/preemption.go#L314-L341))。
    5. ポッドを削除する([prepareCandidate](https://github.com/kubernetes/kubernetes/blob/v1.31.1/pkg/scheduler/framework/preemption/preemption.go#L355-L387))。
- [NodeResourcesBalancedAllocation](https://github.com/kubernetes/kubernetes/blob/v1.31.1/pkg/scheduler/framework/plugins/noderesources/balanced_allocation.go)
  - PreScore プラグイン
  - Score プラグイン
- [ImageLocality](https://github.com/kubernetes/kubernetes/blob/v1.31.1/pkg/scheduler/framework/plugins/imagelocality/image_locality.go)
  - Score プラグイン
- [DefaultBinder](https://github.com/kubernetes/kubernetes/blob/v1.31.1/pkg/scheduler/framework/plugins/defaultbinder/default_binder.go)
  - Bind プラグイン

## 拡張

スケジューラの拡張は Extender または Plugin を使用する。

動作確認に [k8s-scheduler-extension](https://github.com/9506hqwy/k8s-scheduler-extension) を使用する。
ポッド名の末尾の数字とノード名の末尾の数字が一致するノードにデプロイする。

### プラグイン

ビルドする。

```sh
go build ./cmd/index-scheduler
```

スケジューラの構成ファイルを作成する。

実装したプラグインを有効化する。
`kubeconfig` は Kubernetes に接続するための構成ファイルのパスを指定する。

```yaml
# index-scheduler.yaml
apiVersion: kubescheduler.config.k8s.io/v1
kind: KubeSchedulerConfiguration
leaderElection:
  leaderElect: false
clientConnection:
  kubeconfig: /etc/kubernetes/scheduler.conf
profiles:
- schedulerName: index-scheduler
  plugins:
    filter:
      enabled:
      - name: IndexScheduling
```

スケジューラを起動する。
デフォルトスケジューラとプラグイン用のスケジューラの2つが起動する。

`authentication-kubeconfig` と `authorization-kubeconfig` は
リクエストの認証・認可のために必要な情報を取得するための構成ファイルのパスを指定する。
ポート番号はデフォルトスケジューラと重複しないように指定する。

```sh
./index-scheduler \
    --authentication-kubeconfig=/etc/kubernetes/scheduler.conf \
    --authorization-kubeconfig=/etc/kubernetes/scheduler.conf \
    --config=/root/index-scheduler.yaml \
    --secure-port=10260
```

スケジューラを指定してポッドをデプロイする。

```sh
cat | kubectl apply -f - <<EOF
apiVersion: v1
kind: Pod
metadata:
  name: sample-01
spec:
  schedulerName: index-scheduler
  containers:
  - name: sample-01
    image: nginx
---
apiVersion: v1
kind: Pod
metadata:
  name: sample-02
spec:
  schedulerName: index-scheduler
  containers:
  - name: sample-02
    image: nginx
---
apiVersion: v1
kind: Pod
metadata:
  name: sample-03
spec:
  schedulerName: index-scheduler
  containers:
  - name: sample-03
    image: nginx
EOF
```

ポッドのデプロイ先を確認する。

```sh
kubectl get pod -o wide
```

```
NAME        READY   STATUS    RESTARTS   AGE   IP               NODE                  NOMINATED NODE   READINESS GATES
sample-01   1/1     Running   0          12s   172.17.255.166   worker01.home.local   <none>           <none>
sample-02   1/1     Running   0          12s   172.17.51.170    worker02.home.local   <none>           <none>
sample-03   0/1     Pending   0          12s   <none>           <none>                <none>           <none>
```

## 参考
- [Logging](https://github.com/kubernetes/community/blob/master/contributors/devel/sig-instrumentation/logging.md)
- [#63777 Support dynamiclly set glog.logging.verbosity](https://github.com/kubernetes/kubernetes/pull/63777)
- [Scheduling Framework](https://kubernetes.io/docs/concepts/scheduling-eviction/scheduling-framework/)
