# 🎯 LeyFig Scanner Pro — Metodología de Madurez del Movimiento

## El problema que resuelve

Un scanner que mide **fuerza** (momentum, ruptura, RSI alto, máximos) encuentra acciones que ya lo están haciendo bien — y por definición las encuentra *después* de que empezaron a hacerlo bien. El resultado práctico: señales correctas en su diagnóstico pero **tardías en su ejecución**, con el tramo fácil ya recorrido, el stop lejos del precio y poco camino por delante. Muchas fallan no porque la acción sea mala, sino porque se compró en la última parte del impulso.

LeyFig conserva íntegra la metodología anterior (régimen de mercado, calidad Buffett, fuerza relativa, patrones chartistas, backtest con costos y walk-forward, blackout de earnings) y le añade **dos mecanismos** que atacan exactamente ese punto.

---

## Mecanismo 1 — Índice de Madurez del Movimiento

Cada acción recibe una puntuación de **0 a 100** que responde: *¿cuánto del impulso alcista vigente ya se consumió?*

**Componentes y su peso:**

| Componente | Peso | Qué mide | Temprano | Agotado |
|---|---|---|---|---|
| Recorrido desde el origen (ATRs) | 30% | Cuánto ha corrido en unidades de su propia volatilidad | < 3 ATR | > 10 ATR |
| Extensión sobre la SMA20 (ATRs) | 25% | Cuán estirado va sobre su media corta | < 1 ATR | > 5 ATR |
| Velas desde el origen | 20% | Antigüedad del impulso | < 10 | > 50 |
| Avance hacia el objetivo | 15% | % recorrido del *measured move* de la base previa | < 25% | > 100% |
| RSI (14) | 10% | Termómetro clásico de agotamiento | ≤ 45 | ≥ 78 |

**La clave metodológica está en dónde empieza el impulso.** El movimiento NO nace en el mínimo de la base, sino donde el precio **abandona** la base — definido como la última vela en la que el precio aún estaba en el 20% inferior del tramo. Por eso una acción que llevaba tres meses lateral y rompió hace seis sesiones se clasifica **TEMPRANA**, aunque su mínimo sea de hace dos meses. Sin esta corrección, cualquier base larga se contabilizaría como "movimiento viejo" y el sistema descartaría precisamente las mejores rupturas.

**La etiqueta es relativa al universo**, no un número absoluto arbitrario:

- 🟢 **TEMPRANA** — percentil ≤ 33: el tercio menos desarrollado de los ~550 tickers del día.
- 🟡 **MEDIA** — percentil 34-66.
- 🔴 **TARDÍA** — percentil > 66: movimiento consumido.

Esto hace que el filtro **se autocalibre al mercado de cada día**: en un mercado recién girado al alza casi todo estará temprano en términos absolutos, y en un rally maduro casi todo estará extendido; en ambos casos siempre existe un "primer tercio" real que analizar, y nunca te quedas sin candidatos ni te inundas de señales tardías.

### La quinta capa de validación

A las cuatro capas anteriores (Radar, Análisis, Zonas de Entrada, Teoría de Juegos) se suma ahora:

**5ª capa — Etapa del movimiento**: la señal solo se emite si el ticker está en el **primer tercio de madurez del universo**, con una red de seguridad absoluta que descarta cualquier movimiento con madurez > 85% aunque el mercado entero esté extendido. Las candidatas que pasan todo excepto esta capa aparecen en la **watchlist** con el motivo explícito ("Etapa TARDÍA, madurez 82%") — así ves qué buenas ideas se rechazaron solo por llegar tarde, que es información valiosa por sí misma.

Umbral configurable con la variable de entorno `EARLY_STAGE_MAX` (por defecto 33). Subirlo a 45 da más señales pero algo más maduras; bajarlo a 25 da menos señales y más frescas.

---

## Mecanismo 2 — Detector de Arquetipos de Entrada (el cambio de fondo)

Medir la madurez no basta si la señal sigue naciendo del mismo sitio. El problema de raíz es **qué situación dispara la alerta**: un scanner de momentum señala cuando la acción ya subió — por construcción llega tarde. Bajar el precio de entrada no arregla eso; solo compra más barato un movimiento igual de avanzado.

LeyFig exige ahora que el gráfico esté en **uno de los cuatro puntos donde un movimiento empieza**. Si no está en ninguno, no hay señal, por alto que sea el score:

| Arquetipo | Qué busca | Por qué es temprano |
|---|---|---|
| 🎯 **PRE-RUPTURA** | Base de 30 velas con rango ≤10%, precio pegado al techo, tendencia de fondo alcista | El movimiento **aún no arrancó**: te posicionas antes de la ruptura |
| 🚀 **RUPTURA** | Superó el techo de una base de ≥25 velas hace **≤5 sesiones** | El impulso **acaba de nacer** |
| ↩️ **PULLBACK** | Tendencia alcista intacta (precio > SMA200, SMA50 subiendo) que corrigió 3-18% y **gira al alza** cerca de su SMA50 | El tramo anterior terminó: entras al inicio del **siguiente** |
| 🔄 **GIRO** | RSI tocó < 35 y gira al alza con el precio sobre su SMA200 | La recuperación **está comenzando** |

Los tres últimos son tempranos por construcción; la RUPTURA además debe estar en el primer tercio de madurez del universo. Cualquier movimiento con madurez > 85% queda descartado de plano.

**Validación:** el detector fue probado contra seis escenarios sintéticos. Los dos que reproducen tu problema —tendencia madura en máximos y vertical parabólico— **no generan setup** (y por tanto ninguna señal), mientras que los cuatro arquetipos de arranque se identifican correctamente.

### La entrada se adapta al arquetipo

El retroceso genérico de 1/3 fue reemplazado por una entrada coherente con cada situación, porque pedir un descuento adicional donde el descuento ya ocurrió significa no entrar nunca:

- **PULLBACK y GIRO** → entrada al giro (−0.2 ATR). La corrección ya sucedió; ese *es* el descuento.
- **RUPTURA** → retest del techo de la base roto: la entrada natural del breakout.
- **PRE-RUPTURA** → dentro de la base, antes del disparo.
- Sin setup claro (casos residuales) → retroceso de 1/3 con tope de 2 ATR.

Cada alerta indica su modo de entrada y el porcentaje bajo el precio actual.

## Cómo usarlo en la práctica

**En el Screener** (dashboard y app), el filtro por defecto es **🧭 Cualquier setup temprano** y el orden es **Madurez ascendente**: lo primero que ves son los movimientos menos desarrollados del universo. La columna 🎯 Etapa muestra el porcentaje consumido con su color, y el tooltip añade el percentil y el recorrido pendiente estimado. Puedes filtrar por señal **"🟢 Etapa TEMPRANA"** para aislar el primer tercio, o por **"🔴 Etapa TARDÍA"** para ver qué evitar.

**En la ficha de cada ticker** (app móvil) hay un bloque dedicado con la madurez, el percentil, el recorrido en ATRs, las velas transcurridas, el origen y máximo del tramo, el recorrido pendiente y el nivel exacto del retroceso de 1/3.

**En las alertas**, cada señal llega con su etapa, el porcentaje consumido, el recorrido pendiente estimado y —si aplica— el aviso de que la entrada es por retroceso de 1/3 con orden límite.

### Las combinaciones más potentes

- **Setup PULLBACK o PRE-RUPTURA + régimen BULL + Buffett ≥ 55**: negocio de calidad al inicio de su impulso, con el mercado a favor. Es la configuración que este sistema busca.
- **Etapa TEMPRANA + patrón alcista vigente (doble suelo, taza con asa, triángulo ascendente)**: la geometría confirma que el movimiento apenas arranca.
- **Etapa TEMPRANA + fuerza relativa alta**: el activo ya supera al mercado pero su tramo aún es joven — momentum sin agotamiento.
- **Evita**: etapa TARDÍA aunque el score sea alto. Es exactamente la señal que fallaba antes.

---

## Qué NO cambia (y por qué importa)

El resto de la metodología permanece intacta: el filtro de régimen sigue suspendiendo compras en mercado bajista, el Buffett Quality Score y el margen de seguridad siguen midiendo el negocio, la fuerza relativa sigue rankeando el momentum transversal, los patrones chartistas conservan sus reglas de vigencia, y el backtest sigue reportando P&L neto tras costos con validación walk-forward y benchmark contra el SPY.

La madurez es una **capa adicional de timing**, no un reemplazo del análisis. Comprar temprano un mal negocio en un mercado bajista sigue siendo mala idea — el sistema simplemente ya no te propondrá comprar tarde un buen negocio.

## Advertencia metodológica honesta

El índice de madurez es una heurística construida con criterio técnico, calibrada y validada contra escenarios sintéticos de movimiento (ruptura reciente, tramo medio, tramo agotado, vertical parabólico y retroceso), no un modelo entrenado sobre resultados históricos. Reduce la probabilidad de comprar al final de un impulso, pero **no la elimina**: un movimiento temprano también puede fallar, y algunos movimientos maduros siguen extendiéndose mucho más de lo razonable. El seguimiento de resultados del Historial es el juez final: compara en unas semanas el porcentaje de acierto de LeyFig contra el de AndFig y deja que los datos decidan.

⚠️ Este sistema es educativo e informativo. No es asesoría financiera. Toda inversión implica riesgo de pérdida del capital.
