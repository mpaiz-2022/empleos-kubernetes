# **📌 Configuración de SSL con Traefik y Cert-Manager en Kubernetes**
### **Objetivo:** Configurar un certificado SSL con Let's Encrypt en un clúster Kubernetes usando **Traefik** como Ingress Controller.

---

## **1️⃣ Instalar Cert-Manager**
Ejecuta el siguiente comando para instalar **Cert-Manager** en Kubernetes:

```sh
kubectl apply -f https://github.com/cert-manager/cert-manager/releases/latest/download/cert-manager.yaml
```

Verifica que los pods están corriendo:
```sh
kubectl get pods -n cert-manager
```
Si todos están en `Running`, pasamos al siguiente paso.

---

## **2️⃣ Crear un ClusterIssuer para Let's Encrypt**
📌 **Crea el archivo `cluster-issuer.yaml`** con el siguiente contenido:

```yaml
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: letsencrypt-prod
spec:
  acme:
    email: tu-email@dominio.com  # Cambia esto por tu email
    server: https://acme-v02.api.letsencrypt.org/directory
    privateKeySecretRef:
      name: letsencrypt-prod
    solvers:
      - http01:
          ingress:
            class: traefik  # ✅ Asegura que use Traefik
```

📌 **Aplica el ClusterIssuer**:
```sh
kubectl apply -f cluster-issuer.yaml
```

Verifica que se creó correctamente:
```sh
kubectl get clusterissuer
```

---

## **3️⃣ Crear el Ingress para el Certificado SSL**
📌 **Crea el archivo `ingress.yaml`** con la siguiente configuración:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: omega-ingress
  annotations:
    cert-manager.io/cluster-issuer: "letsencrypt-prod"
    traefik.ingress.kubernetes.io/router.entrypoints: websecure
    traefik.ingress.kubernetes.io/router.tls: "true"
spec:
  ingressClassName: traefik
  tls:
    - hosts:
        - omegaempleos.com
      secretName: omegaempleos-tls  # Se generará automáticamente
  rules:
    - host: omegaempleos.com
      http:
        paths:
          - path: "/"
            pathType: Prefix
            backend:
              service:
                name: frontend
                port:
                  number: 3000
          - path: "/backend"
            pathType: Prefix
            backend:
              service:
                name: fastapi
                port:
                  number: 8000
```

📌 **Aplica el Ingress**:
```sh
kubectl apply -f ingress.yaml
```

Verifica que el Ingress se creó correctamente:
```sh
kubectl get ingress
```

---

## **4️⃣ Eliminar Certificados Anteriores y Regenerarlos**
Si ya habías intentado configurar SSL antes, borra cualquier configuración anterior antes de continuar:

```sh
kubectl delete certificate omegaempleos-tls
kubectl delete secret omegaempleos-tls
kubectl delete challenge --all
kubectl apply -f ingress.yaml
```

Verifica que el certificado se generó:
```sh
kubectl get certificate omegaempleos-tls
kubectl get challenges -A
```

---

## **5️⃣ Verificar que el Certificado Está Funcionando**
Ejecuta:
```sh
kubectl get certificate omegaempleos-tls
kubectl get challenges -A
```
Si todo está correcto, el estado del certificado debe ser `READY=True`.

También puedes probar en tu navegador:
```
https://omegaempleos.com
```
🚀 ¡Ahora tu dominio está completamente asegurado con **Let's Encrypt** y **Traefik** en Kubernetes! 🎉

---

## **🔄 ¿Qué Hacer Si Algo Sale Mal?**
Si algo no funciona como esperas:

### **🔹 1. Revisar los logs de Cert-Manager**
```sh
kubectl logs -n cert-manager -l app=cert-manager
```

### **🔹 2. Verificar si Cert-Manager creó un Ingress Temporal**
```sh
kubectl get ingress -A
```
Si ves un Ingress `cm-acme-http-solver-xxxx`, elimínalo:
```sh
kubectl delete ingress cm-acme-http-solver-xxxx -n default
```

### **🔹 3. Reiniciar Traefik**
Si aún no funciona:
```sh
kubectl rollout restart deployment -n kube-system traefik
```