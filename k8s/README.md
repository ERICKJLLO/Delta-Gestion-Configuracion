# Despliegue en Kubernetes (Minikube)

Manifiestos para desplegar **Delta** (`db`, `backend`, `frontend`) en un clúster local de Minikube dentro del namespace `delta`.

## 1. Iniciar Minikube

```bash
minikube start --driver=docker
minikube status
kubectl config current-context
```

## 2. Configurar secretos locales (no versionados en Git)

Copia la plantilla de secretos y define tus valores para `POSTGRES_PASSWORD`, `DATABASE_URL` y `JWT_SECRET` (puedes generar el secreto JWT con `openssl rand -hex 32`):

```bash
cp k8s/01-secret.yaml.example k8s/01-secret.yaml
```

> `k8s/01-secret.yaml` está en `.gitignore` para evitar exponer credenciales en el repositorio.

## 3. Imágenes en Mac con Apple Silicon (ARM64)

Si las imágenes de DockerHub solo están publicadas para `linux/amd64` y estás en un Mac con chip M1/M2/M3/M4, constrúyelas directamente dentro del nodo de Minikube:

```bash
minikube image build -t erickjllo/delta-backend:latest ./Delta-Backend
minikube image build -t erickjllo/delta-frontend:latest ./Delta-Frontend
```

## 4. Aplicar manifiestos y verificar estado

```bash
kubectl apply -f k8s/
kubectl get pods,svc,pvc -n delta
```

## 5. Abrir la aplicación

```bash
minikube service frontend -n delta
# Alternativa con puerto fijo en http://localhost:8080:
# kubectl port-forward svc/frontend -n delta 8080:80
```

## 6. Comandos de verificación y diagnóstico (E2E / Observabilidad)

```bash
# Consultar usuarios registrados en PostgreSQL (incluyendo hash bcrypt)
kubectl exec -it deploy/db -n delta -- psql -U delta_user -d delta_db -c "SELECT id, email, company_name, plan_id, password_hash FROM users;"

# Ver logs de la API
kubectl logs deploy/backend -n delta

# Inspeccionar eventos de un pod ante fallos
kubectl describe pod -l app=backend -n delta

# Probar persistencia (eliminar el pod de BD y comprobar que los datos se mantienen al recrearse)
kubectl delete pod -l app=db -n delta
kubectl get pods -n delta -w
```
