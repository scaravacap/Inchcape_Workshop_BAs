# Taller Inchcape: de reportero a constructor de herramientas

Guía para el grupo de **PMO, Business Analysts y desarrolladores de BI**.
Workshop Inchcape, Medellín, agosto 2026.

Este recorrido no pide escribir código. Todo lo que van a construir se lo van a
pedir en español a Genie Code, y ustedes revisan y ajustan el resultado. Al
terminar, cada uno se lleva un dashboard, una app interna y un asistente que
responde sobre los datos del portafolio, montados sobre un caso de posventa que
se puede investigar de punta a punta.

Corre entero dentro del navegador, en una cuenta gratuita. Nada que instalar.

## Antes de empezar

**Necesitás una cuenta de Databricks Free Edition.** Se crea en dos minutos:

1. Entra a [databricks.com/learn/free-edition](https://www.databricks.com/learn/free-edition).
2. Elige **Sign up with Google** o **Sign up with Microsoft**, o registra tu correo
   y confirma el código de seis dígitos que llega al buzón.
3. Databricks crea tu workspace y te deja adentro. No pide tarjeta de crédito.

Dos cosas que ayuda saber de Free Edition antes de arrancar:

- El cómputo es **serverless**, así que no hay clústeres que configurar ni apagar.
  No hay ninguna perilla de tamaño que puedas equivocar.
- Hay una **cuota diaria de uso**. Es de sobra para este taller, pero si dejás
  consultas corriendo en vano se puede agotar y el cómputo se apaga hasta el día
  siguiente. Cierra las pestañas que no estés usando.

## Paso 0. Traer el material y crear los datos

**Clona este repositorio dentro de tu workspace.**

1. En el menú de la izquierda, entra a **Workspace**.
2. Botón **Create**, luego **Git folder**.
3. Pega esta URL:

```
https://github.com/scaravacap/Inchcape_Workshop_BAs
```

4. Deja el proveedor en **GitHub** y confirma. Es un repositorio público, así que
   no te va a pedir usuario ni token.

**Corre el notebook de datos.**

Abre `notebooks/00_setup_datos.sql` y dale **Run all**. Tarda entre uno y tres
minutos. Es SQL puro: no instala nada ni sale a internet.

**Cómo sabés que funcionó:** la última celda te devuelve once tablas con estas
cantidades de filas.

| Tabla | Filas |
|---|---|
| ops.dim_dealer | 120 |
| ops.dim_material | 800 |
| ops.fact_parts_sales | 175.241 |
| ops.fact_stock | 499.200 |
| ops.fact_workorder | 60.000 |
| ops.fact_supply_incident | 40 |
| pmo.pmo_projects | 25 |
| pmo.pmo_milestones | 150 |
| pmo.pmo_budget | 300 |
| raw.sap_mara | 830 |
| raw.sap_vbap | 44.045 |

Si te salen exactamente esos números, tenés los mismos datos que todo el mundo en
la sala, y por lo tanto los mismos resultados.

**Ahora lee [HISTORIA.md](HISTORIA.md).** Son tres minutos y es lo que le da
sentido a todo lo que sigue. Sin eso, los ejercicios son consultas sueltas.

## Paso 0.5. Dispará la app ahora, aunque la vayas a usar al final

Crear una app tarda unos tres minutos en aprovisionar. En vez de esperar sentado
en el Paso 4, la creamos ahora y se va cocinando sola mientras trabajás.

1. Menú **Compute**, pestaña **Apps**, botón **Create app**.
2. Nombre: `alertas-portafolio-pmo`.
3. Plantilla: **Streamlit**.
4. En la sección de **Resources** o recursos de la app, agregá el
   **SQL warehouse** llamado `Serverless Starter Warehouse` con permiso de uso.
   Este paso es el que la gente olvida, y sin él la app no puede leer datos.
5. Dale crear y **dejala ahí**. Volvemos en el Paso 4.

Mientras se crea, seguí con el Paso 1.

## Paso 1. Preguntarle a los datos en español

**Objetivo:** responder preguntas de negocio sin escribir SQL, y descubrir el
incidente por tu cuenta.

**Qué hacer:** crea un espacio de Genie sobre los datos.

1. Menú de la izquierda, **Genie**.
2. **New**, y elige el catálogo `inchcape_workshop`.
3. Agrega los esquemas `ops` y `pmo` completos.
4. Guarda el espacio con el nombre `Posventa Inchcape Andina`.
5. En la configuración del espacio, pega el bloque de **Instrucciones** que está
   en [SKILLS.md](SKILLS.md). Ese texto le enseña a Genie el vocabulario de
   Inchcape, y es la diferencia entre respuestas buenas y respuestas raras.

**Preguntas para pegar**, en este orden. Cada una se apoya en la anterior.

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

**Cómo sabés que funcionó:** Genie te lleva sola hasta el incidente `INC-0001`, y
podés decir en una frase cuánta plata costó y en qué puntos se concentró.

**Si una respuesta sale mal**, no la aceptes. Escribile qué está mal, igual que le
dirías a un analista. Por ejemplo: `El monto está inflado porque estás contando las
líneas anuladas. Excluí las que tienen monto negativo y volvé a calcular.`

## Paso 2. El cruce que hoy hacés a mano en Excel

**Objetivo:** automatizar la revisión de calidad que la PMO hace antes de cada
reporte oficial. Duplicados, presupuesto pasado y fechas deslizadas.

**Qué hacer:** abre un notebook nuevo y usa Genie Code. Menú **Workspace**,
**Create**, **Notebook**. El panel de Genie Code se abre con el ícono de la chispa,
o con la paleta de comandos.

Los prompts completos de este paso están en
[PROMPTS.md, sección 2](PROMPTS.md#2-el-cruce-de-calidad-de-la-pmo), en dos
versiones. El **camino corto** es un solo prompt que arma el notebook entero. El
**camino paso a paso** pide una consulta por vez. Empezá por el corto; si algo
sale raro, el paso a paso te deja ver cuál consulta se rompió. Este es el primero
del paso a paso:

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

Fijate que le pide un notebook de forma explícita. `Quiero que me armes esa
revisión` te devuelve texto en el chat; `Crea un notebook` te devuelve el
notebook, que es lo que querés.

**Cómo sabés que funcionó:** encontrás **3 nombres de proyecto duplicados**,
**2 proyectos sin fecha comprometida** y **11 de los 25 por encima del
presupuesto**. Si te dan otros números, revisa la consulta con Genie Code antes
de seguir.

Luego pedile que lo convierta en algo reutilizable:

```
Convierte las tres consultas en una sola vista llamada
inchcape_workshop.pmo.vw_alertas_portafolio, con una columna tipo_alerta que
diga cuál de los tres problemas es, una columna impacto_usd, una columna detalle
y las columnas proyecto_id, nombre, lider y pais del proyecto involucrado.
Agrégale comentarios a la vista y a cada columna explicando qué es, para que
cualquiera en la PMO la entienda sin preguntarme.
```

## Paso 3. El dashboard de portafolio y posventa

**Objetivo:** un tablero que sirva para la reunión de seguimiento, construido sin
depender de nadie.

**Qué hacer:** menú **Dashboards**, **Create dashboard**. En la pestaña de datos,
agrega las tablas que necesites. Después usa el asistente describiéndole en
español lo que querés ver.

[PROMPTS.md, sección 3](PROMPTS.md#3-el-dashboard) trae los dos caminos: el
**corto**, un prompt que pide el tablero completo, y el **paso a paso**, seis
prompts, uno por gráfico. El corto es el más rápido cuando funciona, y también el
que más se rompe: seis visualizaciones en una sola pasada es mucho pedirle al
asistente, y si te devuelve `Unable to render visualization` no vas a saber cuál
de las seis lo causó. Ahí pedís esa sola con su prompt del paso a paso. Los seis
gráficos son:

1. Venta de repuestos por mes, con Nippon Parts separado del resto.
2. Porcentaje de quiebre de stock por mes.
3. Costo de espera en dólares por mes.
4. Mapa de calor de costo de espera por país y familia de repuesto.
5. Semáforo del portafolio: proyectos por estado, con presupuesto y ejecutado.
6. Top diez de proyectos por sobregiro.

Más los filtros de país, familia de repuesto y rango de fechas.

**Cómo sabés que funcionó:** el pico de marzo se ve a simple vista en tres de los
gráficos, y podés contar la historia completa moviendo un solo filtro de fecha.

**Si algo sale distinto** de lo que pediste, corregí encima en vez de rehacer el
tablero. Al final de la sección 3 están los ajustes que más se piden.

**Si te sobra tiempo**, agrega un filtro por país y otro por familia de repuesto,
y fíjate cómo cambia la conclusión cuando aislás Colombia.

## Paso 4. La app interna

**Objetivo:** pasar de entregar un reporte a entregar una herramienta. Esto es lo
que cambia el rol de la PMO.

Este es el paso más ambicioso del recorrido. La app que creaste en el Paso 0.5 ya
debería estar lista.

Free Edition permite hasta tres apps por cuenta, y las apaga solas a las
veinticuatro horas. Se prenden de nuevo con un botón.

**Qué hacer:**

1. **Dale permiso a la app sobre los datos.** La app corre con su propia identidad,
   un service principal, que nace sin acceso a nada. En la pantalla de tu app,
   copiá el nombre del service principal. Después, en un notebook, corré esto
   reemplazando el nombre:

```sql
GRANT USE CATALOG ON CATALOG inchcape_workshop TO `<service principal de tu app>`;
GRANT USE SCHEMA, SELECT ON SCHEMA inchcape_workshop.pmo TO `<service principal de tu app>`;
```

   Si te salta la app sin permisos y no entendés por qué, es casi siempre esto.

2. Abrí el código de la app y reemplazalo con lo que te genere Genie Code a partir
   de [PROMPTS.md, sección 4](PROMPTS.md#4-la-app-interna). También acá hay dos
   caminos: el **corto**, un prompt que pide la app entera con las dos pestañas y
   el cache, o el **paso a paso**, que arranca con la versión mínima y le agrega
   lo demás encima. Con una app, desplegar la versión mínima y verla abrir antes
   de pedirle más te ahorra depurar dos problemas a la vez.
3. Dale **Deploy** y esperá. El primer despliegue tarda unos minutos.

El prompt de la app mínima arranca así:

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

**Cómo sabés que funcionó:** abrís la URL de la app y ves las tres tarjetas con
los números del Paso 2. Ese es el momento en que la PMO deja de pedir reportes y
empieza a publicarlos.

**Si la app no arranca**, copia el error del log y pegáselo a Genie Code tal cual,
con la instrucción `Corregí esto`. Es la forma más rápida de resolverlo, y es
exactamente lo que haría yo.

## Paso 5. El asistente de estatus

**Objetivo:** que las preguntas repetidas de estatus se respondan solas.

**Qué hacer:** volvé al espacio de Genie del Paso 1 y trabajá su configuración
para que sirva como asistente de la PMO.

1. Agrega la vista `vw_alertas_portafolio` al espacio.
2. En **Instrucciones**, pega el bloque de asistente de PMO de
   [SKILLS.md](SKILLS.md).
3. Guarda como preguntas sugeridas las cinco de
   [PROMPTS.md, sección 5](PROMPTS.md#5-el-asistente-de-estatus).
4. Comparte el espacio con un compañero y pedile que pregunte algo que vos no
   hayas probado.

**Cómo sabés que funcionó:** alguien que nunca vio estos datos hace una pregunta
de estatus en español y recibe una respuesta correcta, con la tabla que la
respalda.

## Paso 6. Qué te llevás

Cinco minutos, sin pantalla. Responde para vos:

1. ¿Cuál de las cuatro cosas que construiste te ahorra más horas la próxima semana?
2. ¿Qué reporte que hoy armás a mano podrías reemplazar con lo del Paso 2?
3. ¿A quién de tu equipo le entregarías la app del Paso 4, y qué le pedirías que
   le agregue?

Esa tercera respuesta es la que quiero escuchar en el cierre de la tarde.

## Cuando algo se rompe

| Qué ves | Qué hacer |
|---|---|
| `Table or view not found` | El notebook del Paso 0 no terminó. Corré `Run all` de nuevo y esperá la celda de validación. |
| El warehouse aparece detenido | Prende solo con la primera consulta. La primera tarda entre veinte y cuarenta segundos, las siguientes son inmediatas. |
| Genie responde con columnas que no existen | Le falta contexto. Pegá el bloque de Instrucciones de [SKILLS.md](SKILLS.md) en la configuración del espacio. |
| Un número no coincide con esta guía | Corré de nuevo el notebook del Paso 0. Los datos son deterministas, así que si difieren es que el setup quedó a medias. |
| `Quota exceeded` o el cómputo no arranca | Se agotó la cuota diaria de la cuenta. Cierra pestañas con consultas corriendo. Si ya se agotó, se restablece al día siguiente. |
| La app queda en `Stopped` | Free Edition apaga las apps a las veinticuatro horas. Entra a **Compute**, **Apps**, y dale **Start**. |
| Genie Code genera código que falla | Pegale el error completo y escribile `Corregí esto`. No lo arregles a mano en el primer intento. |

## Mapa del repositorio

- [HISTORIA.md](HISTORIA.md), el caso de negocio y el modelo de datos. Leelo primero.
- [PROMPTS.md](PROMPTS.md), todos los prompts de los seis pasos, listos para copiar.
- [SKILLS.md](SKILLS.md), las instrucciones que le tenés que dar a Genie y a Genie
  Code para que trabajen con el vocabulario de Inchcape.
- [notebooks/00_setup_datos.sql](notebooks/00_setup_datos.sql), el generador de datos.

---

Cualquier duda después del taller, escríbanme. Saúl Caravaca, Solution Architect
de Databricks para Colombia y Costa Rica.
