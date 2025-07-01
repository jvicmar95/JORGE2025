# 🧪 Lab 3: Control de Acceso con RBAC en Kubernetes

## 🎯 Objetivo:
Crear un usuario (mediante ServiceAccount) que **solo pueda listar pods** en un namespace concreto (`rbac-lab`) y no pueda borrar, crear ni modificar nada más.

---

## 🔐 Conceptos clave

| Término            | ¿Qué es?                                                                 |
|--------------------|--------------------------------------------------------------------------|
| `Role`             | Define lo que se puede hacer dentro de un namespace.                     |
| `RoleBinding`      | Conecta un `Role` con una identidad (usuario, grupo o ServiceAccount).   |
| `ClusterRole`      | Igual que `Role`, pero para todo el clúster.                             |
| `ClusterRoleBinding` | Igual que `RoleBinding` pero a nivel global.                           |

---

## ✅ Paso 1: Crear el Namespace
```bash
kubectl create namespace rbac-lab
```

---

## ✅ Paso 2: Crear un Role que solo permita listar pods
### `role-list-pods.yaml`
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: pod-reader
  namespace: rbac-lab
rules:
  - apiGroups: [""]
    resources: ["pods"]
    verbs: ["get", "list", "watch"]
```
```bash
kubectl apply -f role-list-pods.yaml
```

📌 *Este role otorga permisos mínimos: solo lectura sobre pods.*

---

## ✅ Paso 3: Crear una ServiceAccount y vincularla al Role
### `sa-and-rolebinding.yaml`
```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: lector-pods
  namespace: rbac-lab
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: leer-pods-binding
  namespace: rbac-lab
subjects:
  - kind: ServiceAccount
    name: lector-pods
    namespace: rbac-lab
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```
```bash
kubectl apply -f sa-and-rolebinding.yaml
```

📌 *Esto crea un "usuario" con permisos limitados y lo asocia al Role.*

---

## ✅ Paso 4: Probar el acceso con esa cuenta
### `reader-pod.yaml`
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: sa-tester
  namespace: rbac-lab
spec:
  serviceAccountName: lector-pods
  containers:
    - name: tester
      image: bitnami/kubectl:latest
      command: ["sleep", "3600"]
  restartPolicy: Never
```
```bash
kubectl apply -f reader-pod.yaml
kubectl exec -n rbac-lab -it sa-tester -- /bin/sh
```

### Dentro del pod ejecuta:
```bash
kubectl get pods -n rbac-lab   # ✅ Funciona
kubectl delete pod sa-tester -n rbac-lab   # ❌ Prohibido
```

📌 *El intento de borrar el pod da error:*
```
Error from server (Forbidden): pods "sa-tester" is forbidden: User "system:serviceaccount:rbac-lab:lector-pods" cannot delete resource "pods"...
```

---

## 🏁 Lab 3 Completado

| Acción                    | Resultado                                          |
|---------------------------|---------------------------------------------------|
| Crear Role con permisos   | Solo lectura (`get`, `list`, `watch`)             |
| Crear ServiceAccount      | Usuario limitado                                  |
| RoleBinding               | Asociación entre identidad y permisos             |
| Prueba real               | ✅ Puede listar pods<br>❌ No puede borrarlos      |