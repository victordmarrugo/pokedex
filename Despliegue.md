# Despliegue.md — Cómo desplegué la PokeDex en Azure

## Datos generales

| | |
|---|---|
| Plataforma | Azure Static Web Apps |
| Repositorio | https://github.com/victordmarrugo/pokedex |
| URL pública | https://polite-ground-0d1b20a0f.7.azurestaticapps.net |
| Fecha | 23 de abril de 2026 |

---

## Paso 1 — Subir el código a GitHub

Lo primero fue crear el repositorio en GitHub y subir los archivos compilados de la app. Abrí PowerShell en la carpeta `dist/pokedex-angular` y ejecuté:

```bash
git init
git add .
git commit -m "main -> main"
git branch -M main
git remote add origin https://github.com/victordmarrugo/pokedex.git
git push -u origin main
```

Subieron 710 objetos, unos 4.78 MiB en total. Sin problemas en este paso.

---

## Paso 2 — Crear el recurso en Azure

Entré a [https://portal.azure.com](https://portal.azure.com), busqué **Static Web Apps** y le di a **+ Create**. Llené el formulario así:

| Campo | Valor |
|-------|-------|
| Resource Group | rg-pokedex |
| Name | pokedex-app |
| Plan type | Free |
| Region | East US 2 |
| Source | GitHub |

Conecté mi cuenta de GitHub, seleccioné el repositorio `pokedex`, rama `main`, y en Build Details dejé todo en Custom con los campos vacíos. Le di a **Review + Create** y me salió la notificación de implementación correcta.

---

## Paso 3 — Problema con las imágenes (error 404)

### Qué pasó

La app cargaba pero las imágenes de los Pokémon no aparecían. Abrí las herramientas de desarrollador (F12 → Network) y vi esto:

```
GET /pokedex-angular/assets/images/pokemon-green.png → 404
```

Azure buscaba las imágenes en `/pokedex-angular/assets/images/` pero en el repositorio estaban en `/assets/images/`.

### Por qué pasaba

El archivo `src/environments/environment.prod.ts` tenía esta línea:

```typescript
imagesPath: '/pokedex-angular/assets/images',
```

Ese `/pokedex-angular/` de más era el problema.

### Cómo lo resolví

Cambié esa línea a:

```typescript
imagesPath: '/assets/images',
```

Luego limpié el caché de compilación y recompilé:

```bash
Remove-Item -Recurse -Force .angular
Remove-Item -Recurse -Force dist
ng build --base-href /
```

Subí los cambios y las imágenes empezaron a cargar al entrar al detalle de cada Pokémon.

---

## Paso 4 — Problema con OneDrive bloqueando la compilación

### Qué pasó

Al intentar compilar desde la carpeta del escritorio (que estaba sincronizada con OneDrive), salió este error:

```
EPERM, Permission denied: dist\pokedex-angular
```

OneDrive intentaba sincronizar los archivos al mismo tiempo que Angular intentaba escribirlos.

### Cómo lo resolví

Extraje el proyecto a una carpeta fuera de OneDrive (`C:\Users\User\Desktop\` sin sincronización activa) y pausé la sincronización de OneDrive por 2 horas. Desde ahí la compilación corrió sin problemas.

---

## Paso 5 — Problema con el redespliegue automático

### Qué pasó

Cada vez que hacía push a GitHub, Azure no se actualizaba. La pestaña Actions del repositorio siempre mostraba el mismo workflow del primer despliegue (16:06) y nunca corría uno nuevo.

### Por qué pasaba

La carpeta que subí a GitHub no tenía el archivo `.github/workflows/azure-static-web-apps.yml` que le indica a GitHub Actions cómo redesplegar en Azure automáticamente.

### Cómo lo resolví

Creé la carpeta y el archivo manualmente:

```bash
mkdir .github
mkdir .github\workflows
notepad .github\workflows\azure-static-web-apps.yml
```

Contenido del archivo:

```yaml
name: Azure Static Web Apps CI/CD

on:
  push:
    branches:
      - main

jobs:
  build_and_deploy_job:
    runs-on: ubuntu-latest
    name: Build and Deploy Job
    steps:
      - uses: actions/checkout@v3
      - name: Deploy
        uses: Azure/static-web-apps-deploy@v1
        with:
          azure_static_web_apps_api_token: ${{ secrets.AZURE_STATIC_WEB_APPS_API_TOKEN_POLITE_GROUND_0D1B20A0F }}
          repo_token: ${{ secrets.GITHUB_TOKEN }}
          action: "upload"
          app_location: "/"
          skip_app_build: true
```

El token de Azure ya existía como secreto en el repositorio desde el primer despliegue, solo había que referenciarlo correctamente. Después de subir este archivo, cada push ya activa el redespliegue automáticamente.

---

## Paso 6 — Problema con el error 500 (pantalla en blanco)

### Qué pasó

La app mostraba un error 500 con el mensaje "¿POKÉAPI está evolucionando?" en lugar de cargar los Pokémon.

### Por qué pasaba

La `Content-Security-Policy` que tenía configurada era muy restrictiva (`default-src 'self'`), lo que bloqueaba las peticiones hacia la PokeAPI que es un servidor externo.

### Cómo lo resolví

Actualicé el archivo `staticwebapp.config.json` con una política más permisiva que deja que la app se conecte a APIs externas:

```json
{
  "globalHeaders": {
    "Content-Security-Policy": "default-src * 'unsafe-inline' 'unsafe-eval'; img-src * data:; font-src * data:;",
    "Strict-Transport-Security": "max-age=31536000; includeSubDomains; preload",
    "X-Content-Type-Options": "nosniff",
    "X-Frame-Options": "DENY",
    "Referrer-Policy": "no-referrer",
    "Permissions-Policy": "camera=(), microphone=(), geolocation=()"
  },
  "navigationFallback": {
    "rewrite": "/index.html",
    "exclude": ["/assets/*", "/*.{css,js,png,jpg,ico,json}"]
  }
}
```

Después de subir ese cambio la app cargó completamente con todas las imágenes y datos.

---

## Resultado final del escaneo de seguridad

Se escaneó la app en [https://securityheaders.com](https://securityheaders.com):

**Calificación: A** ✅

| Encabezado | Resultado |
|-----------|-----------|
| Content-Security-Policy | ✅ |
| Strict-Transport-Security | ✅ |
| X-Content-Type-Options | ✅ |
| X-Frame-Options | ✅ |
| Referrer-Policy | ✅ |
| Permissions-Policy | ✅ |

---

## Desafío Maestro — Dominio Personalizado con DuckDNS

Como parte del desafío opcional, configuré un dominio personalizado gratuito usando DuckDNS.

1. Entré a [https://www.duckdns.org](https://www.duckdns.org) e inicié sesión con GitHub
2. Creé el subdominio `victorporkedex`
3. Apunté la IP del servidor de Azure (`132.220.38.112`) al dominio

**Dominio configurado:** http://victorporkedex.duckdns.org

El plan gratuito de Azure Static Web Apps no permite asociar dominios personalizados externos desde el portal, por lo que aunque el dominio apunta correctamente a la IP del servidor, Azure no redirige las peticiones al no reconocer el host. Para que funcione completamente se requeriría el plan de pago de Azure o un servidor propio. De todas formas el proceso de configuración del DNS quedó completado y documentado.

---

*Víctor D. Marrugo — Abril 2026*
