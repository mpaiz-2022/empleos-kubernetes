# **🚀 Guía de Configuración del VPS con K3s y GitHub Actions**
Este documento explica cómo configurar un VPS desde cero para desplegar una aplicación con Kubernetes (**K3s**) y GitHub Actions.

---

## **🔹 Paso 1: Crear el VPS en Digital Ocean**
1. **Escoger una imagen de Ubuntu 22.04 LTS**  
2. **Configurar el servidor:**
   - **Plan MINIMO recomendado:** `2 GB RAM, 2 vCPU`
   - **Ubicación:** La más cercana a tus usuarios
   - **Autenticación:** `SSH Key` o `Contraseña`
   - **Nombre del servidor:** `empleos-infra`
   - **Asignar una IP pública**

3. **Acceder por SSH**:
   ```sh
   ssh root@<IP_DEL_VPS>
   ```

---

## **🔹 Paso 2: Actualizar el Servidor**
```sh
sudo apt update && sudo apt upgrade -y
```
- **Instalar herramientas necesarias:**
  ```sh
  sudo apt install -y curl vim unzip
  ```

---

## **🔹 Paso 3: Instalar Kubernetes (K3s)**
K3s es una versión ligera de Kubernetes optimizada para VPS.

1. **Descargar e instalar K3s:**
   ```sh
   curl -sfL https://get.k3s.io | sh -
   ```

2. **Verificar que K3s está corriendo:**
   ```sh
   sudo systemctl status k3s
   ```

3. **Habilitar `kubectl` para el usuario actual:**
   ```sh
   mkdir -p ~/.kube
   sudo cp /etc/rancher/k3s/k3s.yaml ~/.kube/config
   sudo chown $(id -u):$(id -g) ~/.kube/config
   export KUBECONFIG=~/.kube/config
   ```

4. **Probar la conexión a Kubernetes:**
   ```sh
   kubectl get nodes
   ```
   **Salida esperada:**
   ```
   NAME            STATUS   ROLES                  AGE     VERSION
   empleos-infra   Ready    control-plane,master   5m      v1.31.5+k3s1
   ```

---

## **🔹 Paso 4: Extraer la Configuración de Kubernetes**
Para permitir que GitHub Actions interactúe con el cluster, necesitamos el archivo de configuración (`KUBECONFIG`).

1. **Convertir el archivo a Base64:**
   ```sh
   cat ~/.kube/config | base64 -w 0
   ```
   **Copiar la salida generada**.

2. **Guardar este valor en los `Secrets` de GitHub**:
   - Ir al repositorio `empleos-kubernetes` en GitHub.
   - **Settings** → **Secrets and variables** → **Actions**.
   - **Nuevo Secret** → Nombre: `KUBECONFIG_BASE64` → Pegar el valor **Base64** generado.

---

## **🔹 Paso 5: Instalar Traefik (Ingress Controller)**
K3s ya incluye **Traefik** como controlador de ingress.

1. **Verificar si está instalado:**
   ```sh
   kubectl get pods -A | grep traefik
   ```
   **Salida esperada:**
   ```
   kube-system   traefik-5d45fc8cc9-xxxxx   1/1   Running   0   2m
   ```

2. **Si no está corriendo, reinstalarlo:**
   ```sh
   kubectl delete -A helmchart traefik
   kubectl apply -f https://raw.githubusercontent.com/traefik/traefik/v2.10/examples/kubernetes/crds.yaml
   ```

---

## **🔹 Paso 6: Verificar que Todo Funciona**
1. **Verificar los pods:**
   ```sh
   kubectl get pods -A
   ```
2. **Probar acceso al cluster:**
   ```sh
   kubectl get services
   ```

🚀 **¡Listo! Ahora GitHub Actions puede gestionar los despliegues en K3s!** ✅