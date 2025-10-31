# 仮想マシン

## SSH 接続

接続元で SSH 鍵を作成する。

```sh
ssh-keygen -t ed25519
```

SSH 鍵をシークレットに登録する。

```sh
kubectl create secret generic my-keys --from-file="$HOME/.ssh/id_ed25519.pub"
```

```text
secret/my-keys created
```

作成したシークレットを指定して仮想マシンをデプロイする。

```sh
cat | kubectl apply -f - <<EOF
apiVersion: kubevirt.io/v1
kind: VirtualMachine
metadata:
  name: vm-ssh
spec:
  runStrategy: Halted
  template:
    spec:
      accessCredentials:
      - sshPublicKey:
          propagationMethod:
            qemuGuestAgent:
              users:
              - centos
          source:
            secret:
              secretName: my-keys
      domain:
        devices:
          disks:
            - name: osdisk
              disk:
                bus: virtio
            - name: cloudinitdisk
              disk:
                bus: virtio
          interfaces:
          - name: default
            masquerade: {}
            model: virtio
        resources:
          requests:
            memory: 2048M
      networks:
      - name: default
        pod: {}
      volumes:
        - name: osdisk
          containerDisk:
            image: quay.io/containerdisks/centos-stream:9
        - name: cloudinitdisk
          cloudInitNoCloud:
            userData: |-
              #cloud-config
              user: centos
              runcmd:
                - [ setsebool, -P, 'virt_qemu_ga_manage_ssh', 'on' ]
EOF
```

```text
virtualmachine.kubevirt.io/vm-ssh created
```

仮想マシンを確認する。

```sh
kubectl get vm
```

```text
NAME     AGE   STATUS    READY
vm-ssh   33s   Stopped   False
```

仮想マシンを起動する。

```sh
kubectl virt start vm-ssh
```

```text
VM vm-ssh was scheduled to start
```

仮想マシンのインスタンスを確認する。

```sh
kubectl get vmi -o wide
```

```text
NAME     AGE     PHASE     IP              NODENAME              READY   LIVE-MIGRATABLE   PAUSED
vm-ssh   2m15s   Running   172.17.51.166   worker02.home.local   True    True
```

SSH で接続する。

```sh
kubectl virt ssh -i "$HOME/.ssh/id_ed25519" -l centos vm/vm-ssh
```

```text
The authenticity of host 'vm.vm-ssh.default (<no hostip for proxy command>)' can't be established.
ED25519 key fingerprint is SHA256:ZO9wvNp3Qe8UYQu0iRtptb4Ll0f5Kkcq7LrmaFS3oFg.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added 'vm.vm-ssh.default' (ED25519) to the list of known hosts.
[centos@vm-ssh ~]$
```

サービスとして公開する。

```sh
kubectl virt expose vm vm-ssh --name vm-ssh-22 --port 20022 --target-port 22 --type LoadBalancer
```

```text
Service vm-ssh-22 successfully created for vm vm-ssh
```

サービスを確認する。

```sh
kubectl get service vm-ssh-22
```

```text
NAME        TYPE           CLUSTER-IP    EXTERNAL-IP    PORT(S)           AGE
vm-ssh-22   LoadBalancer   10.109.92.7   172.16.0.101   20022:30469/TCP   10s
```

接続確認する。

```sh
ssh -i "$HOME/.ssh/id_ed25519" -p 20022 centos@172.16.0.101
```

```text
The authenticity of host '[172.16.0.101]:20022 ([172.16.0.101]:20022)' can't be established.
ED25519 key fingerprint is SHA256:ZO9wvNp3Qe8UYQu0iRtptb4Ll0f5Kkcq7LrmaFS3oFg.
This host key is known by the following other names/addresses:
    ~/.ssh/known_hosts:1: vm.vm-ssh.default
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '[172.16.0.101]:20022' (ED25519) to the list of known hosts.
Last login: Wed Oct 29 07:04:00 2025 from 172.17.2.39
[centos@vm-ssh ~]$
```

## 永続化ストレージ

PersistentVolume は下記とする。

```sh
kubectl get pv -o wide
```

```text
NAME            CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                  STORAGECLASS   VOLUMEATTRIBUTESCLASS   REASON   AGE   VOLUMEMODE
pv-iscsi-lun0   1Gi        RWO            Retain           Bound    default/pvc-iscsi-1g                  <unset>                          89s   Filesystem
pv-iscsi-lun1   2Gi        RWO            Retain           Bound    default/pvc-iscsi-2g                  <unset>                          87s   Filesystem
pv-iscsi-lun2   3Gi        RWO            Retain           Bound    default/pvc-iscsi-3g                  <unset>                          85s   Filesystem
```

PersistentVolumeClaim は下記とする。

```sh
kubectl get pvc -o wide
```

```text
NAME           STATUS   VOLUME          CAPACITY   ACCESS MODES   STORAGECLASS   VOLUMEATTRIBUTESCLASS   AGE   VOLUMEMODE
pvc-iscsi-1g   Bound    pv-iscsi-lun0   1Gi        RWO                           <unset>                 20s   Filesystem
pvc-iscsi-2g   Bound    pv-iscsi-lun1   2Gi        RWO                           <unset>                 18s   Filesystem
pvc-iscsi-3g   Bound    pv-iscsi-lun2   3Gi        RWO                           <unset>                 16s   Filesystem
```

仮想マシンをデプロイする。

```sh
cat | kubectl apply -f - <<EOF
apiVersion: kubevirt.io/v1
kind: VirtualMachine
metadata:
  name: vm-disk
spec:
  runStrategy: Halted
  template:
    spec:
      domain:
        devices:
          disks:
            - name: osdisk
              disk:
                bus: virtio
            - name: cloudinitdisk
              disk:
                bus: virtio
            - name: datadisk
              disk:
                bus: virtio
          interfaces:
          - name: default
            masquerade: {}
            model: virtio
        resources:
          requests:
            memory: 2048M
      networks:
      - name: default
        pod: {}
      volumes:
        - name: osdisk
          containerDisk:
            image: quay.io/containerdisks/centos-stream:9
        - name: cloudinitdisk
          cloudInitNoCloud:
            userData: |-
              #cloud-config
              user: centos
              password: centos
              chpasswd: { expire: False }
        - name: datadisk
          persistentVolumeClaim:
            claimName: pvc-iscsi-1g
EOF
```

```text
virtualmachine.kubevirt.io/vm-disk created
```

仮想マシンを確認する。

```sh
kubectl get vm
```

```text
NAME     AGE   STATUS    READY
vm-disk   60s   Stopped   False
```

仮想マシンを起動する。

```sh
kubectl virt start vm-disk
```

```text
VM vm-disk was scheduled to start
```

仮想マシンのインスタンスを確認する。

```sh
kubectl get vmi -o wide
```

```text
NAME      AGE   PHASE     IP              NODENAME              READY   LIVE-MIGRATABLE   PAUSED
vm-disk   69s   Running   172.17.51.143   worker02.home.local   True    False
```

仮想マシンのディスクを確認する。

```sh
LANG=C lsblk
```

```text
NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
vda    252:0    0   10G  0 disk
`-vda1 252:1    0   10G  0 part /
vdb    252:16   0    1M  0 disk
vdc    252:32   0  900M  0 disk
```

```sh
kubectl describe pvc pvc-iscsi-1g
```

PersistentVolumeClaim を確認する。

```text
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
Used By:       virt-launcher-vm-disk-mb5tn
Events:        <none>
```
