# Prompts del taller, listos para copiar

Todos los prompts del recorrido de PMO, BA y BI, en el orden de los pasos del
[README](README.md).

Dos reglas antes de empezar, que valen para todos:

**Sé específico.** El mismo pedido, mal escrito, da basura. `Muéstrame las ventas`
no dice ni de qué tabla, ni de qué período, ni en qué formato. `Muéstrame el monto
total de venta de repuestos por mes de los últimos doce meses desde
inchcape_workshop.ops.fact_parts_sales, formateado como tabla` sí.

**Iterá, no reescribas.** Cuando la respuesta va en la dirección correcta pero le
falta algo, pedí el ajuste encima de lo que ya hay: `Ahora agregale una columna con
el porcentaje del total`. Empezar de cero cada vez es la forma más lenta de
trabajar con Genie Code.

## Dos caminos para el mismo resultado

Las secciones 2, 3 y 4 construyen algo: un notebook, un dashboard, una app. Cada
una viene en dos versiones del mismo pedido.

**El camino corto** es un solo prompt que lo pide todo de una. Cuando sale bien,
llegás al resultado en un paso, y es la forma en que vas a trabajar el día que ya
le tengas la mano tomada a la herramienta.

**El camino paso a paso** parte el mismo pedido en prompts chicos. Es más lento,
pero cada pieza se ve nacer por separado.

Elegí el corto y, si algo sale mal, caé al paso a paso. Un error genérico sobre un
prompt de seis visualizaciones no te dice cuál de las seis lo causó; el mismo error
sobre un prompt de una visualización sí. Aislar es más rápido que adivinar. Los dos
caminos llegan al mismo lado, así que podés cambiar de uno al otro a mitad de
camino: pedí con el corto lo que ya funcionó y con el paso a paso lo que no.

Las secciones 1 y 5 no tienen esa división: son preguntas a Genie, no
construcción. Cada pregunta se responde sola y se encadena con la anterior.

## 1. Preguntarle a los datos en español

Para el espacio de Genie del Paso 1. Van en orden, cada una construye sobre la anterior.

```
¿Cómo evolucionó la venta de repuestos mes a mes en los últimos doce meses?
Muéstrame el monto total en dólares por mes.
```

```
Compara la venta mensual de repuestos separando el proveedor Nippon Parts
del resto de los proveedores. ¿Ves algo raro en algún mes?
```

```
En marzo y abril de 2026 la venta de Nippon Parts cae fuerte.
¿Qué porcentaje de las combinaciones de punto y repuesto estuvieron
en quiebre de stock cada mes? Compara marzo contra el promedio del resto del año.
```

```
¿Cuánto dinero se perdió por horas de bahía detenidas esperando repuesto,
mes a mes? Usa la columna costo_espera_usd de las órdenes de taller.
```

```
Dame los diez puntos de la red con mayor costo de espera acumulado
entre el 2 de marzo y el 17 de abril de 2026. Incluye el país, la ciudad
y la cantidad de bahías de cada punto.
```

```
¿Hay algún incidente de abastecimiento registrado que explique
lo que pasó en marzo de 2026?
```

```
¿Qué familias de repuesto fueron las más afectadas durante el incidente?
Compara las horas promedio de espera por familia dentro del período del
incidente contra el resto del año.
```

Para corregir a Genie cuando se equivoca, sin empezar de nuevo:

```
El resultado está mal porque estás sumando las órdenes que todavía están
abiertas. Filtra solo las que tienen estado Cerrada y recalculá.
```

```
Necesito el resultado por país, no consolidado. Agregá la columna pais
de dim_dealer y agrupá por ahí.
```

## 2. El cruce de calidad de la PMO

### Camino corto: un prompt, el notebook entero

```
Trabajo en la PMO de Inchcape. Antes de cada reporte oficial reviso a mano el
portafolio en Excel y siempre encuentro los mismos problemas. Crea un notebook
en SQL sobre el esquema inchcape_workshop.pmo que me deje toda esa revisión
automatizada, con una celda de markdown antes de cada consulta explicando qué
resuelve.

Sobre pmo_projects, una consulta por cada problema:
1. Proyectos cargados dos veces: mismo nombre con proyecto_id distinto.
   Muéstrame los pares y cuánto presupuesto se está contando doble.
2. Proyectos sin fecha de cierre comprometida, o sea con fecha_fin_plan en NULL.
3. Proyectos donde el ejecutado ya superó el presupuesto aprobado.
   Muéstrame el sobregiro en dólares y en porcentaje.

Sobre pmo_milestones, el deslizamiento de cada hito en días, como la diferencia
entre fecha_real y fecha_plan. Dame por proyecto el deslizamiento promedio, el
peor hito y cuántos hitos están en estado Vencido. Uní con pmo_projects para
traer el nombre del proyecto y su líder.

Sobre pmo_budget, el seguimiento presupuestal mensual del portafolio:
presupuesto del mes, ejecutado del mes, desviación en dólares y desviación
acumulada del año, con una columna semáforo que diga Verde si la desviación
acumulada está por debajo del 5%, Amarillo hasta el 15% y Rojo por encima.

Una vista llamada inchcape_workshop.pmo.vw_alertas_portafolio que consolide los
tres problemas de pmo_projects, con una columna tipo_alerta que diga cuál de los
tres es, una columna impacto_usd, una columna detalle y las columnas
proyecto_id, nombre, lider y pais del proyecto involucrado. Agrégale comentarios
a la vista y a cada columna, para que cualquiera en la PMO la entienda sin
preguntarme.

Y de último, el cruce con la operación, que es la pregunta que nadie puede
responder hoy: por país, el costo de espera acumulado en ops.fact_workorder
durante el incidente del 2 de marzo al 17 de abril de 2026, cuántos proyectos
del programa Plan Posventa 360 hay en ese país, cuánto presupuesto tienen
asignado y el avance promedio reportado. Quiero ver si estamos invirtiendo
donde más duele.

Todo con nombres de columna en español y ordenado de mayor a menor impacto
en dinero.
```

### Camino paso a paso

El primero, que además crea el notebook donde va a vivir el resto:

```
Trabajo en la PMO de Inchcape. Antes de cada reporte oficial reviso a mano el
portafolio en Excel y siempre encuentro los mismos tres problemas. Crea un
notebook con esa revisión en SQL sobre la tabla inchcape_workshop.pmo.pmo_projects:

1. Proyectos cargados dos veces: mismo nombre con proyecto_id distinto.
   Muéstrame los pares y cuánto presupuesto se está contando doble.
2. Proyectos sin fecha de cierre comprometida, o sea con fecha_fin_plan en NULL.
3. Proyectos donde el ejecutado ya superó el presupuesto aprobado.
   Muéstrame el sobregiro en dólares y en porcentaje.

Devuélveme una consulta por cada problema, con nombres de columna en español y
ordenadas de mayor a menor impacto en dinero.
```

Pedile el notebook de forma explícita. `Quiero que me armes esa revisión` te
devuelve texto en el chat; `Crea un notebook` te devuelve el notebook.

El deslizamiento de fechas, que es la otra mitad del trabajo de la PMO:

```
Sobre inchcape_workshop.pmo.pmo_milestones, calculá el deslizamiento de cada
hito en días, como la diferencia entre fecha_real y fecha_plan. Después dame
por proyecto: el deslizamiento promedio, el peor hito, y cuántos hitos están
en estado Vencido. Uní con pmo_projects para traer el nombre del proyecto y
su líder. Ordená por deslizamiento promedio descendente.
```

Presupuesto contra ejecutado, mes a mes:

```
Con inchcape_workshop.pmo.pmo_budget, armame el seguimiento presupuestal
mensual del portafolio: presupuesto del mes, ejecutado del mes, desviación
en dólares y desviación acumulada del año. Agregá una columna semáforo que
diga Verde si la desviación acumulada está por debajo del 5%, Amarillo hasta
el 15% y Rojo por encima. Todo en español.
```

Consolidar en una vista reutilizable:

```
Convierte las tres consultas en una sola vista llamada
inchcape_workshop.pmo.vw_alertas_portafolio, con una columna tipo_alerta que
diga cuál de los tres problemas es, una columna impacto_usd, una columna detalle
y las columnas proyecto_id, nombre, lider y pais del proyecto involucrado.
Agrégale comentarios a la vista y a cada columna explicando qué es, para que
cualquiera en la PMO la entienda sin preguntarme.
```

El cruce entre los dos mundos, portafolio y operación. Esta es la pregunta que
nadie puede responder hoy:

```
Quiero conectar el portafolio con la operación. Los proyectos del programa
Plan Posventa 360 se abrieron para resolver los quiebres de stock. Armame una
consulta que muestre, por país: el costo de espera acumulado durante el
incidente de marzo y abril de 2026, cuántos proyectos del programa
Plan Posventa 360 hay en ese país, cuánto presupuesto tienen asignado, y el
avance promedio reportado. Quiero ver si estamos invirtiendo donde más duele.
```

## 3. El dashboard

### Camino corto: un prompt, el tablero entero

Pegalo en el asistente del dashboard. Es el camino que más falla de los tres,
porque le pide al asistente seis visualizaciones y tres filtros en una sola
pasada. Si te devuelve `Unable to render visualization` en alguna, o un error
sin explicación, no lo pelees: pasate al paso a paso para la pieza que falló.

```
Armame un dashboard completo para el comité de seguimiento de posventa de
Inchcape Andina. Lee del catálogo inchcape_workshop, esquemas ops y pmo.

El tablero cuenta una historia en dos mitades: arriba qué pasó en la operación,
abajo cómo va el portafolio de proyectos que se abrió para arreglarlo.

Necesito seis visualizaciones, en este orden:

1. Líneas. Suma de monto_usd por mes de los últimos doce meses, con dos series:
   el proveedor Nippon Parts y todos los demás juntos. Fuente: fact_parts_sales
   unida con dim_material por material_id. Los materiales sin proveedor cuentan
   como resto.
   Título: Venta de repuestos, Nippon Parts contra el resto.

2. Barras. Porcentaje de combinaciones de punto y repuesto en quiebre de stock
   por mes, desde fact_stock, usando fecha_snapshot y la bandera en_quiebre.
   Pintá de rojo las barras que superen el 8%.
   Título: Quiebre de stock mensual.

3. Barras con línea de referencia. Suma de costo_espera_usd por mes desde
   fact_workorder, por fecha_apertura, más una línea horizontal con el promedio
   mensual del año, para que se vea cuánto se sale marzo.
   Título: Impacto en dólares de las bahías detenidas.

4. Mapa de calor. Suma de costo_espera_usd por país y por familia de repuesto,
   solo para las órdenes abiertas entre el 2 de marzo y el 17 de abril de 2026.
   Uní fact_workorder con dim_dealer por dealer_id para el país, y con
   dim_material por material_id_principal para la familia.
   Título: Dónde se concentró el daño.

5. Barras horizontales más tabla al lado. Cantidad de proyectos por estado desde
   pmo_projects, y la tabla con presupuesto_usd total y ejecutado_usd total por
   estado.
   Título: Semáforo del portafolio.

6. Tabla. Los diez proyectos de mayor sobregiro: nombre, lider, pais,
   presupuesto_usd, ejecutado_usd, sobregiro en dólares y sobregiro en
   porcentaje. Solo los que tienen ejecutado_usd mayor que presupuesto_usd,
   ordenados por sobregiro en dólares descendente.
   Título: Proyectos por encima del presupuesto.

Además:
- Un filtro por país y otro por familia de repuesto, aplicados a todas las
  visualizaciones que tengan esas columnas disponibles.
- Un filtro de rango de fechas que afecte a las cuatro visualizaciones de
  operación.
- Todos los títulos, etiquetas de eje y nombres de columna en español.
- Los montos en dólares, con separador de miles y sin decimales.

Creá primero los datasets que hagan falta y después las visualizaciones sobre
ellos. Si dos visualizaciones pueden compartir el mismo dataset, compartilo en
vez de duplicar la consulta.
```

### Camino paso a paso: un prompt por gráfico

Uno por gráfico, en este orden. Cada uno se pega en el asistente del dashboard
y se revisa antes de pasar al siguiente.

```
Un gráfico de líneas con la venta mensual de repuestos en dólares de los
últimos doce meses. Dos líneas: una con el proveedor Nippon Parts y otra con
todos los demás proveedores juntos. Título: Venta de repuestos, Nippon Parts
contra el resto.
```

```
Un gráfico de barras con el porcentaje de combinaciones de punto y repuesto en
quiebre de stock por mes, desde inchcape_workshop.ops.fact_stock. El porcentaje
es filas en quiebre sobre filas totales del mes, no el conteo. Pintá de rojo las
barras que superen el 8%. Título: Quiebre de stock mensual.
```

```
Un gráfico de barras con la suma de costo_espera_usd por mes desde
inchcape_workshop.ops.fact_workorder. Agregale una línea con el promedio
mensual del año para que se vea cuánto se sale marzo. Título: Impacto en
dólares de las bahías detenidas.
```

```
Un mapa de calor con la suma de costo_espera_usd por país y por familia de
repuesto, solo para las órdenes abiertas entre el 2 de marzo y el 17 de abril
de 2026. Uní fact_workorder con dim_dealer para el país y con dim_material
para la familia. Título: Dónde se concentró el daño.
```

```
Un gráfico de barras horizontales con la cantidad de proyectos por estado
desde inchcape_workshop.pmo.pmo_projects, y al lado una tabla con el
presupuesto total y el ejecutado total por estado. Título: Semáforo del
portafolio.
```

```
Una tabla con los diez proyectos de mayor sobregiro: nombre, líder, país,
presupuesto aprobado, ejecutado, sobregiro en dólares y sobregiro en
porcentaje. Solo los que tienen ejecutado mayor al presupuesto. Ordená por
sobregiro en dólares descendente. Título: Proyectos por encima del presupuesto.
```

Los filtros, al final, cuando los seis gráficos ya están:

```
Agregá al dashboard un filtro por país y otro por familia de repuesto que
apliquen a todos los gráficos que tengan esas columnas disponibles, y un filtro
de rango de fechas que afecte a los cuatro gráficos de operación.
```

### Ajustes, para cualquiera de los dos caminos

**Cuando algo salga distinto de lo que esperabas**, no rehagas el tablero:
corregí encima. Estos son los ajustes que más se piden.

```
El gráfico de quiebre de stock está tomando todas las filas de fact_stock.
Necesito el porcentaje sobre el total de combinaciones de cada mes, no el
conteo absoluto. Recalculalo como filas en quiebre sobre filas totales del mes.
```

```
El filtro de país no está afectando al mapa de calor. Conectalo también a esa
visualización.
```

```
Movéme la tabla de proyectos sobregirados arriba del semáforo, y hacé que
ocupe todo el ancho.
```

Cuando una visualización queda con el cartel `Unable to render visualization`,
borrala y pedila sola, con el prompt del paso a paso que le corresponde. Antes
de eso, este suele alcanzar:

```
La visualización de venta de repuestos quedó con el error Unable to render
visualization. Borrala y armala de nuevo desde cero, con el dataset más simple
que sirva: una consulta que devuelva mes, serie y monto, y un gráfico de líneas
sobre esas tres columnas.
```

## 4. La app interna

### Camino corto: un prompt, la app completa

Las dos pestañas y el cache, todo en el mismo pedido. Los tres primeros bloques
parecen burocracia y no lo son: son las tres cosas que el modelo inventa cuando
no se las decís. Están explicadas debajo del prompt.

```
Escribime una app de Streamlit para Databricks Apps que le sirva a la PMO de
Inchcape para revisar el portafolio antes del comité. Dame todos los archivos
que necesita para desplegar: app.py, app.yaml y requirements.txt.

Puerto. En app.yaml el comando es exactamente
command: ["streamlit", "run", "app.py"], sin --server.port y sin
--server.address. Databricks Apps le asigna el puerto por la variable
DATABRICKS_APP_PORT y Streamlit lo toma solo.

Conexión. Usá exactamente este patrón, que es el que funciona dentro de
Databricks Apps:

    import os
    from databricks.sdk.core import Config
    from databricks import sql

    cfg = Config()
    conn = sql.connect(
        server_hostname=cfg.host,
        http_path=f"/sql/1.0/warehouses/{os.environ['DATABRICKS_WAREHOUSE_ID']}",
        credentials_provider=lambda: cfg.authenticate,
    )

No inventes variables de entorno: DATABRICKS_SERVER_HOSTNAME,
DATABRICKS_HTTP_PATH y DATABRICKS_TOKEN no existen acá y llegan en None. En
app.yaml declará el warehouse como recurso, nunca con el id escrito a mano:

    env:
      - name: DATABRICKS_WAREHOUSE_ID
        valueFrom: sql-warehouse

Esquema. Antes de escribir el código, consultá el esquema real de
inchcape_workshop.pmo.vw_alertas_portafolio y los valores distintos de su
columna de tipo de alerta, y construí la app contra lo que encuentres. Esa
vista la creé yo con Genie y no sé de memoria cómo quedaron los nombres. No
asumas que la columna de monto se llama monto_usd ni que las etiquetas de
alerta son las que vos supondrías.

Cache. Envolvé cada lectura en una función con st.cache_data y un tiempo de
vida de diez minutos. Todo el filtrado se hace en memoria sobre el DataFrame ya
cargado, no volviendo a consultar. Si la app consulta cada vez que muevo un
filtro, se vuelve inusable.

Pestaña 1, Alertas del portafolio, desde
inchcape_workshop.pmo.vw_alertas_portafolio:
- Arriba, tres tarjetas grandes: cantidad de proyectos duplicados, cantidad sin
  fecha de cierre comprometida, y dólares de sobregiro total. Las tres salen de
  filtrar por tipo de alerta, con los valores reales que encontraste en la vista.
- Abajo, una tabla con filtros por tipo de alerta y por país. Si la vista no
  tiene columna de país, dejá solo el de tipo de alerta y decímelo.
- Un botón que exporte a CSV la tabla ya filtrada, no la tabla completa.

Pestaña 2, Seguimiento presupuestal, desde inchcape_workshop.pmo.pmo_budget,
columnas mes, presupuesto_mes_usd y ejecutado_mes_usd:
- Un gráfico de barras agrupadas con presupuesto contra ejecutado, mes a mes.
- Debajo, la tabla mensual con desviación en dólares, desviación acumulada del
  año y una columna semáforo: Verde por debajo del 5%, Amarillo hasta el 15%,
  Rojo por encima.

Detalles que aplican a las dos pestañas:
- Todo el texto de la interfaz en español.
- Montos en dólares, con separador de miles y sin decimales.
- Si una consulta falla, mostrá el mensaje de error en la interfaz en vez de
  reventar con el stack trace.
```

**Por qué están esos tres bloques.** Los tres salieron de correr este mismo
prompt sin ellos. Cada uno rompe la app de una forma distinta, y ninguna de las
tres es obvia mirando la pantalla.

| Lo que el modelo inventa | Cómo se ve la falla |
|---|---|
| `--server.port=8080` en `app.yaml` | La URL responde **App Not Available**. Y lo peor: los logs dicen `App started successfully` y el estado queda en `RUNNING`, así que todo parece bien. El proxy de Apps golpea el puerto 8000 y Streamlit está escuchando en el 8080. |
| `DATABRICKS_SERVER_HOSTNAME`, `DATABRICKS_HTTP_PATH`, `DATABRICKS_TOKEN` | No existen en Apps, llegan en `None` y la conexión falla. Lo que Apps sí inyecta es `DATABRICKS_CLIENT_ID` y `DATABRICKS_CLIENT_SECRET`, que es justo lo que `Config()` lee solo. |
| Nombres de columna y etiquetas de la vista | La app abre, no tira ningún error, y las tres tarjetas muestran cero. Es la más cara de encontrar de las tres. |

Hay un cuarto problema que ningún prompt puede resolver, porque no es código: la
app corre con su propia identidad y esa identidad nace sin acceso a los datos.
Eso se arregla con los `GRANT` del [Paso 4 del README](README.md#paso-4-la-app-interna).

### Camino paso a paso

Los bloques de puerto, conexión y esquema van igual: no son un lujo del camino
corto, son lo que hace que la app abra. Primero la app mínima que despliega y
se ve:

```
Escribime una app de Streamlit para Databricks Apps que le sirva a la PMO de
Inchcape para revisar el portafolio antes del comité. Dame app.py, app.yaml y
requirements.txt.

Puerto. En app.yaml el comando es exactamente
command: ["streamlit", "run", "app.py"], sin --server.port y sin
--server.address. Databricks Apps asigna el puerto por DATABRICKS_APP_PORT.

Conexión. Usá exactamente este patrón:

    import os
    from databricks.sdk.core import Config
    from databricks import sql

    cfg = Config()
    conn = sql.connect(
        server_hostname=cfg.host,
        http_path=f"/sql/1.0/warehouses/{os.environ['DATABRICKS_WAREHOUSE_ID']}",
        credentials_provider=lambda: cfg.authenticate,
    )

y en app.yaml declará el warehouse como recurso:

    env:
      - name: DATABRICKS_WAREHOUSE_ID
        valueFrom: sql-warehouse

Esquema. Antes de escribir el código, consultá el esquema real de
inchcape_workshop.pmo.vw_alertas_portafolio y los valores distintos de su
columna de tipo de alerta, y construí la app contra eso. No asumas nombres.

Contenido:
- Arriba, tres tarjetas grandes: cantidad de proyectos duplicados, cantidad sin
  fecha de cierre comprometida, y dólares de sobregiro total.
- Abajo, una tabla con filtros por tipo de alerta y por país.
- Un botón que exporte a CSV la tabla ya filtrada.
- Todo el texto de la interfaz en español.
```

Desplegala y comprobá que abre antes de pedirle nada más. Recién ahí, la
segunda pestaña:

```
Agregale a la app una segunda pestaña con el seguimiento presupuestal mensual
del portafolio, leyendo inchcape_workshop.pmo.pmo_budget: un gráfico de barras
agrupadas con presupuesto contra ejecutado mes a mes, y abajo la tabla con la
desviación y el semáforo.
```

Y por último el cache, que es el ajuste que se nota apenas la usás:

```
La app tarda mucho en cargar porque consulta cada vez que muevo un filtro.
Agregale cache a la lectura de datos con un tiempo de vida de diez minutos,
y que el filtrado se haga en memoria.
```

### Cuando falla

El prompt más útil de todo el taller, tomes el camino que tomes:

```
La app falló al desplegar. Este es el log completo:

[pega acá el log tal cual, sin resumirlo]

Corregí el problema y explicame en una línea qué estaba mal.
```

**Si la URL dice App Not Available**, ese prompt no te va a servir, porque no
hay ningún error en el log: el despliegue salió bien y la app está corriendo.
Buscá en el log la línea `Starting app with command:`. Si dice `--server.port=8080`,
ese es el problema completo.

```
La app despliega bien y el log dice App started successfully, pero la URL
responde App Not Available. En el log veo que arranca con
--server.port=8080. Corregí app.yaml para que el comando sea exactamente
["streamlit", "run", "app.py"], sin flags de puerto ni de address, para que
Streamlit tome el puerto que le asigna Databricks Apps.
```

**Si la app abre pero las tarjetas muestran cero**, no es la conexión: es que
los nombres no coinciden.

```
La app abre pero las tres tarjetas muestran cero y la tabla sí trae filas.
Consultá los valores distintos de la columna de tipo de alerta en
inchcape_workshop.pmo.vw_alertas_portafolio y el nombre real de la columna de
monto, y corregí los filtros de las tarjetas para que usen esos valores.
```

## 5. El asistente de estatus

Estas cinco se guardan como preguntas sugeridas del espacio de Genie, para que
cualquiera en la PMO las use sin saber nada de datos.

```
¿Qué proyectos del portafolio están en riesgo hoy y por qué?
```

```
¿Cuál es el presupuesto real del portafolio una vez que descontamos
los proyectos que están cargados dos veces?
```

```
Dame el estatus del programa Plan Posventa 360: cuántos proyectos tiene,
avance promedio, cuántos hitos vencidos y cuánto presupuesto lleva ejecutado.
```

```
¿Qué hitos vencen en los próximos treinta días y quién es el responsable
de cada uno?
```

```
Si tuviera que escalar tres proyectos al comité de esta semana,
¿cuáles serían y con qué dato los justifico?
```
