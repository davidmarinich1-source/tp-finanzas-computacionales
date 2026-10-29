# data/

Acá van los datos que genera el notebook con `CONFIG["descargar"] = True` (ver el `README.md` principal):

| Archivo | Contenido |
|---|---|
| `precios.csv.gz` | Precios diarios ajustados de todas las acciones que pasaron el filtro (comprimido) |
| `universo.csv` | Ticker, nombre, sector, origen y país de cada acción |
| `dolar.csv` | Índice del dólar (DXY) diario |
| `referencias.csv` | Precios de SPY y ACWI |
| `descartados.csv` | Acciones que se sacaron y el motivo |
| `info.json` | Fecha de descarga y cantidad de acciones |

**Todavía no hay datos:** hay que generarlos una vez desde Colab y subirlos acá.
