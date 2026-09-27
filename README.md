# otarepository

Manifiesto de actualizacion de **OpenOrbis Store** (TITLE_ID `BREW00101`).

La app lee `ota.json` al arrancar y vuelve a consultarlo con R1 en la pestana
**Actualizar**. Es un archivo estatico: no hay backend, ni sesion, ni endpoint.
A proposito, para que la app pueda repararse a si misma aunque el servidor PHP
este caido.

## Por que el PKG vive aqui y no en Releases

`sceHttp` **no sigue redirecciones**. GitHub responde un asset de Release con un
`302` hacia una URL firmada en otro host, y el cuerpo de esa respuesta esta
vacio, asi que escribirlo a disco produciria un paquete corrupto. El cliente
detecta el 3xx y lo dice en pantalla en vez de installar basura.

Por eso el `.pkg` se sube como archivo normal al repositorio y se lee por
`raw.githubusercontent.com`, que responde `200` con los bytes directamente.

## Publicar una version

1. Sube el paquete nuevo a la raiz del repo con su nombre exacto:
   `IV0000-BREW00101_00-OPENORBISAPP0000.pkg`
2. Actualiza `ota.json`:
   - `build`: **subir el entero**. Es lo unico que compara la app. Si subes el
     pkg sin subir `build`, la app no reconocera su propia actualizacion.
   - `size`: bytes exactos del `.pkg`. La app lo verifica y borra la descarga si
     no coincide, asi que un valor erroneo bloquea la instalacion.
   - `notes`: texto corto, se ajusta solo.
3. Copia ese mismo `ota.json` a `assets/ota.json` del proyecto y sube `BUILD` en
   el `Makefile`.

Git LFS es opcional. El limite de `raw.githubusercontent.com` esta muy por encima
del tamano de un pkg de homebrew, asi que un LFS dariaria problemas sin dar
ventaja.

## Formato

```json
{
  "schema": 1,
  "packages": [
    {
      "titleId": "BREW00101",
      "contentId": "IV0000-BREW00101_00-OPENORBISAPP0000",
      "version": "1.01",
      "build": 2,
      "url": "https://raw.githubusercontent.com/.../IV0000-BREW00101_00-OPENORBISAPP0000.pkg",
      "size": 6619136,
      "notes": "..."
    }
  ]
}
```

`schema` debe ser `1`. Una entrada sin `build` valido o sin URL `http(s)` se
descarta en silencio, y un manifiesto donde no queda ninguna entrada valida se
rechaza entero sin tocar lo que la app ya tenia.
