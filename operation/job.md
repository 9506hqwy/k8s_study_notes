# ジョブ

ポッドを指定した回数実行する。実行が完了したポッドは停止する。

`uname` を実行するジョブを作成する。

```sh
kubectl create job uname --image=redhat/ubi9 -- uname -a
```

```text
job.batch/uname created
```

ジョブを確認する。

```sh
kubectl get job uname
```

```text
NAME    STATUS     COMPLETIONS   DURATION   AGE
uname   Complete   1/1           18s        51s
```

ポッドを確認する。

```sh
kubectl get pods -o wide
```

```text
NAME          READY   STATUS      RESTARTS   AGE    IP              NODE                  NOMINATED NODE   READINESS GATES
uname-2mkpb   0/1     Completed   0          119s   172.17.51.140   worker02.home.local   <none>           <none>
```

出力を確認する。

```sh
kubectl logs uname-2mkpb
```

```text
Linux uname-2mkpb 5.14.0-522.el9.x86_64 #1 SMP PREEMPT_DYNAMIC Sun Oct 20 13:04:34 UTC 2024 x86_64 x86_64 x86_64 GNU/Linux
```

ジョブを定期的に実行する。

```sh
kubectl create cronjob uname-cron --image=redhat/ubi9 --schedule='*/1 * * * *' -- uname -a
```

```text
cronjob.batch/uname-cron created
```

ジョブを確認する。

```sh
kubectl get cronjob uname-cron
```

```text
NAME         SCHEDULE      TIMEZONE   SUSPEND   ACTIVE   LAST SCHEDULE   AGE
uname-cron   */1 * * * *   <none>     False     0        11s             18s
```

```sh
kubectl get job
```

```text
NAME                  STATUS     COMPLETIONS   DURATION   AGE
uname-cron-28858538   Complete   1/1           5s         14m
uname-cron-28858539   Complete   1/1           5s         13m
uname-cron-28858540   Complete   1/1           5s         12m
```

ポッドを確認する。実行が完了したポッドは既定で 3 つまで保持される。

```sh
kubectl get pods -o wide
```

```text
NAME                        READY   STATUS      RESTARTS   AGE     IP              NODE                  NOMINATED NODE   READINESS GATES
uname-cron-28858530-t4b42   0/1     Completed   0          84s     172.17.51.167   worker02.home.local   <none>           <none>
uname-cron-28858531-svvwh   0/1     Completed   0          24s     172.17.51.182   worker02.home.local   <none>           <none>
```

出力を確認する。

```sh
kubectl logs uname-cron-28858530-t4b42
```

```text
Linux uname-cron-28858530-t4b42 5.14.0-522.el9.x86_64 #1 SMP PREEMPT_DYNAMIC Sun Oct 20 13:04:34 UTC 2024 x86_64 x86_64 x86_64 GNU/Linux
```

ジョブを停止する。

```sh
kubectl patch cronjob uname-cron -p '{"spec":{"suspend":true}}'
```

ジョブを確認する。

```sh
kubectl get cronjob uname-cron
```

```text
NAME         SCHEDULE      TIMEZONE   SUSPEND   ACTIVE   LAST SCHEDULE   AGE
uname-cron   */1 * * * *   <none>     True      0        65s             6m55s
```
