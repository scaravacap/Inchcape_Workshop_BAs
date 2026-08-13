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

## Paso 1. Crea tu Genie Agent y pregúntale a tus datos

**Objetivo:** responder preguntas de negocio sin escribir SQL, y descubrir el
incidente por tu cuenta.

**Qué hacer:** crea un Genie Agent sobre los datos. Hay dos formas y el resultado
es el mismo.

**Pedíselo a Genie Code.** Crear un Genie Agent es una de las cosas que sabe
hacer solo, así que alcanza con una frase:

```
Creá un Genie Agent llamado Posventa Inchcape Andina con las tablas de los
esquemas ops y pmo del catálogo inchcape_workshop.
```

Te lo propone como un **asset** para revisar. Mirá la lista de tablas antes de
darle **Accept**.

**O crealo a mano**, desde el menú **Genie**: **New**, catálogo
`inchcape_workshop`, agregá los esquemas `ops` y `pmo` completos, y guardá con el
nombre `Posventa Inchcape Andina`.

**Con cualquiera de las dos, falta un paso**: abrí el agente y pegá el **bloque
1** de [SKILLS.md](SKILLS.md) en **Instrucciones**. Ese texto le enseña el
vocabulario de Inchcape, y es la diferencia entre respuestas buenas y respuestas
raras.

Este agente lo vas a usar dos veces. Acá, para investigar vos. Y al final del
Paso 2, cuando ya tengas la vista de alertas, para dejárselo abierto al resto de
la PMO.

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

**Si te sobra tiempo**, en
[PROMPTS.md, sección 1](PROMPTS.md#1-crea-tu-genie-agent-y-pregúntale-a-tus-datos)
hay una séptima pregunta, sobre qué familias de repuesto sufrieron más durante
el incidente, y más ejemplos de cómo corregir a Genie sin empezar de cero.

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

Con la vista lista, volvé un momento al Genie Agent del Paso 1: agregale
`vw_alertas_portafolio` como fuente y pegá el **bloque 3** de
[SKILLS.md](SKILLS.md) encima del bloque 1, sin borrar el 1. El bloque 1 le
enseñó el vocabulario; el 3 le enseña a responderle a alguien que entra a una
reunión en cinco minutos. Guardá también las cinco preguntas sugeridas de
[PROMPTS.md, sección 1](PROMPTS.md#abrirlo-al-resto-de-la-pmo): con eso cualquiera
en la PMO abre el agente y pregunta sin saber qué tablas hay.

## Paso 3. El dashboard de portafolio y posventa

**Objetivo:** un tablero que sirva para la reunión de seguimiento, construido sin
depender de nadie.

**Qué hacer:** menú **Dashboards**, **Create dashboard**. En la pestaña de datos,
agrega las tablas que necesites. Después usa el asistente describiéndole en
español lo que querés ver.

[PROMPTS.md, sección 3](PROMPTS.md#3-el-dashboard) trae los dos caminos: el
**corto**, un prompt que pide el tablero completo, y el **paso a paso**, seis
prompts, uno por gráfico. Empezá por el corto. Si una visualización no queda como
esperabas, pedila sola con su prompt del paso a paso, que es más rápido que
rehacer el tablero entero. Los seis gráficos son:

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

Este es el paso más ambicioso del recorrido. Son dos cosas distintas: el
**contenedor** de la app, que creás una vez desde el menú y es donde se define
con qué warehouse habla, y el **código**, que te lo escribe Genie Code.

Free Edition permite hasta tres apps por cuenta, y las apaga solas a las
veinticuatro horas. Se prenden de nuevo con un botón.

**Qué hacer:**

1. **Creá la app.** Menú **Compute**, pestaña **Apps**, botón **Create app**.
   Nombre `alertas-portafolio-pmo`, plantilla **Streamlit**. En la sección de
   **Resources**, agregá el SQL warehouse `Serverless Starter Warehouse` con
   permiso de uso, y dejá la clave del recurso en `sql-warehouse`, que es la que
   viene por defecto: el `app.yaml` la va a referenciar con ese nombre exacto.
   Sin ese recurso la app no tiene con qué consultar.

   Aprovisionar tarda unos tres minutos. No esperes mirando: seguí con el
   punto 2 mientras se crea.

2. **Pedile el código a Genie Code** con
   [PROMPTS.md, sección 4](PROMPTS.md#4-la-app-interna). También acá hay dos
   caminos: el **corto**, un prompt que pide la app entera con las dos pestañas y
   el cache, o el **paso a paso**, que arranca con la versión mínima y le agrega
   lo demás encima. Con una app, desplegar la versión mínima y verla abrir antes
   de pedirle más te ahorra depurar dos problemas a la vez.

3. **Dale permiso a la app sobre los datos.** La app corre con su propia identidad,
   un service principal, que nace sin acceso a nada. En la pantalla de tu app,
   copiá el nombre del service principal. Después, en un notebook, corré esto
   reemplazando el nombre:

```sql
GRANT USE CATALOG ON CATALOG inchcape_workshop TO `<service principal de tu app>`;
GRANT USE SCHEMA ON SCHEMA inchcape_workshop.pmo TO `<service principal de tu app>`;
GRANT SELECT ON SCHEMA inchcape_workshop.pmo TO `<service principal de tu app>`;
```

   Las tres hacen falta: `USE CATALOG` y `USE SCHEMA` la dejan entrar, `SELECT`
   la deja leer. Comprobá con `SHOW GRANTS ON SCHEMA inchcape_workshop.pmo` que
   las tres quedaron aplicadas antes de seguir.

4. **Pegá los tres archivos** en el código de la app, dale **Deploy** y esperá.
   El primer despliegue tarda unos minutos.

**Copiá el prompt entero, con el bloque de Entorno incluido.** Ahí van las
convenciones de Databricks Apps: cómo se despliega, con qué identidad corre la
app y de dónde sale el warehouse. Es el contexto que vos tenés y el asistente
no, y es lo que hace que la app abra a la primera. Guardalo: la próxima app que
pidas reusa ese mismo bloque tal cual.

**Cómo sabés que funcionó:** abrís la URL de la app y ves las tres tarjetas con
los números del Paso 2. Ese es el momento en que la PMO deja de pedir reportes y
empieza a publicarlos.

**Si algo no sale a la primera**, andá a la pestaña **Logs** de la app, copiá el
log entero y pegáselo a Genie Code tal cual, con la instrucción `Corregí esto`.
Es la forma más rápida de resolverlo, y es exactamente lo que haría yo.

## Paso 5. Qué te llevás

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
| Genie responde con columnas que no existen | Le falta contexto. Pegá el bloque 1 de [SKILLS.md](SKILLS.md) en las Instrucciones del agente. |
| Un número no coincide con esta guía | Corré de nuevo el notebook del Paso 0. Los datos son deterministas, así que si difieren es que el setup quedó a medias. |
| `Quota exceeded` o el cómputo no arranca | Se agotó la cuota diaria de la cuenta. Cierra pestañas con consultas corriendo. Si ya se agotó, se restablece al día siguiente. |
| La app queda en `Stopped` | Free Edition apaga las apps a las veinticuatro horas. Entra a **Compute**, **Apps**, y dale **Start**. |
| Genie Code genera código que falla | Pegale el error completo y escribile `Corregí esto`. No lo arregles a mano en el primer intento. |

## Mapa del repositorio

- [HISTORIA.md](HISTORIA.md), el caso de negocio y el modelo de datos. Leelo primero.
- [PROMPTS.md](PROMPTS.md), todos los prompts listos para copiar, en cuatro
  secciones. Arranca con la tabla que dice qué sección le toca a cada paso.
- [SKILLS.md](SKILLS.md), las instrucciones que le tenés que dar al Genie Agent y
  a Genie Code para que trabajen con el vocabulario de Inchcape.
- [notebooks/00_setup_datos.sql](notebooks/00_setup_datos.sql), el generador de datos.

---

Cualquier duda después del taller, escríbanme. Saúl Caravaca, Solution Architect
de Databricks para Colombia y Costa Rica.
