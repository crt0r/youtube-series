# CouchDB Cluster on Kubernetes
> Список команд к видео [CouchDB Кластер на Kubernetes](https://youtu.be/2CKIVF748hM).

## Установка `git`, `nfs-common`, `k3s`, `helm`
1. git
    ```bash
    sudo apt update && sudo apt install git nfs-common
    ```

1. k3s
    ```bash
    curl -sfL https://get.k3s.io | sh -
    ```

    ```bash
    cat << EOF | bash
    sudo su -c '
    echo K3S_KUBECONFIG_MODE=\"644\" >> /etc/systemd/system/k3s.service.env && sudo systemctl restart k3s
    '
    EOF
    ```
    > 👆 На проде так делать не стоит. Это небезопасно, т. к. конфиг становится читаемым для всех пользователей и групп.

1. helm
    ```bash
    sudo snap install --classic helm
    ```

## Получение репо
```bash
git clone https://github.com/crt0r/youtube-series.git
```

## namespace
```bash
kubectl create namespace couchdb-cluster
```

## Кастомный NFS "поставщик"
```bash
export KUBECONFIG=/etc/rancher/k3s/k3s.yml
```

```bash
snap run helm repo add nfs-subdir-external-provisioner https://kubernetes-sigs.github.io/nfs-subdir-external-provisioner/
```

> Поменяйте адрес NFS сервера и базового пути на свои.
```bash
snap run helm install nfs-storage nfs-subdir-external-provisioner/nfs-subdir-external-provisioner \
--set storageClass.name=nfs-storage --set storageClass.provisionerName=cluster.local/nfs-storage \
--set nfs.server=nas.crt0r.internal --set storageClass.volumeBindingMode=WaitForFirstConsumer \
--set storageClass.reclaimPolicy=Retain --set nfs.path=/mnt/NAS/cluster-storage --namespace couchdb-cluster
```

```bash
kubectl get pods --namespace couchdb-cluster -w
```

## Secret
```bash
kubectl create secret generic --from-literal=COUCHDB_USER=admin \
                              --from-literal=COUCHDB_PASSWORD=verysecurepassword \
                              --namespace couchdb-cluster couchdb-cluster
```

## Применение конфигов
> В директории с проектом

```bash
kubectl apply -f configs/statefulsets/node.yml
```

```bash
kubectl apply -f configs/services/couchdb.yml
```

## Мониторинг состояния подов

```bash
kubectl get pods --namespace couchdb-cluster -w
```
