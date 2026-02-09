# Interactuando con la API v2 del World Bank

En este proyecto vas a construir un flujo de datos completo: consumo de API, transformacion a DataFrame, analisis visual y carga a base de datos SQL.

## Contexto del ejercicio

Trabajaras con la API publica del World Bank v2 (sin autenticacion). El objetivo es analizar la evolucion socioeconomica y ambiental de 5 paises elegidos por ti entre 2010 y 2024.

### Seleccion del dataset

1. Elige 5 paises (ISO3) que te interesen.
2. Elige los indicadores que quieras analizar.
3. Recomendacion de indicadores (opcional):

- `SP.POP.TOTL`: Poblacion total
- `NY.GDP.PCAP.CD`: PIB per capita (USD actuales)
- `EN.ATM.CO2E.PC`: Emisiones de CO2 per capita (toneladas metricas)
- `SP.DYN.LE00.IN`: Esperanza de vida al nacer (anios)

API base: `https://api.worldbank.org/v2`

## Requisitos tecnicos

Debes trabajar en un archivo `.ipynb` y usar:

- `requests`
- `pandas`
- `matplotlib` y/o `seaborn`
- `sqlalchemy`

## Paso 1: Preparar entorno

Instala dependencias:

```bash
pip install requests pandas matplotlib seaborn sqlalchemy
```

Crea un notebook, por ejemplo: `src/world_bank_analysis.ipynb`.

## Paso 2: Explorar la API

Revisa estos endpoints de referencia:

- Paises: `https://api.worldbank.org/v2/country`
- Indicadores: `https://api.worldbank.org/v2/indicator`

Verifica la estructura de respuesta. La API pagina resultados (habitualmente hasta `per_page=50`), asi que debes pensar una estrategia para recorrer paginas y almacenar toda la informacion necesaria.

Request de ejemplo en Python (plantilla):

```python
import requests

url = "https://api.worldbank.org/v2/country"
params = {
    "format": "json",
    "per_page": 50,
    "page": 1
}

response = requests.get(url, params=params, timeout=30)
response.raise_for_status()
payload = response.json()

# payload[0] -> metadatos de paginacion
# payload[1] -> datos de la pagina actual
print("Metadatos:", payload[0])
print("Primer elemento:", payload[1][0])
```


*IMPORTANTE:* El código mencionado arriba es orientativo. En el link de abajo tienes toda la información necesaria para llevar a cabo un llamado a la API:

https://datahelpdesk.worldbank.org/knowledgebase/articles/898581

## Paso 3: Descargar datos

Descarga series temporales 2010-2024 para los paises e indicadores que elegiste.

Objetivo:

- Consumir la API para varios paises e indicadores
- Manejar paginacion cuando aplique
- Guardar respuestas en una estructura temporal (lista de diccionarios)

## Paso 4: Transformar respuesta a DataFrames

Crea una tabla (DataFrame) por indicador para facilitar comparaciones entre paises.

Columnas sugeridas por tabla:

- `country` 
- `year`
- `value`

Limpieza minima:

- Eliminar filas con `value` nulo cuando sea necesario
- Convertir `year` a entero
- Convertir `value` a numerico

## Paso 5: Analisis y visualizaciones

Genera al menos 2 graficos y explica hallazgos en celdas Markdown.

Ejemplos:

1. Line chart: evolucion de un indicador por pais (2010-2024)
2. Scatter plot: relacion entre dos indicadores para un anio reciente

## Paso 6: Cargar resultados a base de datos SQL

Usa SQLite con SQLAlchemy para persistir datos:

- Base de datos: `world_bank_analysis.db`
- Recomendacion didactica: una tabla por indicador (ejemplo: `indicator_gdp_per_capita`, `indicator_life_expectancy`, etc.)

Flujo recomendado:

1. Crear engine con SQLAlchemy
2. Guardar cada DataFrame con `to_sql(..., if_exists="replace")`
3. Leer una muestra con `pd.read_sql()` para validar carga

## Cierre

¡Ya tienes todo para comenzar!
Tomate tu tiempo para investigar la documentacion de la API y entender bien la estructura de las respuestas.
Si te surge cualquier duda durante el proceso, contacta a tus mentores.
