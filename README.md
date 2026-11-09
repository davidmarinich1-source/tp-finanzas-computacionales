# Carteras óptimas según la fortaleza del dólar

Trabajo Práctico Integrador · Finanzas Computacionales 2026 (UADE)

[![Abrir en Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/davidmarinich1-source/tp-finanzas-computacionales/blob/main/TP_HomeBias.ipynb)

## De qué se trata

Armamos carteras de acciones de los 11 sectores de la economía (todo el S&P 500 más ADR de otros países, todo en dólares), optimizadas para **tres perfiles de riesgo** (conservador, moderado y agresivo) y **dos escenarios del dólar** (fuerte y débil), con un tope de 8% por acción y 25% por sector. Las comparamos contra la cartera óptima que no tiene en cuenta el dólar, con las métricas de riesgo del TP: VaR (paramétrico, histórico y Monte Carlo), drawdown máximo, volatilidad rolling, Sharpe, alfa y beta (CAPM), tracking error e information ratio.

**Pregunta:** ¿mirar el dólar mejora la cartera? ¿En qué acciones y sectores conviene invertir según el perfil y el dólar?

## Cómo correrlo (lo normal, y lo que se usa en la presentación)

1. Hacer clic en el botón **"Abrir en Colab"** de arriba (o en Colab: `Archivo → Abrir notebook → GitHub` y buscar este repositorio).
2. Menú `Entorno de ejecución → Ejecutar todas` (o `Ctrl + F9`).
3. Listo. Con `CONFIG["descargar"] = False` (el valor por defecto) el notebook usa los datos guardados en la carpeta `data/` de este repositorio: no depende de Yahoo Finance y corre rápido.

Notas:
- Si Colab estuvo quieto un rato se desconecta y "se olvida" de lo calculado. No pasa nada: volver a hacer `Ejecutar todas`.
- Correr el notebook **no modifica** el repositorio. Para guardar cambios del código: `Archivo → Guardar una copia en GitHub` (avisarle antes al compañero para no pisarse).

## Cómo actualizar los datos (solo cuando haga falta)

Se hace una vez (y opcionalmente antes de la entrega, para tener datos hasta la última fecha).

1. Abrir el notebook en Colab.
2. En la celda de configuración (sección 0), cambiar `"descargar": False` por `"descargar": True`.
3. `Entorno de ejecución → Ejecutar todas`. Tarda varios minutos: baja la lista del S&P 500 de Wikipedia y los precios de ~650 acciones de Yahoo Finance.
4. Al terminar la sección 2, Colab descarga a la computadora un archivo **`datos_tp.zip`**. Descomprimirlo.
5. En GitHub, entrar a la carpeta **`data/`** del repositorio → `Add file → Upload files` → arrastrar **todos** los archivos descomprimidos → `Commit changes`.
6. Volver a poner `"descargar": False` en el notebook. Desde ahora todos usan esos datos.

**Indicadores (tasa a 2 años y M2, de FRED):** se actualizan aparte, en segundos. Poner `"descargar_indicadores": True`, ejecutar, y en la sección 6 Colab baja `indicadores.csv`: subirlo a la carpeta `data/` del repositorio y volver a poner `False`. Si el archivo no está en el repositorio, el notebook los baja solo de FRED.

## Estructura

| Carpeta / archivo | Contenido |
|---|---|
| `TP_HomeBias.ipynb` | El notebook con todo el trabajo |
| `data/` | Datos descargados: precios, universo de acciones, índice del dólar, referencias |
| `resultados/` | Pesos de las carteras: `pesos_estimacion.csv` (estimadas con 2015–2022, para el backtest) y `pesos_hoy.csv` (con todos los datos, para la recomendación de hoy) |
| `folleto/` | (próximamente) Folleto comercial en PDF |
| `CLAUDE.md` | Contexto completo del proyecto y decisiones tomadas |

## Fuentes de datos

- Precios ajustados por dividendos: Yahoo Finance (librería `yfinance`).
- Lista del S&P 500 y sectores GICS: Wikipedia, *List of S&P 500 companies*.
- Fortaleza del dólar: índice DXY (`DX-Y.NYB` en Yahoo Finance).
- Indicadores anticipatorios: FRED (Reserva Federal de St. Louis): tasa del Tesoro a 2 años (`DGS2`) y M2 (`M2SL`).
