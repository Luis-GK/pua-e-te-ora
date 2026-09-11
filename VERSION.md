# v1.6.0 — Fase 8: calendario con vistas (mes / semana / día)

## Resumen

El calendario del Dashboard gana un **selector de vista** dentro del propio
calendario, y deja de ser una plantilla de días de la semana para convertirse en
un calendario **con las clases reales de cada fecha**.

Cuando un día tiene más de una clase, su aro se divide en tantos segmentos como
clases, uno por color. Sigue viéndose como hasta ahora, pero ya dice todo lo que
hay ese día.

## Integridad

| Archivo | Tamaño | SHA-256 (primeros 16) |
|---|---|---|
| `index.html` | 434,1 KB | `8B8E6D443528A2F5` |
| `manifest.json` | 0,5 KB | `17489B0FECE45A31` |
| `logo.jpg` | 180,9 KB | `5DE65B0C3D118815` |

## 1. Selector de vista, en el propio calendario

Tres botones **Mes · Semana · Día** justo encima de la rejilla. No hay que ir a
Configuración. Las flechas ‹ › navegan según la vista activa:

| Vista | Avanza |
|---|---|
| Mes | un mes |
| Semana | siete días |
| Día | un día |

«Hoy» aparece cuando no estás en la fecha actual y te devuelve a ella.

## 2. El problema de los varios colores por día

La solución es un **aro segmentado**: el círculo del día conserva su tamaño y su
número, pero el borde se reparte en tantos arcos como clases, un color cada uno.

- Con **una** clase se ve exactamente igual que antes (aro liso).
- Con **dos o tres** se ve un aro partido, sin ocupar un solo píxel más.
- El número mantiene el color de la primera clase, así que el día se sigue
  leyendo de un vistazo.

Además, **el detalle ya no vive en el mes**: al pulsar un día se abre su vista de
Día, con el nombre y la hora de cada clase. Así el mes solo tiene que decir
«cuántas y de qué colores», no repetirlo todo.

> **Alternativas que consideré** y por qué no las elegí:
> - *Puntos debajo del número*: muy legible, pero añade altura a todas las celdas
>   y en el móvil la rejilla queda apretada.
> - *Color dominante + contador*: pierde información (no sabes de qué clases).
> - *Barra fina inferior*: como el aro, pero menos visible.

Si prefieres cualquiera de ellas, el cambio está acotado a una función
(`anilloDia` + `celdaDia`).

## 3. Ahora el calendario dice la verdad

Antes coloreaba cualquier día cuyo **día de la semana** tuviera clase: era una
plantilla semanal, no un calendario. No sabía nada de cancelaciones ni de
reagendados.

Ahora `clasesDeFecha()` resuelve, para cada fecha concreta:

- las modalidades que tocan ese día de la semana,
- menos las que tengan una **cancelación** registrada para ese día y hora,
- más las **reagendadas** cuya fecha nueva sea esa.

Las canceladas aparecen tachadas y en gris, tanto en la semana como en el día.
El `title` de cada día del mes lista sus clases (por ejemplo
*«2 clases: 17:00 Hawaiano, 19:00 ORI ONLINE»*).

## 4. Un bug de fondo que encontré al hacerlo

Al reagendar, `nuevaDia` se guardaba como **1–7** (con 7 = domingo), mientras que
`modalidad.dias` y `evento.dias` usan **0–6** (con 0 = domingo). Consecuencia:
**una clase reagendada a domingo nunca aparecía** en la rejilla del Horario.

Ahora se guarda en el formato correcto y la lectura tolera los valores antiguos,
así que los reagendados que ya tuvieras a domingo vuelven a verse.

## Verificación

```powershell
cd D:\Flore\Documents\Deepseek\Pua-e
node versions/v1.6.0/tests/run-tests.mjs      # 38/38
```

### Prueba de regresión

```powershell
$env:PUA_INDEX='D:\Flore\Documents\Deepseek\Pua-e\versions\v1.6.0\index.html'
node versions/v1.5.2/tests/run-tests.mjs      # 39/39
Remove-Item Env:\PUA_INDEX
```

### Comprobaciones nuevas

| Grupo | Qué verifica |
|---|---|
| Vistas | tres botones, abre en mes, se cambia desde el calendario |
| Navegación | avanza 1 día / 7 días / 1 mes según la vista, y «Hoy» vuelve |
| Varios colores | hay días con 2 clases y el aro las reparte en segmentos |
| Detalle | la semana lista con hora, marca los días libres, y el día muestra su fecha |
| Interacción | los 35 días del mes son pulsables y abren su detalle |

## Cómo desplegar

```powershell
node tools/publicar.mjs v1.6.0 --dest="C:\ruta\a\tu\clon" --confirm
cd C:\ruta\a\tu\clon
git add index.html manifest.json logo.jpg
git commit -m "v1.6.0: calendario con vistas mes/semana/dia y aro multicolor"
git push
```

## Cómo revertir

```powershell
node tools/publicar.mjs v1.5.2 --dest="C:\ruta\a\tu\clon" --confirm
```

Revertir es seguro: no hay cambios en el modelo de datos ni en la
sincronización. Solo vuelve el calendario de una sola vista.
