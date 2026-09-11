# v1.6.2 — Fase 10: propagación rápida entre dispositivos

## Resumen

En la prueba manual, un cambio tardaba **50 segundos** en verse en el segundo
dispositivo. No era un fallo intermitente: era el **intervalo de sondeo**. La app
preguntaba a la hoja cada 60 segundos, así que un cambio podía tardar casi un
minuto en detectarse.

Ahora: **hasta 15 segundos**, y **al instante** en cuanto vuelves a la pestaña.

## Integridad

| Archivo | Tamaño | SHA-256 (primeros 16) |
|---|---|---|
| `index.html` | 440,2 KB | `A0C93ED995C80757` |
| `manifest.json` | 0,5 KB | `17489B0FECE45A31` |
| `logo.jpg` | 180,9 KB | `5DE65B0C3D118815` |

## Reproducido y medido antes de tocar nada

Escribí una prueba que abre los dos dispositivos a la vez y cronometra. Con
**v1.6.1**, el mismo escenario que describes:

| Escenario | v1.6.1 | v1.6.2 |
|---|---|---|
| B visible, eliminación en A | **más de 40 s sin reflejarlo** | **14 989 ms** |
| B en segundo plano → vuelves a la pestaña | 13 092 ms (por el sondeo, no por volver) | **310 ms** |
| Peticiones por comprobación | 12 | **2** |

## Los tres cambios

### 1. Comprueba la hoja al volver a la pestaña

Es justo el momento en que vas a mirar los datos. Antes había que esperar al
siguiente tic del temporizador.

```js
document.addEventListener('visibilitychange', alVolverALaPestana);
window.addEventListener('focus', alVolverALaPestana);
```

Con un freno de 3 segundos para no repetir si acabas de cambiar de ventana, y
**sin sondear nunca con la pestaña en segundo plano** (no gasta cuota ni batería).

> Este es el caso real: tienes la app en el móvil y en el ordenador. Registras
> algo en el móvil, te vas al ordenador y **al mirarlo ya está ahí**.

### 2. El sondeo cuesta 10 veces menos

Antes, cada comprobación leía **las 10 hojas en 10 peticiones**. Ahora usa
`values:batchGet` y las lee **en una sola**, más la de `ensureSheets`: **2
peticiones** en total.

Eso es lo que permite bajar el intervalo sin arriesgar la cuota de Google.

### 3. Intervalo por defecto: de 60 s a 15 s

Sigue siendo configurable en **Configuración → Google & Respaldo**: nunca, 15 s,
30 s, 1 min, 5 min o 15 min.

## Verificación

```powershell
node versions/v1.6.2/tests/run-tests.mjs      # 33/33
```

### Prueba de propagación (la que reproduce tu caso)

```powershell
node _debug/prueba-propagacion.mjs --version=v1.6.1   # la línea base
node _debug/prueba-propagacion.mjs --version=v1.6.2   # el arreglo
```

Deja un registro con tiempos en `_debug/log-propagacion.txt`.

### Prueba activa de dos dispositivos

```powershell
node _debug/prueba-activa-dos-dispositivos.mjs
```

Da de alta una alumna, pone otra en inactiva y elimina una tercera, comprobando
que llegan al otro dispositivo.

### Regresión

```powershell
$env:PUA_INDEX='D:\Flore\Documents\Deepseek\Pua-e\versions\v1.6.2\index.html'
node versions/v1.6.1/tests/run-tests.mjs      # 36/36
Remove-Item Env:\PUA_INDEX
```

## Nota sobre las herramientas de prueba

Dos mejoras en el banco de pruebas, porque la hoja simulada ahora persiste y eso
hacía que algunas verificaciones fueran reproducibles solo la primera vez:

- `entorno-simulacion/verificar.mjs` **vacía la hoja** antes de comprobar.
- Las dos hojas simuladas (entorno y banco de pruebas) entienden ya
  `values:batchGet`.

## Cómo desplegar

```powershell
node tools/publicar.mjs v1.6.2 --dest="C:\ruta\a\tu\clon" --confirm
cd C:\ruta\a\tu\clon
git add index.html manifest.json logo.jpg
git commit -m "v1.6.2: propagacion rapida entre dispositivos"
git push
```

## Cómo revertir

```powershell
node tools/publicar.mjs v1.6.1 --dest="C:\ruta\a\tu\clon" --confirm
```

Revertir es seguro: no hay cambios en el modelo de datos. Solo vuelve a tardar
hasta un minuto en detectar los cambios.
