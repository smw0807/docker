```yaml
# pod.yml
apiVersion: v1
kind: Pod
metadata:
  name: pod1
spec:
  volumes: ##「표5 Volume v1 core」참고
    - name: pvc1
      persistentVolumeClaim:
        claimName: data1 ## <-- PVC의 이름 설정
  containers:
    - name: ubuntu
      image: ubuntu:16.04
      volumeMounts: ## 「표6 VolumeMount v1 core」참고
        - name: pvc1
          mountPath: /mnt ## <-- 컨테이너 상 마운트 경로
      command: ['/usr/bin/tail', '-f', '/dev/null']
```

```yaml
# pvc.yml
apiVersion: v1 ##「표１PersistentVolumeClaim v1 core」
kind: PersistentVolumeClaim
metadata: ##「표2 ObjectMeta v1 meta」
  name: data1
spec: ##「표3 PersistentVolumeClaimSpec v1 core」
  accessModes:
    - ReadWriteOnce
  storageClassName: standard
  resources:
    requests:
      storage: 2Gi
```

pvc를 사용하는 파드의 yaml에는 volumeMounts를 기술하여 pvc1을 컨테이너 파일 시스템 /mnt에 마운트 한다.  
그리고 pvc1은 persistentVolumeClaim의 PVC 이름 data1을 지정하고 있다.

퍼시스턴트 볼륨을 요구하는 PersistentVolumeClaim의 yaml에는 data1의 스토리지 클래스를 standard로 지정하고 있다.  
그 밑에는 스토리지 용량으로 2기가를 요구하고 있다.
