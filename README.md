
##  NATS 설치

```
-- 0 : iSCSI 설치

- account
devNats
password10040314

- base64
ZGV2TmF0cw==
YmVtaWx5MTAwNDAzMTQ=

- iSCSI
dev-nats-01
dev-nats-02
dev-nats-03


- iqn
iqn.2000-01.com.synology:dev-nats-01
iqn.2000-01.com.synology:dev-nats-02
iqn.2000-01.com.synology:dev-nats-03

- LUN
dev-nats-01-lun
dev-nats-02-lun
dev-nats-03-lun

-- 1 : 네이밍 

- namespace : messanger-nats

- helm name :  dev-nats

-- 2 : 네임 스페이스 등록

- namespace crerate 
kubectl create namespace messanger-nats

-- 3 : NAS 로그인 체크
- NAS login

- setting
sudo iscsiadm -m discovery -t st -p 127.0.0.1

sudo iscsiadm -m node -T iqn.2000-01.com.synology:dev-nats-01 -p 127.0.0.1 --o update --name node.session.auth.authmethod --value CHAP --name node.session.auth.username --value devNats --name node.session.auth.password --value password10040314
sudo iscsiadm -m node -T iqn.2000-01.com.synology:dev-nats-02 -p 127.0.0.1 --o update --name node.session.auth.authmethod --value CHAP --name node.session.auth.username --value devNats --name node.session.auth.password --value password10040314
sudo iscsiadm -m node -T iqn.2000-01.com.synology:dev-nats-03 -p 127.0.0.1 --o update --name node.session.auth.authmethod --value CHAP --name node.session.auth.username --value devNats --name node.session.auth.password --value password10040314

- login
sudo iscsiadm -m node -T iqn.2000-01.com.synology:dev-nats-01 -p 127.0.0.1 --login
sudo iscsiadm -m node -T iqn.2000-01.com.synology:dev-nats-02 -p 127.0.0.1 --login
sudo iscsiadm -m node -T iqn.2000-01.com.synology:dev-nats-03 -p 127.0.0.1 --login

- logout
sudo iscsiadm -m node -T iqn.2000-01.com.synology:dev-nats-01 -p 127.0.0.1 --logout
sudo iscsiadm -m node -T iqn.2000-01.com.synology:dev-nats-02 -p 127.0.0.1 --logout
sudo iscsiadm -m node -T iqn.2000-01.com.synology:dev-nats-03 -p 127.0.0.1 --logout

- session status
sudo iscsiadm -m session

- session detail 
sudo iscsiadm -m session --print=3

- lun 번호 확인
(cd /dev/disk/by-path; ls -l *iscsi* | awk '{FS=" "; print $9 " " $10 " " $11}')

-- 4 : secret, pv, pvc 등록

kubectl apply -f nats-pv.yaml

-- 5 : nats 설치(with helm)

helm install dev-benily-nats -f values.yaml . --namespace messanger-nats

- helm 설정 정보
helm status dev-benily-nats --namespace messanger-nats


- 로컬에서 접속

  kubectl exec -n messanger-nats -it deployment/dev-nats-box -- /bin/sh -l

  nats-box:~# nats-sub test &
  nats-box:~# nats-pub test hi
  nats-box:~# nc dev-nats 4222


- NodePort IP/Port 얻기
kubectl get nodes --namespace messanger-nats -o jsonpath="{.items[0].status.addresses[0].address}"

kubectl get --namespace messanger-nats -o jsonpath="{.spec.ports[0].nodePort}" services bai-db-postgresql-postgresql-ha-pgpool

- 외부에서 접근
192.168.0.61:30030

- metrics 정보 얻기
curl 192.168.0.60:30033/metrics

- 서비스 확인
kubectl get all --namespace messanger-nats

