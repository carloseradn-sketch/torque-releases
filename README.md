# Distribución de Torque (GitHub Releases + Pages)

Esta carpeta es el **contenido del repositorio público de distribución**. Reemplaza
al antiguo servidor PHP de InfinityFree: no hay panel de administración, todo se
publica automáticamente desde GitHub Actions al empujar un tag.

- **APK**: se adjunta como asset de un *Release* (descarga directa y anónima).
- **`version.json`**: lo escribe el workflow en este repo y se sirve por GitHub Pages.
- **`index.html`**: página pública de producto/descarga, que lee `version.json`.

```
distribution/
├─ index.html      # página de descarga (se sirve por Pages)
├─ .nojekyll       # evita que Jekyll ignore archivos
├─ version.json    # lo Sobrescribe el workflow en cada release
└─ README.md
```

## 1) Crear el repositorio público

1. Crea un repo **público** llamado `torque-releases` (o el nombre que quieras),
   por ejemplo `carloseradn-sketch/torque-releases`.
2. Sube el contenido de esta carpeta a la raíz de la rama `main`:
   `index.html`, `.nojekyll`, `README.md`.
3. Copia `assets/icon.png` de la app a la raíz como `icon.png`.
4. Activa **Pages**: Settings → Pages → Source: `Deploy from a branch`, Branch:
   `main` / `root`. La página quedará en
   `https://carloseradn-sketch.github.io/torque-releases/`.

> Si usas otro nombre de repo, actualiza `DIST_REPO` en
> `.github/workflows/release.yml`.

## 2) Secrets en el repo privado de la app

Settings → Secrets and variables → Actions → New repository secret:

| Secret | Valor |
|--------|-------|
| `RELEASES_TOKEN` | PAT (fine-grained) con `Contents: Read and write` sobre el repo público |
| `ANDROID_KEYSTORE_BASE64` | El keystore `.jks` en base64 (ver abajo) |
| `ANDROID_KEYSTORE_PASSWORD` | Contraseña del keystore |
| `ANDROID_KEY_ALIAS` | Alias de la key (en tu caso `torque`) |
| `ANDROID_KEY_PASSWORD` | Contraseña de la key |

`RELEASES_TOKEN`: crea un *fine-grained personal access token* en
GitHub → Settings → Developer settings → Personal access tokens, y dale acceso
al repo público con permiso **Contents: Read and write**.

Base64 del keystore:

```powershell
# PowerShell (Windows)
[Convert]::ToBase64String([IO.File]::ReadAllBytes("android\app\torque-keystore.jks")) | Set-Clipboard
```

```bash
# bash
base64 -w0 android/app/torque-keystore.jks
```

Copia el resultado como valor de `ANDROID_KEYSTORE_BASE64`.

## 3) Publicar una versión

1. Sube el número de versión en `pubspec.yaml` (ej. `version: 1.1.5+5`).
2. Haz commit y push a `main`.
3. Crea y empuja un tag:

```bash
git tag v1.1.5
git push origin v1.1.5
```

El workflow `.github/workflows/release.yml`:

- compila el APK release firmado,
- crea el Release `v1.1.5` en el repo público con el APK,
- genera `version.json` (URL de descarga + changelog desde los commits) y lo
  publica en Pages.

La app detecta la nueva versión al leer
`https://carloseradn-sketch.github.io/torque-releases/version.json`.

## Nota sobre el changelog

El workflow arma el changelog con los asuntos (`subject`) de los commits entre
el tag anterior y el nuevo, ignorando `chore`, `ci`, `docs`, `test` y merges.
Si prefieres controlarlo a mano, edita `version.json` en el repo público tras
el release.
