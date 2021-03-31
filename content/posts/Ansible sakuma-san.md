---
title: "ジョブの投入"
date: 2020-12-22T12:11:23+09:00
draft: true
weight: 7
---

## マニュフェストの作成


* マニュフェスト(yamlファイル)を作成します。
```
$ vim test.yaml
```
test.yaml
```
apiVersion: v1
kind: Pod
metadata:
 name: pytorch-job
spec:
 containers:
  - name: pytorch-container
    image: 192.168.100.xxx:xxxx/library/pytorch:20.09-py3
    command: ["/bin/sh"]
    args: ["-c", "python /workspace/examples/upstream/mnist/main.py"]
    resources:
      limits:
        nvidia.com/gpu: 1
 imagePullSecrets:
    - name : nvidia
 restartPolicy: Never
```

* ジョブを投入してみる
```
$ kubectl create -f test.yaml
```

* 動作確認
```
$ kubectl get pod
```
![](/images/job.PNG?height=300px)

* Rancherで実際に動いているか確認してみる
![](/images/rancher_pod.PNG?height=300px)
![](/images/rancher_log.PNG?height=500px)

* もし何度か試したい場合はpodを消す必要がある
```
$ kubectl delete pod pytorch-job
```



##



