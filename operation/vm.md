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

## インスタンス

### 種別

事前にクラスタワイドに種別が設定されている。

```sh
kubectl get virtualmachineclusterinstancetypes | head -n 5
```

```text
NAME             AGE
cx1.2xlarge      10d
cx1.2xlarge1gi   10d
cx1.4xlarge      10d
cx1.4xlarge1gi   10d
```

設定内容を確認する。

```sh
kubectl describe virtualmachineclusterinstancetypes u1.small
```

```text
Name:         u1.small
Namespace:
Labels:       app.kubernetes.io/component=kubevirt
              app.kubernetes.io/managed-by=virt-operator
              instancetype.kubevirt.io/class=general.purpose
              instancetype.kubevirt.io/common-instancetypes-version=v1.4.0
              instancetype.kubevirt.io/cpu=1
              instancetype.kubevirt.io/icon-pf=pficon-server-group
              instancetype.kubevirt.io/memory=2Gi
              instancetype.kubevirt.io/size=small
              instancetype.kubevirt.io/vendor=kubevirt.io
              instancetype.kubevirt.io/version=1
Annotations:  instancetype.kubevirt.io/description:
                The U Series is quite neutral and provides resources for
                general purpose applications.

                *U* is the abbreviation for "Universal", hinting at the universal
                attitude towards workloads.

                VMs of instance types will share physical CPU cores on a
                time-slice basis with other VMs.
              instancetype.kubevirt.io/displayName: General Purpose
              kubevirt.io/generation: 3
              kubevirt.io/install-strategy-identifier: db0d6e1f2860442b4f142ad2879ed541632819b8
              kubevirt.io/install-strategy-registry: quay.io/kubevirt
              kubevirt.io/install-strategy-version: v1.6.2
API Version:  instancetype.kubevirt.io/v1beta1
Kind:         VirtualMachineClusterInstancetype
Metadata:
  Creation Timestamp:  2025-10-26T02:30:17Z
  Generation:          1
  Resource Version:    558430
  UID:                 0cd908d4-4f71-4e36-89b2-e2b3fbe97b73
Spec:
  Cpu:
    Guest:  1
  Memory:
    Guest:  2Gi
Events:     <none>
```

種別を指定して仮想マシンをデプロイする。

```sh
cat | kubectl apply -f - <<EOF
apiVersion: kubevirt.io/v1
kind: VirtualMachine
metadata:
  name: vm-u1-small
spec:
  runStrategy: Halted
  instancetype:
    name: u1.small
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
          interfaces:
          - name: default
            masquerade: {}
            model: virtio
      networks:
      - name: default
        pod: {}
      volumes:
        - name: osdisk
          containerDisk:
            image: quay.io/kubevirt/cirros-container-disk-demo
        - name: cloudinitdisk
          cloudInitNoCloud:
            userData: |-
              Hi.
EOF
```

```text
virtualmachine.kubevirt.io/vm-u1-small created
```

仮想マシンを確認する。

```sh
kubectl get vm
```

```text
NAME          AGE   STATUS    READY
vm-u1-small   25s   Stopped   False
```

仮想マシンを起動する。

```sh
kubectl virt start vm-u1-small
```

```text
VM vm-u1-small was scheduled to start
```

仮想マシンのインスタンスを確認する。

```sh
kubectl get vmi -o wide
```

```text
NAME          AGE   PHASE     IP              NODENAME              READY   LIVE-MIGRATABLE   PAUSED
vm-u1-small   21s   Running   172.17.51.188   worker02.home.local   True    True
```

仮想マシンのリソースを確認する。

```sh
kubectl get vmi vm-u1-small -o jsonpath='CPU={.spec.domain.cpu.sockets}{"\n"}Memory={.spec.domain.memory.guest}{"\n"}'
```

```text
CPU=1
Memory=2Gi
```

名前空間に新しい種別を作成する。

```sh
cat | kubectl apply -f - <<EOF
apiVersion: instancetype.kubevirt.io/v1beta1
kind: VirtualMachineInstancetype
metadata:
  name: c2m1
spec:
  cpu:
    guest: 2
  memory:
    guest: 1Gi
EOF
```

```text
virtualmachineinstancetype.instancetype.kubevirt.io/c2m1 created
```

種別を確認する。

```sh
kubectl get vmfs
```

```text
NAME   AGE
c2m1   37s
```

種別を指定して仮想マシンをデプロイする。

```sh
cat | kubectl apply -f - <<EOF
apiVersion: kubevirt.io/v1
kind: VirtualMachine
metadata:
  name: vm-c2m1
spec:
  runStrategy: Halted
  instancetype:
    name: c2m1
    kind: virtualMachineInstancetype
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
          interfaces:
          - name: default
            masquerade: {}
            model: virtio
      networks:
      - name: default
        pod: {}
      volumes:
        - name: osdisk
          containerDisk:
            image: quay.io/kubevirt/cirros-container-disk-demo
        - name: cloudinitdisk
          cloudInitNoCloud:
            userData: |-
              Hi.
EOF
```

```text
virtualmachine.kubevirt.io/vm-c2m1 created
```

仮想マシンを確認する。

```sh
kubectl get vm
```

```text
NAME      AGE   STATUS    READY
vm-c2m1   9s    Stopped   False
```

仮想マシンを起動する。

```sh
kubectl virt start vm-c2m1
```

```text
VM vm-c2m1 was scheduled to start
```

仮想マシンのインスタンスを確認する。

```sh
kubectl get vmi -o wide
```

```text
NAME      AGE   PHASE     IP              NODENAME              READY   LIVE-MIGRATABLE   PAUSED
vm-c2m1   10s   Running   172.17.51.186   worker02.home.local   True    True
```

仮想マシンのリソースを確認する。

```sh
kubectl get vmi vm-c2m1 -o jsonpath='CPU={.spec.domain.cpu.sockets}{"\n"}Memory={.spec.domain.memory.guest}{"\n"}'
```

```text
CPU=2
Memory=1Gi
```

### 推奨設定

事前にクラスタワイドに推奨設定が設定されている。

```sh
kubectl get virtualmachineclusterpreferences | head -n 5
```

```text
NAME                       AGE
alpine                     10d
centos.stream10            10d
centos.stream10.desktop    10d
centos.stream9             10d
```

設定内容を確認する。

```sh
kubectl describe virtualmachineclusterpreferences cirros
```

```text
Name:         cirros
Namespace:
Labels:       app.kubernetes.io/component=kubevirt
              app.kubernetes.io/managed-by=virt-operator
              instancetype.kubevirt.io/common-instancetypes-version=v1.4.0
              instancetype.kubevirt.io/os-type=linux
              instancetype.kubevirt.io/vendor=kubevirt.io
Annotations:  iconClass: icon-cirros
              kubevirt.io/generation: 3
              kubevirt.io/install-strategy-identifier: db0d6e1f2860442b4f142ad2879ed541632819b8
              kubevirt.io/install-strategy-registry: quay.io/kubevirt
              kubevirt.io/install-strategy-version: v1.6.2
              openshift.io/display-name: Cirros
              openshift.io/documentation-url: "https://github.com/kubevirt/common-instancetypes"
              openshift.io/provider-display-name: KubeVirt
              openshift.io/support-url: "https://github.com/kubevirt/common-instancetypes/issues"
              tags: hidden,kubevirt,cirros
API Version:  instancetype.kubevirt.io/v1beta1
Kind:         VirtualMachineClusterPreference
Metadata:
  Creation Timestamp:  2025-10-26T02:30:18Z
  Generation:          1
  Resource Version:    558449
  UID:                 c2df0de3-9bb5-49a2-863f-77a32386b08f
Spec:
  Annotations:
    vm.kubevirt.io/os:  linux
  Devices:
    Preferred Disk Bus:         virtio
    Preferred Interface Model:  virtio
    Preferred Rng:
  Requirements:
    Cpu:
      Guest:  1
    Memory:
      Guest:  256Mi
Events:       <none>
```

推奨設定を指定して仮想マシンをデプロイする。

```sh
cat | kubectl apply -f - <<EOF
apiVersion: kubevirt.io/v1
kind: VirtualMachine
metadata:
  name: vm-cirros
spec:
  runStrategy: Halted
  preference:
    name: cirros
  template:
    spec:
      domain:
        devices:
          disks:
            - name: osdisk
            - name: cloudinitdisk
          interfaces:
          - name: default
            masquerade: {}
        resources:
          requests:
            memory: 512Mi
      networks:
      - name: default
        pod: {}
      volumes:
        - name: osdisk
          containerDisk:
            image: quay.io/kubevirt/cirros-container-disk-demo
        - name: cloudinitdisk
          cloudInitNoCloud:
            userData: |-
              Hi.
EOF
```

```text
virtualmachine.kubevirt.io/vm-cirros created
```

仮想マシンを確認する。

```sh
kubectl get vm
```

```text
NAME        AGE   STATUS    READY
vm-cirros   22s   Stopped   False
```

仮想マシンを起動する。

```sh
kubectl virt start vm-cirros
```

```text
VM vm-cirros was scheduled to start
```

仮想マシンのインスタンスを確認する。

```sh
kubectl get vmi -o wide
```

```text
NAME        AGE   PHASE     IP              NODENAME              READY   LIVE-MIGRATABLE   PAUSED
vm-cirros   12s   Running   172.17.51.140   worker02.home.local   True    True
```

仮想マシンのリソースを確認する。

```sh
kubectl get vmi vm-cirros -o jsonpath='CPU={.spec.domain.cpu.sockets}{"\n"}Memory={.spec.domain.memory.guest}{"\n"}'
```

```text
CPU=1
Memory=512Mi
```

名前空間に新しい推奨設定を作成する。

```sh
cat | kubectl apply -f - <<EOF
apiVersion: instancetype.kubevirt.io/v1beta1
kind: VirtualMachinePreference
metadata:
  name: c2m2
spec:
  devices:
    preferredDiskBus: virtio
    preferredInterfaceModel: virtio
  requirements:
    cpu:
      guest:  2
    memory:
      guest:  2Gi
EOF
```

```text
virtualmachinepreference.instancetype.kubevirt.io/c2m2 created
```

推奨設定を確認する。

```sh
kubectl get vmprefs
```

```text
NAME   AGE
c2m2   9s
```

推奨設定を指定して仮想マシンをデプロイする。

```sh
cat | kubectl apply -f - <<EOF
apiVersion: kubevirt.io/v1
kind: VirtualMachine
metadata:
  name: vm-c2m2
spec:
  runStrategy: Halted
  preference:
    name: c2m2
    kind: virtualMachinepreference
  template:
    spec:
      domain:
        devices:
          disks:
            - name: osdisk
            - name: cloudinitdisk
          interfaces:
          - name: default
            masquerade: {}
        resources:
          requests:
            cpu: 2
            memory: 2Gi
      networks:
      - name: default
        pod: {}
      volumes:
        - name: osdisk
          containerDisk:
            image: quay.io/kubevirt/cirros-container-disk-demo
        - name: cloudinitdisk
          cloudInitNoCloud:
            userData: |-
              Hi.
EOF
```

```text
virtualmachine.kubevirt.io/vm-c2m2 created
```

仮想マシンを確認する。

```sh
kubectl get vm
```

```text
NAME      AGE   STATUS    READY
vm-c2m2   22s   Stopped   False
```

仮想マシンを起動する。

```sh
kubectl virt start vm-c2m2
```

```text
VM vm-c2m2 was scheduled to start
```

仮想マシンのインスタンスを確認する。

```sh
kubectl get vmi -o wide
```

```text
NAME      AGE   PHASE     IP               NODENAME              READY   LIVE-MIGRATABLE   PAUSED
vm-c2m2   10s   Running   172.17.255.157   worker01.home.local   True    True
```

仮想マシンのリソースを確認する。

```sh
kubectl get vmi vm-c2m2 -o jsonpath='CPU={.spec.domain.cpu.sockets}{"\n"}Memory={.spec.domain.memory.guest}{"\n"}'
```

```text
CPU=2
Memory=2Gi
```

## ネットワーク

### ポート

公開するポートを指定する。

公開するポート番号を指定して仮想マシンをデプロイする。

```sh
cat | kubectl apply -f - <<EOF
apiVersion: kubevirt.io/v1
kind: VirtualMachine
metadata:
  name: vm-http
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
          interfaces:
          - name: default
            masquerade: {}
            model: virtio
            ports:
              - name: http
                port: 80
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
              packages:
                - nginx
              runcmd:
                - [ systemctl, enable, --now, nginx ]
EOF
```

```text
virtualmachine.kubevirt.io/vm-http created
```

仮想マシンを確認する。

```sh
kubectl get vm
```

```text
NAME      AGE   STATUS    READY
vm-http   29s   Stopped   False
```

仮想マシンを起動する。

```sh
kubectl virt start vm-http
```

```text
VM vm-http was scheduled to start
```

仮想マシンのインスタンスを確認する。

```sh
kubectl get vmi -o wide
```

```text
NAME      AGE   PHASE     IP    NODENAME              READY   LIVE-MIGRATABLE   PAUSED
vm-http   76s   Running         worker02.home.local   True    True
```

サービスとして公開する。

```sh
kubectl virt expose vm vm-http --name vm-http-80 --port 20080 --target-port 80 --type LoadBalancer
```

```text
Service vm-http-80 successfully created for vm vm-http
```

サービスを確認する。

```sh
kubectl get service vm-http-80
```

```text
NAME         TYPE           CLUSTER-IP     EXTERNAL-IP    PORT(S)           AGE
vm-http-80   LoadBalancer   10.107.15.76   172.16.0.101   20080:31333/TCP   15s
```

接続確認する。

```sh
curl -s http://172.16.0.101:20080
```

HTML が返却される。

SSH 接続はできない。

```sh
kubectl virt ssh vm/vm-http
```

```text
Internal error occurred: dialing VM: dial tcp 172.17.51.181:22: connect: connection refused
Connection closed by UNKNOWN port 65535
exit status 255
```

### ネットワークポリシー

アクセス制御を指定する。

すべての接続を拒否する。

```sh
cat | kubectl apply -f - <<EOF
kind: NetworkPolicy
apiVersion: networking.k8s.io/v1
metadata:
  name: deny-all
spec:
  podSelector: {}
  ingress: []
EOF
```

```text
networkpolicy.networking.k8s.io/deny-all created
```

接続を確認する。

```sh
curl -s --connect-timeout 5 http://172.16.0.101:20080
```

タイムアウトが発生する。

HTTP の接続を許可する。

```sh
cat | kubectl apply -f - <<EOF
kind: NetworkPolicy
apiVersion: networking.k8s.io/v1
metadata:
  name: allow-http
spec:
  podSelector: {}
  ingress:
  - ports:
    - protocol: TCP
      port: 80
EOF
```

```text
networkpolicy.networking.k8s.io/allow-http created
```

接続を確認する。

```sh
curl -s http://172.16.0.101:20080
```

HTML が返却される。

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
