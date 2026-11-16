# ¿Conviene mirar el dólar para armar una cartera?

Trabajo Práctico Integrador · Finanzas Computacionales 2026 (UADE) · David Marinich y Mateo Ariati

[![Abrir en Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/davidmarinich1-source/tp-finanzas-computacionales/blob/main/TP_HomeBias.ipynb)

## De qué se trata

Armamos una cartera de acciones de los 11 sectores de la economía (todo el S&P 500 más 123 empresas de otros países que cotizan como ADR, todo en dólares), optimizada con la función de utilidad de la Clase 8, con un tope de 8% por acción y 25% por sector y sin ventas en corto. La armamos en tres versiones: sin mirar el dólar, para dólar fuerte y para dólar débil. Cada mes, un pronóstico hecho con la tasa del Tesoro a 2 años y el M2 elige qué versión usar.

**Pregunta:** ¿mejora la cartera si se la arma según el dólar? ¿Qué sectores convienen con dólar fuerte y con dólar débil?

Las carteras se estiman con datos de 2015 a 2022 y se prueban fuera de muestra de 2023 a noviembre de 2026, contra el ACWI (benchmark), el S&P 500, la cartera 1/N y dos carteras recomendadas por el mercado. Medimos volatilidad, VaR (paramétrico, histórico y Monte Carlo), CVaR, caída máxima, volatilidad rolling, Sharpe, alfa y beta (CAPM), tracking error e information ratio, y hacemos stress tests.

La cartera principal es la de riesgo medio (λ = 10, `CONFIG["perfil_principal"]`); las de λ = 50 y λ = 3 se muestran como complemento.

## Cómo ejecutarlo

1. Hacer clic en el botón **"Abrir en Colab"** de arriba (o en Colab: `Archivo → Abrir notebook → GitHub` y buscar este repositorio).
2. Menú `Entorno de ejecución → Ejecutar todas` (o `Ctrl + F9`).
3. Listo: corre de punta a punta en unos 15 segundos. Con los valores por defecto (`descargar = False`, `recalcular = False`) lee los datos de la carpeta `data/` y las carteras ya optimizadas de `resultados/` de este repositorio, así que solo necesita internet y no depende de Yahoo Finance.

Notas:
- Si Colab estuvo quieto un rato se desconecta y "se olvida" de lo calculado. Volver a hacer `Ejecutar todas`.
- Ejecutar el notebook **no modifica** el repositorio.
- También se puede correr en Jupyter local (Python 3 con numpy, pandas, matplotlib, scipy, scikit-learn, requests y yfinance).

## Qué hace cada sección

Cada sección empieza con un texto que explica qué se hace y por qué, y el código está comentado.

| Sección | Qué hace | Qué muestra |
|---|---|---|
| 0. Configuración | Diccionario `CONFIG` con todos los parámetros e importación de librerías | – |
| 1. Universo | Lista del S&P 500 (Wikipedia) y de los ADR, con su sector | Cantidad de acciones |
| 2. Datos de mercado | Clase `DatosMercado`: precios ajustados (Yahoo Finance), filtros de calidad | Acciones descartadas y motivo |
| 3. Resumen del universo | Acciones por sector y origen | Tabla y gráfico |
| 4. Señal del dólar | Clase `SenalDolar`: DXY contra su promedio de 200 días | Gráfico del DXY y días de dólar fuerte/débil |
| 5. Primera mirada | Retorno medio por sector según el dólar | Tabla y gráfico por sector |
| 6. Indicadores | Clase `IndicadoresMacro`: tasa a 2 años y M2 de FRED | Gráficos de los indicadores |
| 7. Pronóstico del dólar | Clase `PronosticoDolar`: índice compuesto, correlaciones, aciertos, sensibilidad y regla de confirmación | Tablas de aciertos y correlaciones |
| 8. Escenarios | Clase `Escenarios`: meses de suba y baja del DXY; beta de cada sector contra el dólar | Retornos y betas por sector y origen |
| 9. Estimación | Clase `Estimador`: μ y Σ por escenario (Σ vectorizada, comprobada contra `np.cov`, y Ledoit-Wolf) | Comparación de Σ |
| 10. Optimización | Clases `Optimizador`, `Cartera` y `ConjuntoCarteras`: SLSQP con topes y limpieza en dos pasadas | Composición de las carteras 2015–2022 |
| 11. Recomendación de hoy | Lectura actual del pronóstico y carteras con todos los datos | Escenario esperado y carteras |
| 12. Backtest y métricas | Clases `CarteraDinamica`, `Backtest`, `Metricas` y `SimuladorMC` | Tabla de métricas (contra el ACWI, el S&P 500, 1/N y las carteras de PPI y Citi), evolución de USD 10.000, volatilidad rolling, aciertos vs. retorno, comparación de los tres niveles de riesgo, VaR y su backtesting |
| 13. Stress test | Clase `StressTest`: crisis históricas, shocks hipotéticos y Monte Carlo estresado | Pérdidas por escenario y VaR a 1 y 21 días |
| 14. Conclusiones | Tabla resumen, recomendación final, limitaciones y mejoras | Cartera recomendada y su composición completa (23 acciones con sector, país y peso) |

## Parámetros que se pueden cambiar

Todo se cambia en el diccionario `CONFIG` de la sección 0; no hay números fijos en el medio del código. Después de cambiar algo, volver a `Ejecutar todas`.

| Parámetro | Valor | Para qué sirve |
|---|---|---|
| `descargar` | `False` | `True` vuelve a bajar precios de Yahoo Finance (varios minutos) |
| `descargar_indicadores` | `False` | `True` vuelve a bajar la tasa y el M2 de FRED |
| `corte` | `"2022-12-31"` | Hasta qué fecha se estiman las carteras; después empieza la prueba |
| `lambdas` | 50 / 10 / 3 | Aversión al riesgo: más alto = cartera más conservadora |
| `perfil_principal` | `"Moderado"` | Cartera con la que se evalúa la idea y se recomienda |
| `tope_accion`, `tope_sector` | 0,08 / 0,25 | Máximo por acción y por sector |
| `ventana_dolar` | 200 | Días del promedio móvil del DXY |
| `ventana_indicadores` | 3 | Meses para medir el cambio de la tasa y el M2 |
| `recalcular` | `False` | `True` vuelve a optimizar las carteras (unos 30 a 60 segundos) |
| `rf`, `nivel_var`, `simulaciones`, `semilla` | 4%, 95%, 10.000, 42 | Tasa libre de riesgo, confianza del VaR, caminos de Monte Carlo y semilla fija |

**Importante:** si se cambia algo que afecta a las carteras (`corte`, `lambdas`, topes o los datos), hay que poner `recalcular = True`; si no, se cargan las carteras guardadas, que se calcularon con los valores originales.

## Cómo actualizar con datos nuevos

1. Poner `descargar = True` y `recalcular = True` y `Ejecutar todas`. Tarda varios minutos: baja la lista del S&P 500 y los precios de ~650 acciones.
2. Al terminar la sección 2, Colab descarga `datos_tp.zip`; en las secciones 10 y 11, los CSV de pesos de las carteras.
3. Descomprimir el zip y subir los archivos a la carpeta `data/` del repositorio (`Add file → Upload files`); subir los CSV de pesos a `resultados/`.
4. Volver a poner `descargar = False` y `recalcular = False`.

La recomendación de la sección 14 se recalcula sola con los datos nuevos.

Los indicadores se actualizan aparte, en segundos: `descargar_indicadores = True`, ejecutar, y subir el `indicadores.csv` que baja la sección 6 a `data/`.

## Estructura del repositorio

| Carpeta / archivo | Contenido |
|---|---|
| `TP_HomeBias.ipynb` | El notebook con todo el trabajo |
| `data/` | Datos descargados: precios, universo de acciones, índice del dólar, referencias e indicadores |
| `resultados/` | Pesos de las carteras: `pesos_estimacion.csv` (estimadas con 2015–2022, para el backtest) y `pesos_hoy.csv` (con todos los datos, para la recomendación) |
| `folleto/` | Folleto comercial en PDF |
| `CLAUDE.md` | Contexto del proyecto y decisiones tomadas |

## Fuentes de datos

- Precios ajustados por dividendos: Yahoo Finance (librería `yfinance`, `auto_adjust=True`).
- Lista del S&P 500 y sectores GICS: Wikipedia, *List of S&P 500 companies*.
- Fortaleza del dólar: índice DXY (`DX-Y.NYB` en Yahoo Finance).
- Indicadores anticipatorios: FRED (Reserva Federal de St. Louis): tasa del Tesoro a 2 años (`DGS2`) y M2 (`M2SL`).
- Carteras recomendadas: PPI (Bloomberg Línea, 06/12/2022) y Citi (El Cronista, 28/12/2022).
