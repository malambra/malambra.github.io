---
title: "Jugando con Kind - Creando clustrer K8s."
excerpt_separator: "<!--more-->"
categories:
- SysAdmin
- DevOps
tags:
- tips
- linux
- Kubernetes
---
# Que es Kind
Según se indica en su web, kind es una herramienta para ejecutar clústers de Kubernetes en local, usando como "nodos", contenedores Docker.
kind se diseñó principalmente para probar Kubernetes, pero se puede usar para desarrollo local o CI.
<!--more-->

# Instalación
No lo explicaría mejor que en la web, así que como pincelada:
Necesitas **go** y **docker**

```bash
curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.17.0/kind-linux-amd64
chmod +x ./kind
sudo mv ./kind /usr/local/bin/kind
```

Consulta: (https://kind.sigs.k8s.io/docs/user/quick-start/)



# Uso
El uso y sus múltiples opciones se detallan en la web, aquí daré las pinceladas del uso rápido que hago yo.
Básicamente quiero levantar un cluster lo más rápido posible, tener conexión a el y empezar a probar lo que necesito. Para esto, dispongo de un **Makefile** que me permite lanzar estas opciones lo más rápido posible,

## Configuración

En el directorio de trabajo que uso para lanzar **kind** tengo lo siguiente:
```bash
.
├── kind-config.yaml
├── kubeconfig.yml
└── Makefile
```

### kubeconfig.yml

Se construye en cada ejecución y es el fichero de conexión a nuestro cluster.

### kind-config.yaml

Es el fichero que define que cluster queremos, número de nodos etc...

https://kind.sigs.k8s.io/docs/user/quick-start/#configuring-your-kind-cluster

Yo uso por defecto esta conf: (1 Control-Plane y 3 Workers)

```bash
# four node (three workers) cluster config
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
- role: control-plane
- role: worker
- role: worker
- role: worker
```

### Makefile

Este fichero define las opciones que podemos lanzar:

**all:** Opción por defeto, crea el cluster y descarga el kubeconfig

**clean:** Limpia el cluster que hemos creado.

**info:** Obtiene info de salud del cluster.

**re:** Recrea el cluster, lo elimina y lo lanza de nuevo.

```
all:
	kind create cluster --config kind-config.yaml
	kind get kubeconfig > kubeconfig.yml

clean:
	kind delete cluster
	rm -f kubeconfig.yml

info:
	kubectl get nodes
	kubectl get pods -o wide
	kubectl get svc

re: clean all

.PHONY: all clean info re
```

## Ejecución
Aquí las muestras de ejecución y el tiempo empleado en un portatil. (i7, 16gb, ssd)

### Creación de cluster
Algo más de 43seg.

```bash
malambra@xxx:~/GITHUB/homelab/create_Kind_K8s $ time make 
kind create cluster --config kind-config.yaml
Creating cluster "kind" ...
 ✓ Ensuring node image (kindest/node:v1.25.3) 🖼
 ✓ Preparing nodes 📦 📦 📦 📦  
 ✓ Writing configuration 📜 
 ✓ Starting control-plane 🕹️ 
 ✓ Installing CNI 🔌 
 ✓ Installing StorageClass 💾 
 ✓ Joining worker nodes 🚜 
Set kubectl context to "kind-kind"
You can now use your cluster with:

kubectl cluster-info --context kind-kind

Have a question, bug, or feature request? Let us know! https://kind.sigs.k8s.io/#community 🙂
kind get kubeconfig > kubeconfig.yml

real	0m43,186s
user	0m5,158s
sys	0m2,251s
```

### Eliminación de cluster
Menos de 2 seg.

```bash
malambra@xxx:~/GITHUB/homelab/create_Kind_K8s $ time make clean
kind delete cluster
Deleting cluster "kind" ...
rm -f kubeconfig.yml

real	0m1,367s
user	0m0,099s
sys	0m0,065s
```

### Reconstruir cluster
Dado que el tiempo de eliminación es despreciable, tarda lo mismo que la creación.

### Información del cluster
Si se quiere otro tipo de información, basta con editar el Makefile.

```bash
malambra@xxxx:~/GITHUB/homelab/create_Kind_K8s $ make info
kubectl get nodes
NAME                 STATUS   ROLES           AGE   VERSION
kind-control-plane   Ready    control-plane   81s   v1.25.3
kind-worker          Ready    <none>          61s   v1.25.3
kind-worker2         Ready    <none>          61s   v1.25.3
kind-worker3         Ready    <none>          61s   v1.25.3
kubectl get pods -o wide
No resources found in default namespace.
kubectl get svc
NAME         TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   80s
```

### Ejemplos
Al finalizar la creación tenemos un cluster funcional, al que estamos conectados.

```bash
malambra@xxx:~/GITHUB/homelab/create_Kind_K8s $ k get nodes
NAME                 STATUS   ROLES           AGE     VERSION
kind-control-plane   Ready    control-plane   2m49s   v1.25.3
kind-worker          Ready    <none>          2m29s   v1.25.3
kind-worker2         Ready    <none>          2m29s   v1.25.3
kind-worker3         Ready    <none>          2m29s   v1.25.3
```

```bash
malambra@xxx:~/GITHUB/homelab/create_Kind_K8s $ k get pods -A
NAMESPACE            NAME                                         READY   STATUS    RESTARTS   AGE
kube-system          coredns-565d847f94-n2l6d                     1/1     Running   0          3m17s
kube-system          coredns-565d847f94-qk7xw                     1/1     Running   0          3m17s
kube-system          etcd-kind-control-plane                      1/1     Running   0          3m30s
kube-system          kindnet-8r4tf                                1/1     Running   0          3m13s
kube-system          kindnet-9ddwf                                1/1     Running   0          3m13s
kube-system          kindnet-n5dzd                                1/1     Running   0          3m17s
kube-system          kindnet-rnxcm                                1/1     Running   0          3m13s
kube-system          kube-apiserver-kind-control-plane            1/1     Running   0          3m30s
kube-system          kube-controller-manager-kind-control-plane   1/1     Running   0          3m30s
kube-system          kube-proxy-7bsgb                             1/1     Running   0          3m17s
kube-system          kube-proxy-8f8xz                             1/1     Running   0          3m13s
kube-system          kube-proxy-8jqxt                             1/1     Running   0          3m13s
kube-system          kube-proxy-ddrmr                             1/1     Running   0          3m13s
kube-system          kube-scheduler-kind-control-plane            1/1     Running   0          3m30s
local-path-storage   local-path-provisioner-684f458cdd-j47vs      1/1     Running   0          3m17s
```

# Conclusión
Ya no hay escusa para no jugar con un cluster, si nos cuesta menos de 1min tenerlo listo¡¡¡