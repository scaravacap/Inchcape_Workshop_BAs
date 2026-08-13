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
diga cuál de los tres problemas es, una columna impacto_usd y una columna
detalle. Agrégale comentarios a la vista y a cada columna explicando qué es,
para que cualquiera en la PMO la entienda sin preguntarme.
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

Uno por gráfico. Pegá cada uno en el asistente del dashboard.

```
Un gráfico de líneas con la venta mensual de repuestos en dólares de los
últimos doce meses. Dos líneas: una con el proveedor Nippon Parts y otra con
todos los demás proveedores juntos. Título: Venta de repuestos, Nippon Parts
contra el resto.
```

```
Un gráfico de barras con el porcentaje de combinaciones de punto y repuesto en
quiebre de stock por mes, desde inchcape_workshop.ops.fact_stock. Pintá de
rojo las barras que superen el 8%. Título: Quiebre de stock mensual.
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

Filtros, al final:

```
Agregá al dashboard un filtro por país y otro por familia de repuesto que
apliquen a todos los gráficos que tengan esas columnas disponibles.
```

## 4. La app interna

```
Escribime una app de Streamlit para Databricks Apps que le sirva a la PMO de
Inchcape para revisar el portafolio antes del comité. Requisitos:

- Lee de la vista inchcape_workshop.pmo.vw_alertas_portafolio usando
  databricks-sql-connector con las credenciales que Databricks Apps inyecta
  por variables de entorno. No hardcodees ningún token.
- Arriba, tres tarjetas grandes: cantidad de proyectos duplicados,
  cantidad sin fecha comprometida, y dólares de sobregiro total.
- Abajo, una tabla filtrable por tipo de alerta y por país.
- Un botón que exporte la tabla filtrada a CSV.
- Todo el texto de la interfaz en español.
- Incluí el archivo app.yaml y el requirements.txt que necesita.
```

Para iterar sobre la app una vez que arranca:

```
Agregale a la app una segunda pestaña con el seguimiento presupuestal mensual
del portafolio, usando la consulta de presupuesto contra ejecutado que armamos
antes. Un gráfico de barras agrupadas y abajo la tabla con el semáforo.
```

```
La app tarda mucho en cargar porque consulta cada vez que muevo un filtro.
Agregale cache a la lectura de datos con un tiempo de vida de diez minutos,
y que el filtrado se haga en memoria.
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
