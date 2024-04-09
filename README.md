# Домашнее задание к занятию «Как работает сеть в K8s»

### Цель задания

Настроить сетевую политику доступа к подам.

### Чеклист готовности к домашнему заданию

1. Кластер K8s с установленным сетевым плагином Calico.

### Инструменты и дополнительные материалы, которые пригодятся для выполнения задания

1. [Документация Calico](https://www.tigera.io/project-calico/).
2. [Network Policy](https://kubernetes.io/docs/concepts/services-networking/network-policies/).
3. [About Network Policy](https://docs.projectcalico.org/about/about-network-policy).

-----

### Задание 1. Создать сетевую политику или несколько политик для обеспечения доступа

1. Создать deployment'ы приложений frontend, backend и cache и соответсвующие сервисы.
2. В качестве образа использовать network-multitool.
3. Разместить поды в namespace App.
4. Создать политики, чтобы обеспечить доступ frontend -> backend -> cache. Другие виды подключений должны быть запрещены.
5. Продемонстрировать, что трафик разрешён и запрещён.

**Подготовим ВМ на Yandex Cloud**

<p align="center">
    <img width="1200 height="600" src="/img/np_ter_inv.png">
</p>

<p align="center">
    <img width="1200 height="600" src="/img/np_yc_vm.png">
</p>

**Предварительно установим на ноды python3 и разрешим маршрутизацию трафика**
<br>
[Playbook](playbook.yml)

<p align="center">
    <img width="1200 height="600" src="/img/np_playbook.png">
</p>

**Развернём кластер при помощи Kubespray**

- Подготовим файл Inventory 
```bash
cp -rfp inventory/sample inventory/k8s
```

- Для генерирования конфигурации обьявим переменную IPS, с адресами ВМ
```bash
declare -a IPS=(51.250.66.4 178.154.204.76 178.154.204.25 178.154.220.33)
```

- Сгенерируем конфигурацию
```bash
CONFIG_FILE=inventory/k8s/hosts.yaml python3 contrib/inventory_builder/inventory.py ${IPS[@]}
```

- Для успешного деплоя Kubespray на ВМ YC, добавим в файл - kubespray/inventory/k8s/hosts.yaml, локальные адреса ВМ

```yaml
all:
  hosts:
    node1:
      ansible_host: 51.250.66.4
      ip: 10.0.1.27
      access_ip: 10.0.1.27
    node2:
      ansible_host: 178.154.204.76
      ip: 10.0.1.13
      access_ip: 10.0.1.13
    node3:
      ansible_host: 178.154.204.25
      ip: 10.0.1.10
      access_ip: 10.0.1.10
    node4:
      ansible_host: 178.154.220.33
      ip: 10.0.1.17
      access_ip: 10.0.1.17
  children:
    kube_control_plane:
      hosts:
        node1:
        node2:
    kube_node:
      hosts:
        node1:
        node2:
        node3:
        node4:
    etcd:
      hosts:
        node1:
        node2:
        node3:
    k8s_cluster:
      children:
        kube_control_plane:
        kube_node:
    calico_rr:
      hosts: {}
```

- Для подключения к ВМ, добавим в файл - kubespray/ansible.cfg:
```cfg
remote_user=root
ansible_ssh_private_key_file=/root/.ssh/id_ed25519
```

- Результат:

<p align="center">
    <img width="1200 height="600" src="/img/np_kubespray.png">
</p>

<p align="center">
    <img width="1200 height="600" src="/img/np_playbooknp_get_nodes.png">
</p>

<p align="center">
    <img width="1200 height="600" src="/img/np_get_pods_kube_sys.png">
</p>

**Подготовим конфиги подов, сервисов, политик  скопируем на мастер-ноду**
<br>
[frontend](frontend.yml)
<br>
[backend](backend.yml)
<br>
[cache](cache.yml)
<br>
[policy-namespace](policy-nm-default.yml)
<br>
[policy-backend](policy-backend.yml)
<br>
[policy-cache](policy-cache.yml)
<br>

<p align="center">
    <img width="1200 height="600" src="/img/np_cp_deploments.png">
</p>

<p align="center">
    <img width="1200 height="600" src="/img/np_cp_polices.png">
</p>

**Создадим namespace, pods, policy**

<p align="center">
    <img width="1200 height="600" src="/img/np_create_nm_deployments.png">
</p>

<p align="center">
    <img width="1200 height="600" src="/img/np_get_polices.png">
</p>

<p align="center">
    <img width="1200 height="600" src="/img/np_get_polices.png">
</p>

<p align="center">
    <img width="1200 height="600" src="/img/np_get_pods.png">
</p>

<p align="center">
    <img width="1200 height="600" src="/img/np_get_pods_svc.png">
</p>

**Проверим работу политик**

<p align="center">
    <img width="1200 height="600" src="/img/np_curl_deployments.png">
</p>

**Обеспечен доступ (80 и 443 порты) frontend -> backend -> cache. Другие виды подключений запрещены.**

### Правила приёма работы

1. Домашняя работа оформляется в своём Git-репозитории в файле README.md. Выполненное домашнее задание пришлите ссылкой на .md-файл в вашем репозитории.
2. Файл README.md должен содержать скриншоты вывода необходимых команд, а также скриншоты результатов.
3. Репозиторий должен содержать тексты манифестов или ссылки на них в файле README.md.
