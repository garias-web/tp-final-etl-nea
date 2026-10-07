# TP Final - Pipeline ETL de exportaciones del NEA

## Qué hace el pipeline
Este proyecto es un proceso ETL (Extracción, Transformación y Carga) construido íntegramente en Python. Su función es conectarse a una API pública para descargar datos crudos, limpiarlos, transformarlos de un formato de tabla ancha a uno tabular, y calcular métricas como la variación interanual y la participación de cada destino. Finalmente, verifica la calidad de los datos y genera un dataset analítico en formato CSV, junto con un resumen en JSON y un registro de historial.

## De dónde salen los datos
Los datos provienen de los registros del INDEC, obtenidos a través de la API de Series de Tiempo del portal datos.gob.ar. Representan las exportaciones en millones de dólares de las provincias del Noreste Argentino (Chaco, Corrientes, Formosa y Misiones), desglosadas por país de destino y rubro comercial, abarcando el período desde 1993 hasta 2024.

## Cómo instalarlo y ejecutarlo
El proyecto utiliza únicamente la biblioteca estándar de Python, por lo que requiere tener instalado Python 3.8 o superior. No es necesario instalar dependencias externas.

Para correr el pipeline completo y descargar los datos de internet, ejecutá en la terminal:

```bash
python src/main.py
```

Para correr el pipeline utilizando los archivos locales ya descargados (modo offline), ejecutá:

```bash
python src/main.py --sin-internet
```

## Hallazgos en los datos
Al analizar el archivo CSV resultante, se observa una clara tendencia histórica donde Misiones y Chaco lideran el volumen de exportaciones de la región. El principal socio comercial indiscutido del NEA es Brasil, ocupando el primer lugar de destino en la mayoría de los años registrados. Un año particular que resalta en los datos es el 2020: allí Corrientes alcanzó el récord histórico absoluto de la región en un solo año, exportando a Brasil por un valor de 399,14 millones de dólares, traccionado casi en su totalidad por el rubro de Combustibles y Energía.

## Estructura del código

### src/transform.py
- **`ancho_a_largo()`**: Convierte los datos originales de un formato de tabla ancha a un formato largo (tabular) fila por fila.
- **`clasificar_region()`**: Asigna a cada país de destino su región correspondiente usando un diccionario predefinido.
- **`calcular_decada()`**: Calcula a qué década pertenece un año específico.
- **`calcular_participacion()`**: Calcula qué porcentaje del total exportado por la provincia representa un destino puntual.
- **`calcular_variacion()`**: Calcula la variación porcentual entre el valor exportado de un año y el año anterior.
- **`agregar_variacion_interanual()`**: Ordena los datos cronológicamente y aplica el cálculo de variación interanual a todas las filas del dataset.
- **`agregar_ranking()`**: Ordena los destinos de mayor a menor monto exportado y marca cuáles son los tres principales de cada año.
- **`construir_indice_rubros()`** y **`unir_con_rubros()`**: Crea un índice de los rubros exportados y los cruza con la tabla principal de destinos.

### src/load.py
- **`chequear_unicidad()`**: Verifica que no existan filas duplicadas en el dataset final revisando la combinación de provincia, año y destino.
- **`chequear_rangos()`**: Controla que los montos exportados no sean negativos y estén dentro de límites lógicos.
- **`construir_resumen()`**: Genera estadísticas descriptivas generales del dataset, como valores mínimos, máximos y promedios.
- **`guardar_resumen()`** y **`escribir_log_corrida()`**: Guarda el resumen generado en un archivo JSON y añade una nueva línea al historial de ejecuciones para mantener un registro.