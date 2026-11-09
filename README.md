# Práctica 3: configurar el frontend de la calculadora

### Propósito

Desacoplar la configuración de Nginx de la imagen del frontend mediante un
`ConfigMap`. El proxy reenviará `/api/` y `/health` al backend de la calculadora
mediante el nombre DNS estable `calculadora-backend`.

## Archivos

- `nginx-configmap.yaml`: plantilla de configuración de Nginx.
- `nginx-deployment.yaml`: frontend de la calculadora montado con esa plantilla.
- `nginx-service.yaml`: acceso interno al frontend.

Para usar un registro externo, publica
`tu-registro/calculadora-frontend:latest` y reemplaza esa imagen en
`nginx-deployment.yaml`.

## 1. Entrar al proyecto frontend

Entra al directorio que contiene el frontend.

```bash
cd ../calculadora-frontend
```

## 2. Construir la imagen

Construye la imagen que montará la configuración desde el ConfigMap.

```bash
docker build -t calculadora-frontend:latest .
```

## 3. Cargar la imagen en Minikube

Carga la imagen dentro de Minikube.

```bash
minikube image load calculadora-frontend:latest
```

## 4. Volver a los manifiestos

Regresa al directorio de la práctica.

```bash
cd ../practice-3
```

## 5. Crear el namespace

Crea el namespace aislado `practice-3`.

```bash
kubectl apply -f namespace.yaml
```

## 6. Crear el ConfigMap

Crea el ConfigMap con la plantilla de configuración de Nginx.

```bash
kubectl apply -f nginx-configmap.yaml
```

## 7. Crear el Deployment

Crea el Deployment que monta la plantilla mediante el volumen configurado.

```bash
kubectl apply -f nginx-deployment.yaml
```

## 8. Crear el Service

Expone el frontend mediante un Service `ClusterIP`.

```bash
kubectl apply -f nginx-service.yaml
```

## 9. Esperar el despliegue

Espera a que el Pod nuevo quede disponible.

```bash
kubectl rollout status deployment/calculadora-frontend -n practice-3
```

## 10. Abrir el acceso local

Mantén el proceso abierto y abre `http://localhost:8081`.

```bash
kubectl port-forward service/calculadora-frontend 8081:80 -n practice-3
```

## 11. Editar la configuración de Nginx

Añade esta línea dentro de `location /` del archivo `nginx-configmap.yaml`. La cabecera permite verificar qué configuración está sirviendo Nginx.

```nginx
add_header X-Config-Source practice-3 always;
```

La modificación debe quedar en una ubicación similar a:

```nginx
location / {
  try_files $uri $uri/ /index.html;
  add_header X-Config-Source practice-3 always;
}
```

## 12. Aplicar la modificación

Actualiza el objeto ConfigMap en Kubernetes.

```bash
kubectl apply -f nginx-configmap.yaml
```

## 13. Verificar el ConfigMap

Comprueba que el ConfigMap existe y muestra sus metadatos.

```bash
kubectl get configmap calculadora-nginx -n practice-3
```

Como el archivo se monta mediante `subPath`, los Pods existentes no reciben automáticamente el contenido actualizado:

## 14. Reiniciar el Deployment

Recrea los Pods para montar la nueva versión del ConfigMap.

```bash
kubectl rollout restart deployment/calculadora-frontend -n practice-3
```

## 15. Verificar la cabecera

En otra terminal, solicita la página y verifica la cabecera añadida.

```bash
curl -I http://localhost:8081
```

Debe aparecer `X-Config-Source: practice-3`.

## 16. Limpiar la práctica

Elimina el namespace y todos los recursos asociados.

```bash
kubectl delete namespace practice-3
```

## 16. Limpiar la práctica

Elimina el namespace y todos los recursos asociados.

```bash
kubectl delete namespace practice-3
```
