# CLAUDE.md: contexto del proyecto

Trabajo Práctico Integrador (segundo parcial) de **Finanzas Computacionales 2026** (UADE, prof. Diego Onna, curso de los lunes a la noche).
Integrantes: David Marinich y Mateo (compañero). Presentación y entrega: **lunes 16/11/2026** (8 a 15 minutos de exposición + preguntas sobre el código).

Este archivo es para que cualquier sesión de Claude (la de David o la de su compañero) tenga el contexto completo. **Mantenerlo actualizado** cuando se tome una decisión nueva.

## Enunciado acordado con el profesor

> Vamos a armar carteras de acciones de los 11 sectores de la economía, optimizadas para tres perfiles de riesgo y dos escenarios del dólar (fuerte y débil), con un tope de 8% por acción y 25% por sector. Las vamos a comparar contra la cartera óptima que no tiene en cuenta el dólar, con las métricas de riesgo del TP. El objetivo es ver si mirar el dólar mejora la cartera, y en qué acciones y sectores conviene invertir según el perfil y el dólar.

Pedidos del profesor a lo largo de las consultas:
- No limitarse en la cantidad de acciones.
- Medir todo en una moneda homogénea: **todo en dólares**, las acciones extranjeras como **ADR**, sin conversiones de tipo de cambio.
- Usar varias medidas de riesgo, incluyendo **tracking error** e **information ratio**, y fundamentar bien el benchmark.
- Armar carteras para distintas aversiones al riesgo.
- Considerar ciclos / escenarios macro (por eso la señal del dólar).

## Lo que exige la consigna (no negociable)

1. Código en Python con **POO** y **operaciones vectorizadas**, ejecutable sin errores, bien comentado. **Cada integrante tiene que poder explicar todo el código**: si no, el TP se desaprueba.
2. Datos de proveedores financieros; justificar la selección de datos.
3. Matriz de varianzas y covarianzas calculada de forma vectorizada.
4. Carteras optimizadas y no optimizadas.
5. VaR **paramétrico, no paramétrico (histórico) y por Monte Carlo**.
6. **Backtesting** contra un benchmark de mercado y carteras recomendadas, con **Maximum Drawdown** y **Rolling Volatility**.
7. **Alfa y beta (CAPM)**.
8. Stress testing y simulación de escenarios.
9. Folleto comercial en PDF + presentación + instrucciones de ejecución.
10. Puntos extra: restricciones realistas (topes de concentración, sin ventas en corto), originalidad.

## Reglas para escribir código

- **Usar solo técnicas vistas en clase** (lista abajo). Si algo no se vio, preferir la versión más simple y estándar, y explicarla en un comentario.
- **No usar**: Sortino, Calmar ni otras métricas no vistas (salvo tracking error e information ratio, que pidió el profe). No usar librerías de optimización de carteras (PyPortfolioOpt, riskfolio, cvxpy, etc.): la optimización se hace con `scipy.optimize.minimize`, como en la Clase 8.
- Comentarios y textos en **castellano** (rioplatense, como el resto del notebook). Nombres de variables en castellano.
- Código simple y explicable antes que "elegante". Cada celda de código va precedida de una celda de texto que explica qué hace y por qué.
- Todo parámetro va en el diccionario `CONFIG` de la sección 0. Nada de fechas o números "mágicos" en el medio del código.
- Compatible con el pandas y numpy de Google Colab.

### Técnicas vistas en clase (las clases están en la carpeta de David, no en el repo)
- **Clase 3:** funciones, parámetros, `return`, alcance, manejo de errores, módulos.
- **Clase 4:** POO: clases, `__init__`, atributos, métodos, composición, herencia, polimorfismo, buenas prácticas.
- **Clase 5:** NumPy, vectorización, broadcasting, `cumsum`, máscaras, comparación de estrategias (Buy & Hold, DCA, comprar en la caída), **caída máxima (drawdown)**.
- **Clase 6:** Pandas: `read_csv`, limpieza (fechas, faltantes, duplicados), retornos simples y logarítmicos, **`rolling()`**, volatilidad rolling, Matplotlib.
- **Clase 7:** `yfinance` con `auto_adjust=True`, retornos simples (entre activos) vs. log (en el tiempo), media ×252 y volatilidad ×√252, histogramas, **Jarque-Bera**, dividendos y retención, inflación (Fisher), media aritmética vs. geométrica, `linprog`, lambdas, sensibilidad.
- **Clase 8:** covarianza, matriz Σ, `w @ Sigma @ w`, `minimize` con **SLSQP**, `LinearConstraint` (lb ≤ A·w ≤ ub), `Bounds`, separar igualdades y desigualdades, frontera eficiente, **Sharpe** (rf = 4%), máximo Sharpe, **utilidad −wᵀμ + λ·wᵀΣw**, penalización γ, `args` / closures / `partial`, **gradiente analítico `jac=`** y `check_grad`, grilla (λ, γ), fragilidad del óptimo, in-sample vs. out-of-sample, simulación con **Cholesky**, **Ledoit-Wolf** (`sklearn.covariance.LedoitWolf`), `trust-constr` para problemas grandes.
- **Clases sin PDF todavía** (VaR 05/10, Monte Carlo 19/10, CAPM 26/10): usar versiones estándar simples. VaR paramétrico = z·σ (normal); VaR histórico = percentil de los retornos; VaR Monte Carlo = simulación con Cholesky; CVaR = promedio de las pérdidas peores que el VaR; beta = cov(r_p, r_m) / var(r_m); alfa de Jensen.

## Diseño del análisis

- **Universo:** todas las acciones actuales del S&P 500 (lista y sector GICS desde Wikipedia) + ~130 ADR de otros países de los 11 sectores (lista `ADRS` en el notebook). Se descartan las que no tienen precios desde enero de 2015, las que dejaron de cotizar y las que tienen más de 2% de días sin precio. Sesgo de supervivencia: mencionarlo como limitación.
- **Período:** desde 2015. **Estimación hasta el 31/12/2022** (`CONFIG["corte"]`), **prueba fuera de muestra desde 2023**.
- **Enfoque (definido el 09/11 con el profesor):** usar indicadores **anticipatorios** para pronosticar cada mes si el dólar (DXY) se va a fortalecer o debilitar el mes siguiente, y elegir la cartera óptima para ese escenario. No quedarse con un solo indicador, salvo que se demuestre que uno no suma nada.
- **Indicadores (sección 6, clase `IndicadoresMacro`, FRED con `pd.read_csv(url)` como en la Clase 7):** tasa del Tesoro a 2 años (`DGS2`) y M2 (`M2SL`). Todo mensual (fin de mes). El M2 se usa con 1 mes de atraso (`CONFIG["atrasos"]`) porque se publica unas 4 semanas después. **Descartados por decisión del grupo/profesor:** VIX y balance de la Fed.
- **Índice compuesto (sección 7, clase `PronosticoDolar`):** cambio de cada indicador en los últimos 3 meses (`CONFIG["ventana_indicadores"]`), estandarizado `(x − media) / desvío` con estadísticas de 2015–2022, con signo de la teoría (tasa +, M2 −) y promediado. Positivo = dólar fuerte el mes siguiente; negativo = débil. Base 1 solo para graficar DXY y M2 (no para la tasa: cerca de 0% distorsiona).
- **Validación del pronóstico (2015–2022):** correlación con el cambio del DXY a 1, 3 y 6 meses; % de aciertos de tasa sola, M2 solo, índice combinado, tendencia del propio DXY y azar (50%). Sensibilidad de la ventana (1, 3, 6 meses). Elegir todo solo con 2015–2022.
- **Regla de confirmación:** si el índice y la tendencia del DXY (señal de 200 días, `SenalDolar`, sección 4) coinciden, cartera de ese escenario; si no, cartera "sin señal".
- **Escenarios de estimación:** (1) sin considerar el dólar (todos los meses), (2) meses en que el DXY subió, (3) meses en que bajó. μ y Σ se estiman con los días de esos meses; Σ con Ledoit-Wolf (con ~600 acciones la Σ muestral tiene mucho ruido).
- **Recomendación de hoy:** leer el índice con los últimos datos → escenario esperado para el mes siguiente → cartera recomendada por perfil (va al folleto). No se hace proyección a 12 meses con Monte Carlo (decisión del grupo); Monte Carlo se usa solo para el VaR.
- **Perfiles:** Conservador, Moderado y Agresivo = función de utilidad de la Clase 8 con λ = 50, 10 y 3 (`CONFIG["lambdas"]`). En una prueba con datos reales dieron volatilidades esperadas de ~11%, ~14% y ~20%; SLSQP con 580 acciones tarda 2–9 s por cartera.
- **Restricciones:** sin ventas en corto, máximo 8% por acción, máximo 25% por sector, suma = 1. Después de optimizar, los pesos menores a 1% se eliminan y se reescala.
- **Carteras:**
  - 9 optimizadas: 3 perfiles × 3 escenarios (sin dólar / fuerte / débil). Las "sin dólar" son la comparación principal.
  - Estrategias del backtest (por perfil, revisión **mensual**): sin señal, solo tendencia del DXY (reactiva), índice combinado, índice + confirmación, y "oráculo" (sabiendo qué hizo el dólar; techo teórico, no implementable).
  - Referencias: **ACWI** (benchmark principal, acciones globales, se parece al universo), SPY (referencia), 1/N, y una cartera recomendada (pendiente: idealmente publicada por un ALyC o un diario a fines de 2022 / principios de 2023).
- **Métricas** (para todas las carteras): volatilidad, VaR paramétrico/histórico/Monte Carlo, CVaR, drawdown máximo, volatilidad rolling, Sharpe, alfa y beta (CAPM contra ACWI), tracking error e information ratio contra ACWI, Sharpe separado por escenario del dólar.
- **Métrica propia del pronóstico:** % de aciertos fuera de muestra (2023–2026).
- **Stress tests:** Covid (feb–mar 2020) y 2022 (dólar muy fuerte).

## Que funcione el día de la presentación

- Un solo notebook: `TP_HomeBias.ipynb`. "Ejecutar todas" con `CONFIG["descargar"] = False` **tiene que correr de punta a punta sin errores** en un Colab nuevo. Verificarlo después de cada cambio.
- Los datos se descargan una vez y se guardan en `data/`. El notebook los lee desde GitHub (repo público, `raw.githubusercontent.com`), así no depende de Yahoo en vivo.
- `CONFIG["recalcular"] = False` (cuando exista la optimización) usa carteras ya calculadas y guardadas en `resultados/`, para que la demo sea rápida.
- Semilla fija (`CONFIG["semilla"] = 42`) para que Monte Carlo dé siempre los mismos números que el folleto.
- No cambiar `data/` sin regenerar todo y avisar al compañero.

## Estructura del repositorio

```
TP_HomeBias.ipynb   notebook único del trabajo (se abre en Colab)
data/               datos descargados (precios, universo, dólar, referencias, indicadores)
resultados/         (próximamente) pesos de las carteras y tablas de métricas
folleto/            (próximamente) folleto en PDF y gráficos
README.md           cómo correrlo y cómo actualizar los datos
CLAUDE.md           este archivo
```

## Flujo de trabajo entre los dos

- Antes de empezar, traer la última versión del repo. No editar el notebook los dos al mismo tiempo: avisarse quién lo toma.
- Los cambios se guardan en GitHub con un mensaje claro de qué se cambió.

## Estado y cronograma (trabajan lunes, martes y miércoles)

- [x] Repo, `CLAUDE.md`, `README.md`.
- [x] Secciones 0 a 5 del notebook: configuración, universo, `DatosMercado`, resumen, `SenalDolar`, primera mirada por sector.
- [x] Datos reales generados el 02/11/2026 y subidos a `data/`: 580 acciones (457 S&P 500 + 123 ADR), 02/01/2015 a 02/11/2026, 50 descartadas.
- [x] Lun 09/11: configuración de indicadores y sección 6 (`IndicadoresMacro`: descarga de FRED de tasa a 2 años y M2).
- [x] Indicadores reales bajados de FRED el 09/11/2026 y subidos a `data/indicadores.csv` (01/2013 a 11/2026; el M2 llega hasta 09/2026 porque se publica con atraso).
- [x] Lun 09/11: sección 7, `PronosticoDolar` (índice compuesto, correlaciones a 1/3/6 meses, aciertos, sensibilidad de la ventana, regla de confirmación). Parámetros nuevos en `CONFIG`: `medicion`, `signos`, `horizontes`, `ventanas_sensibilidad`.
- [ ] Secciones 8 a 11: escenarios, μ y Σ, optimizador y carteras, recomendación de hoy.
- [ ] Secciones 12 a 14: backtest y métricas, stress test, conclusiones.
- [ ] Mar 10/11 y Mié 11/11: folleto en PDF, presentación, ensayo y simulacro de preguntas.
- [ ] Lun 16/11: presentación y entrega.

## Hallazgos hasta ahora (datos reales)

- Período de estimación (2015–2022): 1.282 días de dólar fuerte y 732 de dólar débil.
- Retorno medio anual con dólar fuerte vs. débil: EE.UU. 12,7% vs. 21,4% (−8,6 pp); ADR 6,1% vs. 21,7% (−15,7 pp). Las acciones de afuera son más sensibles al dólar.
- Sectores más sensibles: materiales (−19,9 pp), comunicación (−16,2), tecnología y financiero (−14,9). Menos sensibles: consumo básico (0) y servicios públicos (+1,8).

### Resultados del pronóstico (sección 7, datos reales)

- Correlación con el DXY de los meses siguientes (2015–2022): tasa a 2 años +0,07 (1 mes), +0,26 (3), +0,48 (6); M2 +0,19 / +0,20 / +0,22; índice combinado +0,16 / +0,29 / +0,45. Signos según la teoría; la señal es más fuerte a 3–6 meses que a 1 mes (ojo: ventanas superpuestas inflan las correlaciones largas).
- Aciertos a 1 mes en 2015–2022: tasa 56%, M2 55%, índice 54%, tendencia DXY 53%, decir siempre "fuerte" 56%. En 2023 en adelante: índice 56%, tendencia 56%, M2 47%, siempre "fuerte" 47%.
- Ventana: 1 mes 55% / 3 meses 54% / 6 meses 49% (estimación). Se mantiene 3.
- Confirmación: acierta 55% (estimación) y 58% (prueba) en los meses en que toma posición.
- Lectura: el poder de anticipación a 1 mes es débil (cerca de "siempre fuerte"); mejor a 3–6 meses. Es una conclusión válida para el TP. El valor real se mide en el backtest de carteras.
- Lectura más reciente (fin de 10/2026 y 11/2026 parcial): índice positivo, tendencia fuerte → decisión "fuerte".

### Indicadores probados y descartados (09/11)

- Se probaron 5 candidatos: tasa a 2 años, M2, diferencial EE.UU. − Europa (`DFF` − `ECBDFR`), tasa real a 10 años (`DFII10`) y curva 10a − 2a (`T10Y2Y`), eligiendo los 3 con mayor correlación con el DXY del mes siguiente (2015–2022). Quedaron M2, diferencial y tasa a 2 años; la tasa real se descartó por ir en contra de la teoría a 1 mes.
- El índice de 3 subió la correlación (+0,18 a 1 mes, +0,32 a 3) pero bajó los aciertos de dirección (49% en 2015–2022, 53% en la prueba, contra 54% y 56% con tasa + M2). **El grupo decidió volver a tasa a 2 años + M2.** La versión con 5 candidatos queda en el historial de GitHub (commits `c00d604` y `6c3c0f3`) por si se quiere recuperar.
- La tendencia del DXY (200 días) se deja afuera del índice a propósito: el índice mide causas (anticipatorias) y la tendencia es el propio dólar (reactiva); se usa como confirmación y como referencia en la tabla de aciertos.

## Decisiones abiertas

- Cartera recomendada para el backtest: falta definir cuál.
- Repaso del código con David: llegaron hasta la parte 4c (descarga en `DatosMercado`); sigue la 4d (filtro).
