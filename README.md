# POC - APIM API Demo (OpenAPI and CI/CD)

🔧 Demo: despliegue automático de una API OpenAPI (YAML) a Azure API Management (APIM) usando GitHub Actions.

## Contenido del repositorio
- `openapi.yaml` - OpenAPI 3.0.1 con la operación GET `/farmacias/disponibilidad` (usa query params `medicamento`, `ciudad`).
- `.github/workflows/deploy-apim.yml` - GitHub Actions que importa/actualiza la API en APIM con `az apim api import`.


## Objetivo
- Mostrar integración CI/CD: al hacer push a `main`, el workflow importa la definición OpenAPI en APIM y conecta el API con el backend: `http://4.157.52.221/api`.

## Pre-requisitos
- Ya existe un APIM creado en Azure (ocrea uno si es necesario).
- Crear un Service Principal de Azure con permisos sobre el recurso APIM (o su Resource Group).

## Cómo crear el Service Principal (ejemplo mínimo):
```powershell
az login
az ad sp create-for-rbac --name "github-deploy-apim" --role Contributor --scopes /subscriptions/<SUBSCRIPTION_ID>/resourceGroups/<RG_NAME>/providers/Microsoft.ApiManagement/service/<APIM_NAME>
```
Guarda el JSON resultante como secreto `AZURE_CREDENTIALS` en GitHub (Settings → Secrets and variables → Actions → New repository secret).

## GitHub Secrets que necesitarás (repo settings → Secrets):
- `AZURE_CREDENTIALS` : El JSON devuelto por `az ad sp create-for-rbac`.
- `AZURE_RESOURCE_GROUP` : El nombre del resource group en Azure.
- `APIM_SERVICE_NAME` : Nombre del servicio de APIM, por ejemplo `apim-provefarma-demo`.
- `APIM_API_ID` : Un id único para la API dentro de APIM (ej: `demo-api-farmacia`).


## Cómo funciona el workflow
1. El workflow se dispara al hacer push en `main`.
2. Hace login con `azure/login` (usando `AZURE_CREDENTIALS`).
3. Importa la especificación OpenAPI (`openapi.yaml`) usando `az apim api import` y configura `service-url` apuntando a `http://4.157.52.221/api`.


## Notas y recomendaciones
- Si APIM requiere una suscripción (subscription key), deberás crear un producto y/o modificar políticas para exposición pública o incluir un `policy` que permita acceso sin key (solo para demo).
- Si quieres que la API tenga path raíz `/api/farmacias`, puedes ajustar `--path` en el workflow a `api` y `openapi.yaml` a `...` según convenga.
- Para entornos de producción, limita el Service Principal a un rol mínimo necesario (por ejemplo `Api Management Service Contributor`), y restringe la scope al APIM o resource group.

## Prueba rápida (once deployed)
Ejemplo de `curl` (reemplaza host y clave si aplica):
```bash
curl -s "https://apim-provefarma-demo.azure-api.net/api/farmacias/disponibilidad?medicamento=paracetamol&ciudad=quito" -H "Ocp-Apim-Subscription-Key: <YOUR_KEY>"
```

Si la APIM está configurada sin suscripción, puede funcionar sin `Ocp-Apim-Subscription-Key`.


---

### Extensiones y mejoras que puedes añadir
- Añadir test step: ejecutar un `curl` desde el workflow para validar retorno 200.
- Crear `policy` de APIM (XML) para logging, rate-limit y transformaciones.
- Configurar `openapi.yaml` para incluir `x-` extensions si deseas policies embebidas.

¡Listo! El workflow implementa una integración CI/CD simple y demostrable para desplegar la OpenAPI en APIM.
