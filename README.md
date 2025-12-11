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
 
## Datos del APIM de demo (proporcionados)
- Grupo de recursos: `rg-demo-provefarma-apim`
- Nombre del servicio APIM: `apim-provefarma-demo`
- URL del portal para desarrolladores: `https://apim-provefarma-demo.developer.azure-api.net`
- URL de puerta de enlace: `https://apim-provefarma-demo.azure-api.net`
- Subscripción: `a407594f-18e1-4ed7-b941-507c946bb6b3`
- Ubicación: East US

## Cómo crear el Service Principal (ejemplo recomendado para demo)
Para cifrar las credenciales y limitar el scope, crea un Service Principal con rol `Api Management Service Contributor` y limita el scope al recurso APIM:
```powershell
az login
az ad sp create-for-rbac --name "github-deploy-apim" --role "Api Management Service Contributor" --scopes /subscriptions/a407594f-18e1-4ed7-b941-507c946bb6b3/resourceGroups/rg-demo-provefarma-apim/providers/Microsoft.ApiManagement/service/apim-provefarma-demo
```
Guarda el JSON resultante (incluye appId, password, tenant) como secreto `AZURE_CREDENTIALS` en GitHub (Settings → Secrets and variables → Actions → New repository secret).
Guarda el JSON resultante como secreto `AZURE_CREDENTIALS` en GitHub (Settings → Secrets and variables → Actions → New repository secret).

## GitHub Secrets que necesitarás (repo settings → Secrets):
- `AZURE_CREDENTIALS` : El JSON devuelto por `az ad sp create-for-rbac`.
- `AZURE_RESOURCE_GROUP` : `rg-demo-provefarma-apim`.
- `APIM_SERVICE_NAME` : `apim-provefarma-demo`.
- `APIM_API_ID` : Un id único para la API dentro de APIM (ej: `demo-api-farmacia`).
- `AZURE_SUBSCRIPTION_ID` : `a407594f-18e1-4ed7-b941-507c946bb6b3`.
- `APIM_GATEWAY_URL` : `https://apim-provefarma-demo.azure-api.net`.
- `APIM_SUBSCRIPTION_KEY` : (opcional) Si tu APIM requiere suscripción, agrega la clave para el producto que expone la API.

⚠️ Nota: si ya existe una API con `APIM_API_ID` (por ejemplo `api-mock-farmacia` en tu APIM), el workflow importará y **reemplazará** la definición y la `policy` si `policy.xml` está presente. Para preservar APIs existentes en el entorno de demo, usa un `APIM_API_ID` nuevo o revisa la política en `policy.xml` antes de ejecutar el workflow.


## Cómo funciona el workflow
1. El workflow se dispara al hacer push en `main`.
2. Hace login con `azure/login` (usando `AZURE_CREDENTIALS`).
3. Importa la especificación OpenAPI (`openapi.yaml`) usando `az apim api import` y configura `service-url` apuntando a `http://4.157.52.221/api`.
4. (Opcional) Si existe `policy.xml` en el repo, el workflow aplicará la política al API en APIM (ej. establecer backend y remover encabezado de suscripción para demo).
5. El workflow ejecutará una prueba `curl` contra la puerta de enlace APIM y reportará error si la respuesta no es HTTP 200.


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
