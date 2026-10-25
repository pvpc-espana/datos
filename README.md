# PVPC España · Datos

Precios del **PVPC** (Precio Voluntario para el Pequeño Consumidor, tarifa 2.0TD) de la
electricidad en España, hora a hora, publicados como ficheros JSON estáticos en
**<https://datos.pvpc.linaresdigital.com/v1/>** para la aplicación
[pvpc.linaresdigital.com](https://pvpc.linaresdigital.com).

Este repositorio no contiene código: sólo los datos que genera y publica automáticamente
[`nucleo`](https://github.com/pvpc-espana/nucleo). La interfaz web está en
[`frontal`](https://github.com/pvpc-espana/frontal).

---

## 1. Origen de los datos

| | |
|---|---|
| **Fuente** | [Red Eléctrica de España (REE)](https://www.ree.es), operador del sistema eléctrico, a través de su plataforma [e·sios](https://www.esios.ree.es) |
| **Indicador** | `1001` · *Término de facturación de energía activa del PVPC 2.0TD* |
| **Zona** | Península (`geo_id` 8741), que aplica también a Baleares y Canarias. Ceuta y Melilla tienen precios propios y no se incluyen. |
| **Acceso** | API de e·sios (`api.esios.ree.es`) con un *token* personal concedido por REE, guardado como secreto de Actions (`TESTIGO`) en `nucleo` y nunca publicado. Si e·sios falla se usa la API pública [REData](https://www.ree.es/es/datos/apidatos) (`apidatos.ree.es`). |
| **Publicación por REE** | Los precios del día siguiente, a partir de las 20:00 (hora peninsular). |
| **Granularidad** | Horaria: 24 periodos al día, 23 o 25 los días de cambio de hora. |
| **Impuestos** | No incluidos (precio del término de energía, sin IVA ni impuesto eléctrico). |

### Transformaciones aplicadas

Los valores se publican **sin alterar su contenido ni su sentido**. Sólo se aplican estas
transformaciones técnicas:

- Unidad: de €/MWh (como los da REE) a **€/kWh**, dividiendo entre 1000 y redondeando a
  5 decimales.
- Hora: cada periodo indica su inicio en hora peninsular **con su desfase horario
  explícito** (`+01:00` o `+02:00`), para que los días de 23 y 25 horas no sean ambiguos.
- Resumen del día: mínimo, máximo y media aritmética de los periodos.
- Validación: un día incompleto, con huecos o con precios fuera de rango **no se publica**.

## 2. Condiciones de uso de los datos

Los datos pertenecen a su fuente y se obtienen, alojan y redistribuyen respetando sus
condiciones:

- **e·sios**: [Aviso legal y política de privacidad de e·sios](https://www.esios.ree.es/es/aviso-legal-y-politica-de-privacidad).
- **Red Eléctrica**: [Aviso legal de REE](https://www.ree.es/en/legal-notice-privacy-policy)
  y condiciones de uso de su información estadística: uso público y gratuito, salvo
  fines comerciales o publicitarios, indicando el origen (www.ree.es) y la fecha de
  actualización cuando la haya ([ejemplo en las series estadísticas de REE](https://www.ree.es/en/datos/publications/national-statistical-series)).
- **Marco legal de reutilización**: [Ley 37/2007, sobre reutilización de la información
  del sector público](https://www.boe.es/buscar/act.php?id=BOE-A-2007-19814), cuyo
  artículo 8 exige no desnaturalizar la información, citar la fuente y mencionar la
  fecha de la última actualización.

Cómo se cumplen en este proyecto:

| Condición | Cómo se respeta |
|---|---|
| Citar la fuente | Cada fichero lleva `"fuente": "REE"`; la web muestra «Datos de Red Eléctrica (e·sios)» con enlace, y este README lo detalla. |
| Fecha de actualización | `v1/indice.json` incluye `actualizado`; el historial de Git registra cuándo cambió cada día. |
| No alterar ni desnaturalizar | Sólo cambio de unidad, redondeo y validación (ver arriba), documentados. |
| Uso no comercial | Proyecto personal, gratuito, sin publicidad ni fines comerciales. |
| Sin garantías | Los datos se ofrecen tal cual. Ni REE ni este proyecto responden de su uso; para fines oficiales o de facturación consulta siempre [e·sios](https://www.esios.ree.es). |
| Credenciales | El *token* de e·sios es personal: sólo se usa desde `nucleo` y nunca se publica ni se comparte. |

Si reutilizas estos ficheros, mantén la atribución a **Red Eléctrica de España (e·sios)**
y revisa sus condiciones vigentes en los enlaces anteriores: prevalecen sobre este
resumen.

## 3. Contenido y formato

```
docs/                       ← publicado con GitHub Pages (rama main, carpeta /docs)
├── CNAME                   datos.pvpc.linaresdigital.com
├── .nojekyll               sin procesado de Jekyll: se sirve tal cual
├── index.html              página informativa
└── v1/                     versión 1 del formato
    ├── indice.json
    └── AAAA/MM/AAAA-MM-DD.json
```

### `v1/indice.json`

```json
{ "version": 1, "primera": "2026-10-16", "ultima": "2026-10-26", "actualizado": "2026-10-25T20:33:04Z" }
```

Rango de días disponibles y momento (UTC) de la última publicación. Sólo se reescribe
cuando cambia algún día.

### `v1/AAAA/MM/AAAA-MM-DD.json`

```json
{
  "fecha": "2026-10-25",
  "zona": "peninsula",
  "unidad": "EUR/kWh",
  "resolucion": "PT1H",
  "fuente": "REE",
  "resumen": { "min": 0.04138, "max": 0.34683, "media": 0.18234 },
  "precios": [
    { "inicio": "2026-10-25T00:00:00+02:00", "precio": 0.26199 },
    { "inicio": "2026-10-25T02:00:00+02:00", "precio": 0.2263 },
    { "inicio": "2026-10-25T02:00:00+01:00", "precio": 0.21201 }
  ]
}
```

(Extracto del día de 25 horas: la hora 02:00 aparece dos veces, en verano y en invierno.)

| Campo | Descripción |
|---|---|
| `fecha` | Día en hora peninsular (`AAAA-MM-DD`). |
| `precios[].inicio` | Inicio del periodo, ISO 8601 con desfase horario. |
| `precios[].precio` | €/kWh con 5 decimales (puede ser negativo). |
| `resolucion` | `PT1H` (horaria); el formato admite `PT15M` si REE pasara a cuartos de hora. |
| `resumen` | `min`, `max` y `media` del día en €/kWh. |

### Acceso

- URL principal: `https://datos.pvpc.linaresdigital.com/v1/...` (HTTPS, CORS abierto:
  GitHub Pages responde con `Access-Control-Allow-Origin: *`).
- Respaldo: `https://raw.githubusercontent.com/pvpc-espana/datos/main/docs/v1/...`
- Un día aún no publicado responde **404**.
- GitHub Pages indica una caché de 10 minutos (`max-age=600`).
- Cambios incompatibles de formato se publicarían en una nueva versión (`v2/`), sin
  romper `v1/`.

## 4. Flujo de actualización

```
REE publica (≥ 20:00)          nucleo · GitHub Actions                     datos · GitHub Pages
┌──────────────────┐    ┌──────────────────────────────────────┐    ┌──────────────────────────┐
│ api.esios.ree.es │◄───│ cron cada 20 min (tarde) / 1 h (mañana) │    │ main/docs                │
│  (TESTIGO)       │    │ 1. «Decidir»: ¿falta hoy, o mañana     │    │  └ commit «PVPC fecha»   │
└──────────────────┘    │    desde las 20:00? si no, termina     │───►│ pages-build-deployment   │
                        │ 2. descarga, valida y normaliza        │ SSH│  └ datos.pvpc.…/v1/      │
                        │ 3. commit + push sólo si hay cambios   │    └──────────────────────────┘
                        └──────────────────────────────────────┘
```

1. `nucleo` lanza su workflow cada 20 minutos de 18:07 a 22:47 UTC y cada hora de 04:17 a
   08:17 UTC. GitHub puede retrasar o descartar ejecuciones programadas, así que la
   frecuencia garantiza que alguna llegue a tiempo.
2. El paso «Decidir» calcula la hora peninsular y consulta qué hay publicado aquí. Sólo
   continúa si falta el día en curso o, desde las 20:00, el siguiente.
3. Descarga hoy y mañana desde e·sios, valida y escribe los JSON de forma determinista
   (misma entrada, mismos bytes).
4. Con la **clave de despliegue** (SSH, sólo con acceso a este repositorio) hace commit y
   push **sólo si algún fichero cambió**. Las correcciones de REE sobrescriben el día y
   quedan registradas en el historial.
5. El push dispara la publicación de GitHub Pages (`pages-build-deployment`), que tarda
   menos de un minuto.
6. Si a las 23:40 siguen sin estar los precios de mañana, o falta el día en curso, la
   ejecución falla y GitHub avisa por correo.

**Persistencia**: el repositorio es el archivo histórico. Cada día publicado queda
versionado en Git: se puede consultar cualquier versión anterior o reconstruir el
histórico completo.

## 5. Uso de recursos y límites de GitHub

Estimaciones con los tamaños reales de los ficheros (unos 2,2 KB por día, 0,4 KB
comprimidos con gzip):

| Recurso | Límite de GitHub | Uso estimado | Margen |
|---|---|---|---|
| Tamaño del sitio publicado | 1 GB ([límites de Pages](https://docs.github.com/en/pages/getting-started-with-github-pages/github-pages-limits)) | ~0,8 MB al año | > 1000 años |
| Tamaño del repositorio | 1 GB recomendado | ~1–2 MB al año con historial (hoy 0,6 MB) | Cientos de años |
| Transferencia | 100 GB/mes (límite flexible) | ~1–3 KB por visita (índice, día y precarga de los días vecinos, comprimidos) | Millones de visitas al mes |
| Publicaciones de Pages | 10 por hora (límite flexible, para publicación desde rama) | 1–2 al día (sólo cuando cambian los datos) | Amplio |
| Minutos de Actions (en `nucleo`) | Gratis e ilimitados con los ejecutores estándar en repositorios públicos ([facturación de Actions](https://docs.github.com/en/billing/managing-billing-for-github-actions/about-billing-for-github-actions)) | ~20 ejecuciones al día, casi todas de < 10 s | — |
| Peticiones a e·sios | Uso razonable con *token* personal | 2 por ejecución útil (hoy y mañana), unas 2–10 al día | — |

Otros puntos a vigilar:

- **Inactividad**: GitHub desactiva los workflows programados de un repositorio público
  tras 60 días sin actividad. El workflow de `nucleo` se reactiva a sí mismo en cada
  ejecución programada (paso «Mantener activo el workflow»).
- **Retrasos**: las ejecuciones programadas pueden llegar horas tarde; el diseño del
  paso 2 lo compensa sin publicar datos erróneos.

## Agradecimientos

Gracias a [Claude](https://claude.ai), de [Anthropic](https://www.anthropic.com), por su
inestimable ayuda en la creación y depuración de este proyecto.
