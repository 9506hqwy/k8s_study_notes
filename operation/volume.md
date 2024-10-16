# ボリューム

## ポッドのデプロイ

iSCSI LUN をマウントするポッドをデプロイする。

```yaml
# pod-with-iscsi.yaml
apiVersion: v1
kind: Pod
metadata:
  name: demo-vol-iscsi
  namespace: default
spec:
  containers:
  - image: nginx
    name: nginx
    ports:
    - containerPort: 80
      protocol: TCP
    volumeMounts:
    - mountPath: /mnt/iscsi0
      name: iscsi-lun0
  volumes:
  - name: iscsi-lun0
    iscsi:
      targetPortal: 10.0.0.92:3260
      portals:
      - 0.0.0.0:3260
      iqn: iqn.2003-01.org.linux-iscsi.storage.x8664:sn.a6b9465d3f31
      lun: 0
      fsType: xfs
```

デプロイする。

```sh
kubectl apply -f pod-with-iscsi.yaml
```

```
pod/demo-vol-iscsi created
```

ポッドを確認する。

```sh
kubectl get pods demo-vol-iscsi -o wide
```

```
NAME             READY   STATUS    RESTARTS   AGE   IP               NODE                  NOMINATED NODE   READINESS GATES
demo-vol-iscsi   1/1     Running   0          78s   172.17.255.177   worker01.home.local   <none>           <none>
```

ワーカノードで iSCSI イニシエータを確認する。

```sh
iscsiadm --mode node
```

```
10.0.0.92:3260,1 iqn.2003-01.org.linux-iscsi.storage.x8664:sn.a6b9465d3f31
```

ワーカノードでデバイスを確認する。

```sh
lsblk -S
```

```
NAME HCTL       TYPE VENDOR   MODEL         REV SERIAL                               TRAN
sda  6:0:0:0    disk LIO-ORG  disk01       4.0  997ecac1-8498-46cb-a765-01c96f4b7642 iscsi
sr0  0:0:0:0    rom  QEMU     QEMU DVD-ROM 2.5+ QM00001                              sata
```

ワーカノードでマウント状況を確認する。

```sh
mount | grep sda
```

```
/dev/sda on /var/lib/kubelet/plugins/kubernetes.io/iscsi/iface-default/10.0.0.92:3260-iqn.2003-01.org.linux-iscsi.storage.x8664:sn.a6b9465d3f31-lun-0 type xfs (rw,relatime,seclabel,attr2,inode64,logbufs=8,logbsize=32k,noquota)
/dev/sda on /var/lib/kubelet/pods/70316ed8-9a87-444f-8bfa-aaac84f6b013/volumes/kubernetes.io~iscsi/iscsi-lun0 type xfs (rw,relatime,seclabel,attr2,inode64,logbufs=8,logbsize=32k,noquota)
```

ワーカノードでコンテナの状態を確認する。

```sh
crictl inspect 0a549d82e82b2 | jq '.status.mounts[] | select(.containerPath == "/mnt/iscsi0")'
```

```json
{
  "containerPath": "/mnt/iscsi0",
  "gidMappings": [],
  "hostPath": "/var/lib/kubelet/pods/70316ed8-9a87-444f-8bfa-aaac84f6b013/volumes/kubernetes.io~iscsi/iscsi-lun0",
  "propagation": "PROPAGATION_PRIVATE",
  "readonly": false,
  "recursiveReadOnly": false,
  "selinuxRelabel": true,
  "uidMappings": []
}
```

コンテナ内を確認する。

```sh
kubectl exec demo-vol-iscsi -- df -Th /mnt/iscsi0
```

```
Filesystem     Type  Size  Used Avail Use% Mounted on
/dev/sda       xfs   960M   39M  922M   5% /mnt/iscsi0
```

ファイルを書き込む。

```sh
kubectl exec demo-vol-iscsi -- bash -c 'echo test > /mnt/iscsi0/test.txt'
```

```sh
kubectl exec demo-vol-iscsi -- cat /mnt/iscsi0/test.txt
```

```
test
```

## ポッドの削除

ポッドを削除する。

```sh
kubectl delete pod demo-vol-iscsi
```

```
pod "demo-vol-iscsi" deleted
```

ワーカノードで iSCSI イニシエータを確認する。

```sh
iscsiadm --mode node
```

```
iscsiadm: No records found
```

## 永続ボリューム

PersistentVolume と PersistentVolumeClaim を使用してポッドにボリュームを割り当てる準備をする。

PersistentVolume を作成する。

```yaml
# pv-iscsi-lun0.yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-iscsi-lun0
spec:
  capacity:
    storage: 1Gi
  volumeMode: Filesystem
  accessModes:
    - ReadWriteOnce
  storageClassName: ""
  persistentVolumeReclaimPolicy: Retain
  iscsi:
    targetPortal: 10.0.0.92:3260
    portals:
    - 0.0.0.0:3260
    iqn: iqn.2003-01.org.linux-iscsi.storage.x8664:sn.a6b9465d3f31
    lun: 0
    fsType: xfs
```

```sh
kubectl apply -f pv-iscsi-lun0.yaml
```

```
persistentvolume/pv-iscsi-lun0 created
```

PersistentVolume を確認する。

```sh
kubectl get pv -o wide
```

```
NAME            CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM   STORAGECLASS   VOLUMEATTRIBUTESCLASS   REASON   AGE   VOLUMEMODE
pv-iscsi-lun0   1Gi        RWO            Retain           Available                          <unset>                          19s   Filesystem
```

PersistentVolumeClaim を作成する。

```yaml
# pvc-iscsi-1g.yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-iscsi-1g
spec:
  accessModes:
    - ReadWriteOnce
  volumeMode: Filesystem
  resources:
    requests:
      storage: 1Gi
```

```sh
kubectl apply -f pvc-iscsi-1g.yaml
```

```
persistentvolumeclaim/pvc-iscsi-1g created
```

PersistentVolumeClaim を確認する。

```sh
kubectl get pvc -o wide
```

```
NAME           STATUS   VOLUME          CAPACITY   ACCESS MODES   STORAGECLASS   VOLUMEATTRIBUTESCLASS   AGE   VOLUMEMODE
pvc-iscsi-1g   Bound    pv-iscsi-lun0   1Gi        RWO                           <unset>                 5s    Filesystem
```

ポッドを作成する。
Block の場合は `volumeMounts` ではなく `volumeDevices` を使用する([参考](https://github.com/rook/rook/issues/14483))。

```yaml
# pod-with-pvc.yaml
apiVersion: v1
kind: Pod
metadata:
  name: demo-vol-pvc
  namespace: default
spec:
  containers:
  - image: nginx
    name: nginx
    ports:
    - containerPort: 80
      protocol: TCP
    volumeMounts:
    - mountPath: /mnt/iscsi0
      name: iscsi-pvc
  volumes:
  - name: iscsi-pvc
    persistentVolumeClaim:
      claimName: pvc-iscsi-1g
```

デプロイする。

```sh
kubectl apply -f pod-with-pvc.yaml
```

```
pod/demo-vol-pvc created
```

ポッドを確認する。

```sh
kubectl get pods demo-vol-pvc -o wide
```

```
NAME           READY   STATUS    RESTARTS   AGE   IP               NODE                  NOMINATED NODE   READINESS GATES
demo-vol-pvc   1/1     Running   0          77s   172.17.255.176   worker01.home.local   <none>           <none>
```

PersistentVolumeClaim を確認する。

```sh
kubectl describe pvc pvc-iscsi-1g
```

```
Name:          pvc-iscsi-1g
Namespace:     default
StorageClass:
Status:        Bound
Volume:        pv-iscsi-lun0
Labels:        <none>
Annotations:   pv.kubernetes.io/bind-completed: yes
               pv.kubernetes.io/bound-by-controller: yes
Finalizers:    [kubernetes.io/pvc-protection]
Capacity:      1Gi
Access Modes:  RWO
VolumeMode:    Filesystem
Used By:       demo-vol-pvc
Events:        <none>
```

コンテナ内を確認する。

```sh
kubectl exec demo-vol-pvc -- df -Th /mnt/iscsi0
```

```
Filesystem     Type  Size  Used Avail Use% Mounted on
/dev/sda       xfs   960M   39M  922M   5% /mnt/iscsi0
```

ファイルを確認する。

```sh
kubectl exec demo-vol-pvc -- cat /mnt/iscsi0/test.txt
```

```
test
```
