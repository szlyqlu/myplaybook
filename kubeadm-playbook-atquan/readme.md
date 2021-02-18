# 说明
2021-2-18更新
- 增加了ha架构的部署流程，但ha架构中控制面的添加key只有两个小时，目前没有控制面加入的角色
- 针对性的修改了host.yml一些参数
- 添加了一个flanneld的描述文件

2021-1-20更新
- 增加了后期添加node的功能
- 修正了site.yml空白
- 修正了host.yml缺少apiserverip这个参数
- 增加了host.yml中的addnode部分

该playbook是用kubeadm来部署一个无HA的k8s集群

具有以下特点
- 用的yum repo来自阿里云，如果想要更换，可以修改roles/common/files/下的repo文件
- 用的是k8s镜像来自阿里云仓库，如果需要更换可以修改host.yml中imagehub参数
- 没有部署网络插件，网络插件请查看 https://kubernetes.io/docs/concepts/cluster-administration/addons/

# 参数说明
所有参数均存于host.yml文件，是个yml格式的inventory文件
```
master:
  hosts: # master主机地址，可以多个，多个是基于ha架构的，所以以下的vars中hacluster请写yes
    192.168.30.52:
node:
  hosts: # node主机地址，可以填多个
    192.168.30.53:
addnode: 这个部分用于追加node的场景，如果用于初始化，没必要添加
  hosts:
    192.168.30.54:
all:
  children:
    master:
    node:
  vars:
    repolist: # repo文件名，用来给主机使用yum安装kubelet，docker等一些工具，可以直接修改文件或是换自己的repo文件
      - { src: "kubernetes.repo", dest: "/etc/yum.repos.d/kubernetes.repo" }
      - { src: "docker-ce.repo", dest: "/etc/yum.repos.d/docker-ce.repo" }
    k8sver: 1.18.2 # k8s版本，目前我只测试过1.18.2版本
    etc_hosts: # 把集群信息写入/etc/hosts的列表，建议所有节点都写入（特别是多网卡主机）
      - "192.168.30.52 k8s1"
      - "192.168.30.53 k8s2"
    imghub: registry.aliyuncs.com/google_containers # 镜像仓库地址
    apiserverip: 192.168.30.52 # apiserver的地址，用来node加入集群用的，当你使用ha架构的时候这个ip为vip，需要你使用nginx，haproxy，F5此类的elb实现，转发规则为vip:6443->masterip:6443
    podcidr: 172.10.0.0/16 # pod的网段
    hacluster: "no" # 如果你要部署的是ha架构的，那就填yes。
```
# 执行方法
在该playbook目录下，用以下命令执行
```
ansible-playbook -i host.yml site.yml -uroot -k
```