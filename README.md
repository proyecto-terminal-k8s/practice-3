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
docker build -t calculadora-frontend:latest .
minikube image load calculadora-frontend:latest
cd ../practice-3
```

Para usar un registro externo, publica
`tu-registro/calculadora-frontend:latest` y reemplaza esa imagen en
`nginx-deployment.yaml`.

## Crear y probar

```bash
kubectl apply -f namespace.yaml
kubectl apply -f nginx-configmap.yaml
kubectl apply -f nginx-deployment.yaml
kubectl apply -f nginx-service.yaml
kubectl rollout status deployment/calculadora-frontend -n practice-3
kubectl port-forward service/calculadora-frontend 8081:80 -n practice-3
```

Abre `http://localhost:8081`. Cambiar el ConfigMap no requiere reconstruir la imagen del frontend. Como este archivo se monta mediante `subPath`, reinicia el Deployment para que cargue el contenido actualizado.

```bash
kubectl apply -f nginx-configmap.yaml
kubectl get configmap calculadora-nginx -n practice-3
kubectl rollout restart deployment/calculadora-frontend -n practice-3
```

## Limpieza

```bash
kubectl delete namespace practice-3
```
