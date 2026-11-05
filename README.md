# Práctica 3: configurar el frontend de la calculadora

### Propósito

Desacoplar la configuración de Nginx de la imagen del frontend mediante un
`ConfigMap`. El proxy reenviará `/api/` y `/health` al backend de la calculadora
mediante el nombre DNS estable `calculadora-backend`.

## Archivos

- `nginx-configmap.yaml`: plantilla de configuración de Nginx.
- `nginx-deployment.yaml`: frontend de la calculadora montado con esa plantilla.
- `nginx-service.yaml`: acceso interno al frontend.

Construye la imagen y cárgala en Minikube antes de aplicar los manifiestos:

```bash
cd ../calculadora-frontend
```

Entra al proyecto que construye la imagen Nginx con el frontend.

```bash
docker build -t calculadora-frontend:latest .
```

Construye la imagen que montará la configuración desde el ConfigMap.

```bash
minikube image load calculadora-frontend:latest
```

Carga la imagen dentro de Minikube.

```bash
cd ../practice-3
```

Para usar un registro externo, publica
`tu-registro/calculadora-frontend:latest` y reemplaza esa imagen en
`nginx-deployment.yaml`.

## Crear y probar

```bash
kubectl apply -f namespace.yaml
```

Crea el namespace aislado `practice-3`.

```bash
kubectl apply -f nginx-configmap.yaml
```

Crea el ConfigMap con la plantilla de configuración de Nginx.

```bash
kubectl apply -f nginx-deployment.yaml
```

Crea el Deployment que monta la plantilla mediante el volumen configurado.

```bash
kubectl apply -f nginx-service.yaml
```

Expone el frontend mediante un Service `ClusterIP`.

```bash
kubectl rollout status deployment/calculadora-frontend -n practice-3
```

Espera a que el Pod nuevo quede disponible.

```bash
kubectl port-forward service/calculadora-frontend 8081:80 -n practice-3
```

Abre `http://localhost:8081`. Cambiar el ConfigMap no requiere reconstruir la imagen del frontend.

Para comprobar una actualización real, añade esta línea dentro de `location /` del archivo `nginx-configmap.yaml`:

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

La cabecera permite verificar desde el navegador o con `curl` cuál configuración está sirviendo Nginx.

Aplica la modificación guardada en el archivo:

```bash
kubectl apply -f nginx-configmap.yaml
```

Actualiza el objeto ConfigMap en Kubernetes.

```bash
kubectl get configmap calculadora-nginx -n practice-3
```

Comprueba que el ConfigMap existe y muestra sus metadatos.

Como el archivo se monta mediante `subPath`, los Pods existentes no reciben automáticamente el contenido actualizado:

```bash
kubectl rollout restart deployment/calculadora-frontend -n practice-3
```

Recrea los Pods para montar la nueva versión del ConfigMap.

En otra terminal, verifica la cabecera añadida:

```bash
curl -I http://localhost:8081
```

Debe aparecer `X-Config-Source: practice-3`.

## Limpieza

```bash
kubectl delete namespace practice-3
```
