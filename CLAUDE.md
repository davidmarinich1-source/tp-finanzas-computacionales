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
- **Señal del dólar:** índice DXY (`DX-Y.NYB`). Dólar **fuerte** si está por encima de su promedio móvil de 200 días; **débil** si está por debajo. Se usa la señal del día anterior (`shift(1)`) para no mirar el futuro.
- **Escenarios de estimación:** (1) sin considerar el dólar (todos los días), (2) solo días de dólar fuerte, (3) solo días de dólar débil. μ y Σ se estiman con los días de cada escenario; Σ con Ledoit-Wolf (con ~600 acciones la Σ muestral tiene mucho ruido).
- **Perfiles:** Conservador, Moderado y Agresivo = función de utilidad de la Clase 8 con λ distinto (`CONFIG["lambdas"]`, a calibrar para que las volatilidades queden razonables).
- **Restricciones:** sin ventas en corto, máximo 8% por acción, máximo 25% por sector, suma = 1. Después de optimizar, los pesos menores a 1% se eliminan y se reescala.
- **Carteras:**
  - 9 optimizadas: 3 perfiles × 3 escenarios (sin dólar / fuerte / débil). Las "sin dólar" son la comparación principal.
  - 3 dinámicas (una por perfil): miran la señal y usan la cartera de dólar fuerte o débil (frecuencia de revisión a definir: diaria o mensual, según cuántas veces cambie la señal en la sección 4).
  - Referencias: **ACWI** (benchmark principal, acciones globales, se parece al universo), SPY (referencia), 1/N, y una cartera recomendada (pendiente: idealmente publicada por un ALyC o un diario a fines de 2022 / principios de 2023).
- **Métricas** (para todas las carteras): volatilidad, VaR paramétrico/histórico/Monte Carlo, CVaR, drawdown máximo, volatilidad rolling, Sharpe, alfa y beta (CAPM contra ACWI), tracking error e information ratio contra ACWI, Sharpe separado por escenario del dólar.
- **Stress tests:** Covid (feb–mar 2020), 2022 (dólar muy fuerte), escenario hipotético "dólar +10%" (beta de cada acción contra el DXY), Monte Carlo con la Σ de cada escenario.

## Que funcione el día de la presentación

- Un solo notebook: `TP_HomeBias.ipynb`. "Ejecutar todas" con `CONFIG["descargar"] = False` **tiene que correr de punta a punta sin errores** en un Colab nuevo. Verificarlo después de cada cambio.
- Los datos se descargan una vez y se guardan en `data/`. El notebook los lee desde GitHub (repo público, `raw.githubusercontent.com`), así no depende de Yahoo en vivo.
- `CONFIG["recalcular"] = False` (cuando exista la optimización) usa carteras ya calculadas y guardadas en `resultados/`, para que la demo sea rápida.
- Semilla fija (`CONFIG["semilla"] = 42`) para que Monte Carlo dé siempre los mismos números que el folleto.
- No cambiar `data/` sin regenerar todo y avisar al compañero.

## Estructura del repositorio

```
TP_HomeBias.ipynb   notebook único del trabajo (se abre en Colab)
data/               datos descargados (precios, universo, dólar, referencias)
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
- [ ] **Generar los datos reales** (correr con `descargar = True` en Colab y subir `data/`): pendiente de los integrantes.
- [ ] Lun 02/11: sección 6 y 7 (μ y Σ por escenario, optimizador, 12 carteras).
- [ ] Mar 03/11: sección 8 (métricas).
- [ ] Mié 04/11: sección 9 (backtest, CAPM, tracking error, information ratio).
- [ ] Lun 09/11: sección 10 (stress tests) y conclusiones.
- [ ] Mar 10/11: folleto en PDF.
- [ ] Mié 11/11: presentación, ensayo y simulacro de preguntas.
- [ ] Lun 16/11: presentación.

## Decisiones abiertas

- Cartera recomendada para el backtest: falta definir cuál.
- Valores de λ de cada perfil: calibrar cuando estén las carteras.
