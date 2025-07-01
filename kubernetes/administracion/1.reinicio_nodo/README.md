# 🧪 Kubernetes Lab 1: Reinicio Controlado de un Nodo

Este laboratorio documenta el proceso para realizar un reinicio controlado de un nodo en un clúster Kubernetes en DigitalOcean, asegurando que los pods se reprogramen correctamente.

---

## 🔧 Objetivo

Reiniciar de forma segura un nodo sin perder disponibilidad de los servicios.

---

## 🗂️ Estado Inicial del Clúster

```bash
kubectl get nodes -o wide
kubectl get pods -A -o wide

NAME                   STATUS   ROLES    AGE     VERSION   INTERNAL-IP   EXTERNAL-IP      OS-IMAGE                         KERNEL-VERSION   CONTAINER-RUNTIME
pool-pl8qpi0aj-lqfwl   Ready    <none>   9m19s   v1.33.1   10.114.0.3    164.90.218.50    Debian GNU/Linux 12 (bookworm)   6.1.0-35-amd64   containerd://1.6.33
pool-pl8qpi0aj-lqfwt   Ready    <none>   8m47s   v1.33.1   10.114.0.4    207.154.206.96   Debian GNU/Linux 12 (bookworm)   6.1.0-35-amd64   containerd://1.6.33

```

---

## 📍 Paso 1: Cordon del nodo

Se evita que el nodo reciba nuevos pods.

```bash
kubectl cordon pool-pl8qpi0aj-lqfwt
```
PS C:\Users\0020360\Documents\DEVOPS\JORGE2025\kubernetes\administracion\reinicio_nodo> kubectl get nodes -o wide          
NAME                   STATUS                     ROLES    AGE   VERSION   INTERNAL-IP   EXTERNAL-IP      OS-IMAGE                         KERNEL-VERSION   CONTAINER-RUNTIME
pool-pl8qpi0aj-lqfwl   Ready                      <none>   11m   v1.33.1   10.114.0.3    164.90.218.50    Debian GNU/Linux 12 (bookworm)   6.1.0-35-amd64   containerd://1.6.33
pool-pl8qpi0aj-lqfwt   Ready,SchedulingDisabled   <none>   11m   v1.33.1   10.114.0.4    207.154.206.96   Debian GNU/Linux 12 (bookworm)   6.1.0-35-amd64   containerd://1.6.33


---

## 📍 Paso 2: Drain del nodo

Se eliminan los pods existentes (excepto los DaemonSets) y se vacía el nodo.

```bash
kubectl drain pool-pl8qpi0aj-lqfwt --ignore-daemonsets --delete-emptydir-data
```

---

## 📍 Paso 3: Reinicio del nodo (manual)

Reiniciamos el nodo desde la interfaz de DigitalOcean o con SSH.

---

## ✅ Verificación tras reinicio

Comprobamos que el nodo vuelve a estar activo, aunque deshabilitado para programación.

```bash
kubectl get nodes -o wide
kubectl get pods -A -o wide
```

---

## 📍 Paso 4: Habilitar el nodo (uncordon)

Permitimos que el nodo reciba pods de nuevo.

```bash
kubectl uncordon pool-pl8qpi0aj-lqfwt
```

---

## 📍 Paso 5: Reprogramar pods si es necesario

Como Kubernetes no mueve los pods al nodo original, eliminamos un pod y comprobamos su reprogramación.

```bash
kubectl delete pod -n web-apps apache-deployment-xxx
kubectl get pods -A -o wide
```

---

## ✅ Resultado final

Todos los pods vuelven a estar distribuidos, el nodo reiniciado funciona correctamente y está listo para recibir carga.

---

## 🧠 Nota

El reinicio fue exitoso y sin pérdida de disponibilidad. Los pods del namespace `web-apps` se reprogramaron automáticamente después del reinicio.