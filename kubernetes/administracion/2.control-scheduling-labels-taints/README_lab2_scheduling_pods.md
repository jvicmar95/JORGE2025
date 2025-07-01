# 🧪 Lab 2 - Control de Scheduling de Pods en Kubernetes

Este laboratorio demuestra cómo controlar el **scheduling** (asignación) de pods en un clúster de Kubernetes usando:
- Labels en nodos
- `nodeSelector`
- Taints y Tolerations

---

## 🔧 Preparación inicial

### 1. Ver nodos del clúster
```bash
kubectl get nodes -o wide
```

### 2. Etiquetar los nodos
```bash
kubectl label node pool-pl8qpi0aj-lqfwl rol=worker1
kubectl label node pool-pl8qpi0aj-lqfwt rol=worker2
```

Ver etiquetas:
```bash
kubectl get nodes --show-labels
```

---

## 🚀 Programar Pods con `nodeSelector`

Creamos un pod que solo se ejecuta en `worker1`:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-worker1
spec:
  containers:
    - name: nginx
      image: nginx:stable
  nodeSelector:
    rol: worker1
```

```bash
kubectl apply -f pod_worker1.yaml
kubectl get pod nginx-worker1 -o wide
```

---

## ⛔ Bloquear un nodo con Taints

Se marca el nodo `worker2` con un taint para impedir que reciba pods normales:

```bash
kubectl taint node pool-pl8qpi0aj-lqfwt acceso=bloqueado:NoSchedule
kubectl describe node pool-pl8qpi0aj-lqfwt | grep Taint
```

---

## ✅ Permitir un Pod con `tolerations`

Creamos un pod que **tolera el taint** y además se fuerza a ejecutar en `worker2`:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: toleration-pod
spec:
  containers:
    - name: busybox
      image: busybox
      command: ["sleep", "3600"]
  tolerations:
    - key: "acceso"
      operator: "Equal"
      value: "bloqueado"
      effect: "NoSchedule"
  nodeSelector:
    rol: worker2
```

```bash
kubectl apply -f toleration-pod.yaml
kubectl get pod toleration-pod -o wide
```

---

## 📋 Resumen del Laboratorio

| Paso | Acción | Comando / YAML | ¿Por qué se hace? |
|------|--------|----------------|--------------------|
| 1 | Consultar los nodos | `kubectl get nodes` | Ver qué nodos hay en el clúster |
| 2 | Etiquetar nodos | `kubectl label node ...` | Clasificar nodos para usarlos luego |
| 3 | Pod en worker1 | `nodeSelector` en YAML | Limitar pod a un nodo específico |
| 4 | Taint a worker2 | `kubectl taint node ...` | Impedir que reciba pods sin tolerancia |
| 5 | Pod con toleration | `tolerations` en YAML | Permitir que el pod se ejecute en nodo vetado |
| 6 | Forzar a worker2 | `nodeSelector` en YAML | Combinación de forzar nodo + permitir taint |

---

## 🧹 Limpiar el laboratorio

```bash
kubectl delete pod nginx-worker1 toleration-pod
kubectl taint node pool-pl8qpi0aj-lqfwt acceso=bloqueado:NoSchedule-
kubectl label node pool-pl8qpi0aj-lqfwl rol-
kubectl label node pool-pl8qpi0aj-lqfwt rol-
```