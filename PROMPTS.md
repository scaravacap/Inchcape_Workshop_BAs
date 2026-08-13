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

El prompt principal del Paso 2:

```
Trabajo en la PMO de Inchcape. Antes de cada reporte oficial reviso a mano el
portafolio en Excel y siempre encuentro los mismos tres problemas. Quiero que me
armes esa revisión en SQL sobre la tabla inchcape_workshop.pmo.pmo_projects:

1. Proyectos cargados dos veces: mismo nombre con proyecto_id distinto.
   Muéstrame los pares y cuánto presupuesto se está contando doble.
2. Proyectos sin fecha de cierre comprometida, o sea con fecha_fin_plan en NULL.
3. Proyectos donde el ejecutado ya superó el presupuesto aprobado.
   Muéstrame el sobregiro en dólares y en porcentaje.

Devuélveme una consulta por cada problema, con nombres de columna en español y
ordenadas de mayor a menor impacto en dinero.
```

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

Un solo prompt para el tablero completo. Pegalo en el asistente del dashboard.

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

## 4. La app interna

Un solo prompt para la app completa, con las dos pestañas y el cache ya adentro.

```
Escribime una app de Streamlit para Databricks Apps que le sirva a la PMO de
Inchcape para revisar el portafolio antes del comité. Dame todos los archivos
que necesita para desplegar: app.py, app.yaml y requirements.txt.

Conexión a datos:
- Leé con databricks-sql-connector, usando las credenciales que Databricks Apps
  inyecta por variables de entorno. No hardcodees ningún token, host ni
  warehouse id.
- Envolvé cada lectura en una función cacheada con st.cache_data y un tiempo de
  vida de diez minutos. Todo el filtrado se hace en memoria sobre el DataFrame
  ya cargado, no volviendo a consultar. Si la app consulta cada vez que muevo un
  filtro, se vuelve inusable.

Pestaña 1, Alertas del portafolio. Lee de
inchcape_workshop.pmo.vw_alertas_portafolio:
- Arriba, tres tarjetas grandes: cantidad de proyectos duplicados, cantidad sin
  fecha comprometida, y dólares de sobregiro total.
- Abajo, una tabla filtrable por tipo de alerta y por país.
- Un botón que exporte a CSV la tabla ya filtrada, no la tabla completa.

Pestaña 2, Seguimiento presupuestal. Lee de inchcape_workshop.pmo.pmo_budget,
columnas mes, presupuesto_mes_usd y ejecutado_mes_usd:
- Un gráfico de barras agrupadas con presupuesto contra ejecutado, mes a mes.
- Debajo, la tabla mensual con desviación en dólares, desviación acumulada del
  año y una columna semáforo: Verde por debajo del 5%, Amarillo hasta el 15%,
  Rojo por encima.

Detalles que aplican a las dos pestañas:
- Todo el texto de la interfaz en español.
- Montos en dólares, con separador de miles y sin decimales.
- Si una consulta falla, mostrá un mensaje claro en la interfaz en vez de
  reventar con el stack trace.
```

Cuando falla, el prompt más útil de todo el taller:

```
La app falló al desplegar. Este es el log completo:

[pega acá el log tal cual, sin resumirlo]

Corregí el problema y explicame en una línea qué estaba mal.
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
