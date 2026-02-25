# ROURA CEVASA CFO v9 · Configuración Dropbox — Guía Completa

## Resumen de lo que hace esta versión

En la v9, la **misma cuenta Dropbox** que almacena los informes CFO ahora también almacena y sincroniza:

| Archivo Dropbox | Contenido |
|---|---|
| `/ROURA_CFO/_SISTEMA_USUARIOS/usuarios.json` | Lista completa de usuarios, roles, contraseñas y ubicaciones |
| `/ROURA_CFO/_SISTEMA_USUARIOS/registro_accesos.json` | Log de cada login/logout y acción de admin |
| `/ROURA_CFO/{PEDIDO}_CFO_{FECHA}/` | Documentos del pedido (PDF incidencias, PDF cierre, etc.) |

> **Nota sobre seguridad:** El archivo `usuarios.json` se almacena en tu Dropbox privado. Solo quien tenga acceso a esa cuenta Dropbox puede leerlo. Las contraseñas se guardan en texto plano en ese archivo, por lo que se recomienda que la cuenta Dropbox tenga autenticación de dos factores activada.

---

## Paso 1 — Crear la Dropbox App (solo una vez)

1. Ve a **[https://www.dropbox.com/developers/apps](https://www.dropbox.com/developers/apps)**
2. Pulsa **Create app**
3. Configura:
   - **API:** Scoped access
   - **Type of access:** Full Dropbox
   - **Name:** `ROURA CFO App` (o cualquier nombre único)

4. En la pestaña **Permissions**, activa estos scopes:

   | Scope | Para qué |
   |---|---|
   | `files.content.write` | Guardar PDFs e informes |
   | `files.content.read` | Leer usuarios y datos |
   | `files.metadata.write` | Crear carpetas |
   | `account_info.read` | Mostrar nombre de usuario conectado |

   > ⚠️ Pulsa **Submit** después de marcar los permisos.

5. En la pestaña **Settings → OAuth 2**:
   - En **Redirect URIs**, añade la URL exacta donde está alojada la app.
   - Si usas GitHub Pages: `https://TU_USUARIO.github.io/TU_REPO/`
   - Si usas Netlify: `https://TU_SUBDOMINIO.netlify.app/`
   - Si pruebas en local: `http://localhost:8080/` o `http://127.0.0.1:8080/`
   - ⚠️ La URL debe coincidir exactamente (con o sin `/` final)

6. Copia el **App key** (en la pestaña Settings, primer campo)

---

## Paso 2 — Conectar en la app

1. Abre la app y accede como **admin/admin123**
2. Pulsa el icono ⚙️ (Configuración)
3. Pega el **App key** en el campo "Dropbox App Key"
4. Pulsa **CONECTAR CON DROPBOX**
5. Se abrirá la página de autorización de Dropbox → **Autorizar**
6. Volverás a la app con el estado: `✅ Dropbox conectado`

Al conectar, la app automáticamente:
- Crea la carpeta `/ROURA_CFO/_SISTEMA_USUARIOS/` en Dropbox
- Sube los usuarios existentes a `usuarios.json`
- Muestra `📁 Usuarios guardados en Dropbox por primera vez`

---

## Paso 3 — Estructura de carpetas en Dropbox

```
📦 Dropbox/
└── 📁 ROURA_CFO/
    ├── 📁 _SISTEMA_USUARIOS/          ← Gestión de usuarios del sistema
    │   ├── 📄 usuarios.json           ← Lista de usuarios y contraseñas
    │   └── 📄 registro_accesos.json   ← Log de accesos (hasta 500 entradas)
    │
    ├── 📁 PED-2024-001_CFO_20240315/  ← Carpeta PEDIDO (empieza por nº pedido)
    │   ├── 📄 00_INFO_PEDIDO.json
    │   ├── 📄 PED-2024-001 Informe Final CFO.pdf
    │   ├── 📄 PED-2024-001 Informe Cierre CFO.pdf
    │   └── 📄 PED-2024-001-INC-001-20240315.pdf
    │
    ├── 📁 PED-2024-002_CFO_20240320/  ← Otra visita
    │   └── ...
    │
    └── 📁 PED-2024-003_CFO_20240401/  ← El nº de pedido va siempre primero
        └── ...
```

> **Cambio importante en v9:** Los nombres de carpeta ahora **empiezan siempre por el número de pedido** para facilitar la búsqueda y ordenación en Dropbox. Antes era `ROURA_CFO_PED001_20240101`, ahora es `PED001_CFO_20240101`.

---

## Formato del archivo `usuarios.json`

```json
{
  "_tipo": "ROURA_CFO_USUARIOS_V1",
  "actualizadoEn": "2024-03-15T10:30:00.000Z",
  "version": "1.0",
  "usuarios": [
    {
      "username": "admin",
      "password": "admin123",
      "fullname": "Administrador",
      "role": "admin",
      "online": false,
      "lastSeen": 1710500000000,
      "created": 1710500000000,
      "location": {
        "city": "Madrid",
        "x": 380,
        "y": 230,
        "updated": 1710500000000
      }
    },
    {
      "username": "jlopez",
      "password": "pass123",
      "fullname": "Juan López",
      "role": "user",
      "online": true,
      "lastSeen": 1710500000000,
      "location": {
        "city": "Barcelona",
        "x": 545,
        "y": 140,
        "lat": 41.3851,
        "lon": 2.1734,
        "isReal": true,
        "updated": 1710500000000
      }
    }
  ]
}
```

---

## Formato del `registro_accesos.json`

```json
[
  {
    "ts": "2024-03-15T08:00:00.000Z",
    "usuario": "jlopez",
    "accion": "LOGIN",
    "ua": "Mozilla/5.0 (iPhone; CPU iPhone OS 17_0...)"
  },
  {
    "ts": "2024-03-15T17:30:00.000Z",
    "usuario": "admin",
    "accion": "EDIT_USER:jlopez",
    "ua": "Mozilla/5.0 (Macintosh; Intel Mac OS X...)"
  },
  {
    "ts": "2024-03-15T18:00:00.000Z",
    "usuario": "jlopez",
    "accion": "LOGOUT",
    "ua": "Mozilla/5.0 (iPhone; CPU iPhone OS 17_0...)"
  }
]
```

---

## Panel de Administración — Indicadores Dropbox

Al entrar en el **Panel de Administración**, la barra superior muestra el estado de la sincronización:

| Indicador | Significado |
|---|---|
| 🔴 Rojo | Dropbox no conectado. Los usuarios solo se guardan en este dispositivo. |
| 🟡 Amarillo | Dropbox conectado pero verificando o con error temporal. |
| 🟢 Verde | ✓ Sincronización activa. Muestra nº de usuarios y ruta del archivo. |

### Botón ↻ SYNC
Fuerza la sincronización inmediata de todos los usuarios a Dropbox. Útil cuando:
- Acción de crear/editar usuario no sincronizó (sin conexión momentánea)
- Quieres verificar que Dropbox tiene la versión más reciente

### Icono 📦 (azul Dropbox) en la barra superior
Abre directamente la carpeta `_SISTEMA_USUARIOS` en el navegador Dropbox web.

---

## Comportamiento de sincronización

| Situación | Comportamiento |
|---|---|
| **Dropbox conectado** | `loadUsers` descarga siempre desde Dropbox (fuente de verdad) |
| **Dropbox no conectado** | `loadUsers` usa caché local (`window.storage`) |
| **Crear/editar/borrar usuario** | Se guarda en local inmediatamente, luego se sincroniza con Dropbox en segundo plano |
| **Primer login tras conectar Dropbox** | Se comprueba si hay usuarios en Dropbox; si los hay, se importan; si no, se suben los locales |
| **Múltiples dispositivos** | Todos leen de Dropbox al hacer `loadUsers`; el último en guardar gana |
| **Sin internet** | Funciona con caché local; se sincroniza al recuperar conexión (próximo guardado) |

---

## Flujo multi-dispositivo

```
 Dispositivo A (Admin)          Dropbox                  Dispositivo B (Técnico)
        │                          │                              │
        │── Crea usuario ──────────▶ usuarios.json actualizado   │
        │                          │                              │
        │                          │◀── loadUsers() ─────────────│
        │                          │── devuelve usuario nuevo ───▶│
        │                          │                              │
        │                          │◀── LOGIN (registro) ─────────│
        │                          │── registro_accesos.json ────▶│ (log)
```

> Los datos de usuarios fluyen siempre a través de Dropbox. No se necesita ningún servidor propio.

---

## Resolución de problemas

### "Dropbox no conectado" tras recargar la página
La sesión Dropbox se guarda en `sessionStorage` (se pierde al cerrar pestaña/navegador). Solución: vuelve a conectar en Configuración. El token es de larga duración (refresh automático).

### Error 409 al subir usuarios
Significa que el archivo ya existe en Dropbox. La app usa `mode: overwrite` por lo que siempre sobreescribe. Si sigue fallando, abre Dropbox web y borra el archivo `usuarios.json` para que se regenere.

### "HTTP 400" al conectar
El **Redirect URI** en la app Dropbox no coincide con la URL de la página. Verifica que en la consola Dropbox está la URL exacta (incluyendo protocolo y `/` final si corresponde).

### Los usuarios creados en dispositivo A no aparecen en dispositivo B
1. Verifica que ambos dispositivos tienen Dropbox conectado (indicador verde en Config)
2. Desde el panel admin, pulsa ↻ SYNC en el dispositivo A
3. En el dispositivo B, sal del panel admin y vuelve a entrar (recarga desde Dropbox)

---

## Seguridad — Recomendaciones

1. **Activa 2FA en Dropbox** — La cuenta almacena contraseñas de usuario
2. **Cambia la contraseña del admin** desde el primer acceso
3. **No uses contraseñas de otros servicios** para los usuarios CFO
4. **Revisa el `registro_accesos.json`** periódicamente para detectar accesos no autorizados
5. Si compartes la Dropbox App Key, cualquier persona podría autorizar la app con su cuenta. Mantén el App Key privado.

---

## Despliegue recomendado

Para que el flujo OAuth funcione correctamente necesitas una URL pública (no `file://`):

### GitHub Pages (recomendado, gratuito)
```bash
# 1. Crea un repo en GitHub
# 2. Sube los archivos:
git init
git add index_v9_dropbox_usuarios.html manifest.json
git mv index_v9_dropbox_usuarios.html index.html
git commit -m "ROURA CFO v9"
git remote add origin https://github.com/TU_USUARIO/roura-cfo.git
git push -u origin main
# 3. Settings → Pages → Branch: main → Save
# 4. URL: https://TU_USUARIO.github.io/roura-cfo/
# 5. Añadir esa URL exacta como Redirect URI en Dropbox Developer Console
```

### Netlify Drop (sin registro, más rápido)
1. Ve a **[app.netlify.com/drop](https://app.netlify.com/drop)**
2. Arrastra la carpeta con `index.html` y `manifest.json`
3. Obtén la URL pública (ej: `https://random-name-123.netlify.app/`)
4. Añádela como Redirect URI en Dropbox

---

*ROURA CEVASA CFO App v9.0 · Dropbox multi-usuario · Gestión de instalaciones*
