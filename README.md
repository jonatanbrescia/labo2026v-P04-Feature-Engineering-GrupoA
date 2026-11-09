# labo2026v-P04-Feature-Engineering-GrupoA
# El Caso del Historial Perdido

Experimento de la Maestría en Ciencia de Datos con Inteligencia Artificial de la Universidad Austral. Trabajo de Jonatan y Leonardo.

## Pregunta del experimento

¿Agregar más historial de cada cliente mejora la ganancia de un modelo de predicción de baja (*churn*)? Comparamos un modelo de referencia con cuatro variantes que usan rezagos, tendencias o medias móviles. El objetivo es medir si esas variables aportan valor sobre la historia reciente que ya usa el modelo de referencia.

El experimento se trabajó en **Jupyter sobre Google Cloud**, con **R** y **LightGBM**. La comparación de esta entrega usa la ganancia registrada en **Kaggle para septiembre de 2021**. La variable objetivo del flujo de modelado es `clase_ternaria`.

## Estructura del repositorio

El material fuente se organiza en `Maestría/Labo1/Repo`. Los enlaces siguientes son relativos a la raíz del repositorio y respetan los nombres y las mayúsculas de las carpetas.

| Ubicación | Contenido |
| --- | --- |
| [`Notebooks/`](Notebooks/) | Cinco carpetas experimentales. Cada una contiene un notebook de modelado y un lanzador por lotes. |
| [`Kaggle/resultados_kaggle_todos_los_cortes.csv`](Kaggle/resultados_kaggle_todos_los_cortes.csv) | 350 resultados únicos con experimento, semilla, corte y ganancia. |
| [`Kaggle/maximos_por_experimento_y_semilla.csv`](Kaggle/maximos_por_experimento_y_semilla.csv) | 50 máximos con el corte y la ganancia de cada experimento y semilla. |
| [`Resultados/Test_Wilcoxon.ipynb`](Resultados/Test_Wilcoxon.ipynb) | Notebook de R con los máximos cargados en el código y el cálculo de Wilcoxon pareado exacto. |
| [`Resultados/resumen_wilcoxon.csv`](Resultados/resumen_wilcoxon.csv) | Medias de los máximos, diferencias medianas, recuento de mejoras y valores p de las pruebas individuales. |

## Diseño experimental

| Variante y transformación | Serie en la tabla de Kaggle | Notebook de modelado | Lanzador incluido |
| --- | --- | --- | --- |
| **E0 · Baseline.** Rezagos de uno y dos meses y diferencias respecto de ambos rezagos. | 9100 | [Prueba 620](Notebooks/E0%20Baseline/Prueba_620_WorkFlow_01_junior_grupoA%20%281%29.ipynb) | [Lanzador 9000](Notebooks/E0%20Baseline/Lanzador%209000.ipynb) |
| **E1 · Lags 3 y 4.** Añade los rezagos y las diferencias de orden 3 y 4 a los de E0. | 9200 | [Prueba 621](Notebooks/E1%20lags%203%20y%204/Prueba_621_WorkFlow_01_junior_grupoA.ipynb) | [Lanzador 9100](Notebooks/E1%20lags%203%20y%204/Lanzador%209100.ipynb) |
| **E2 · Tendencias.** Añade variaciones relativas a uno, dos y seis meses, pendiente de seis observaciones y segunda diferencia. | 9300 | [Prueba 622](Notebooks/E2%20Tendencias/Prueba_622_WorkFlow_01_junior_grupoA.ipynb) | [Lanzador 9200](Notebooks/E2%20Tendencias/Lanzador%209200.ipynb) |
| **E3 · Promedios móviles.** Usa medias y desviaciones estándar móviles de tres y seis observaciones y un coeficiente de variación de tres observaciones. El bloque de rezagos y diferencias de E0 queda vacío en esta variante. | E3 | [Prueba 623](Notebooks/E3%20Promedios%20moviles/Prueba_623_WorkFlow_01_junior_grupoA.ipynb) | [Lanzador 9300](Notebooks/E3%20Promedios%20moviles/Lanzador%209300.ipynb) |
| **E4 · Todo junto.** Combina rezagos y diferencias de orden 1 a 4 con tendencias, medias móviles e indicadores de estabilidad. | 9400 | [Prueba 624](Notebooks/E4%20Todo%20Junto/Prueba_624_WorkFlow_01_junior_grupoA.ipynb) | [Lanzador 9400](Notebooks/E4%20Todo%20Junto/Lanzador%209400.ipynb) |

Los códigos de la segunda columna identifican las series **en los resultados consolidados**. Los números en los nombres de los lanzadores corresponden a su configuración guardada. Para vincular código y resultados, usar la variante E0–E4 y esta correspondencia. En E1, el código confirma que se agregaron los rezagos **3 y 4**.

### Flujo de modelado

Los notebooks leen el panel mensual, corrigen datos y ajustan variables monetarias por deflación. Después ordenan el historial por `numero_de_cliente` y `foto_mes` y generan las variables de cada variante. Para entrenar LightGBM, agrupan las clases `BAJA+1` y `BAJA+2` como positivas a partir de `clase_ternaria`.

| Etapa | Período configurado |
| --- | --- |
| Entrenamiento para ajustar el modelo | Enero de 2019 a mayo de 2021 |
| Validación para seleccionar la configuración | Julio de 2021 |
| Entrenamiento final | Enero de 2019 a julio de 2021 |
| Predicción para Kaggle | Septiembre de 2021 |

El ajuste usa el área bajo la curva característica operativa del receptor (AUC) en la validación. La comparación final usa la **ganancia de Kaggle**. Los notebooks generan un ranking por probabilidad de baja y marcan con `Predicted = 1` a los primeros clientes de cada corte.

### Cómo se compararon las variantes

1. Se usaron las mismas diez semillas para emparejar las variantes con E0.
2. Para cada semilla se ordenaron los clientes por el puntaje del modelo. Cada corte determina cuántos clientes, a partir del primero del ranking, se marcan para el envío a Kaggle.
3. Los archivos de Kaggle registran siete cortes de **1.800 a 2.400 clientes**, en pasos de 100. Para cada experimento y semilla se toma **la mayor ganancia de los siete cortes**. Si dos cortes empatan en el máximo, la tabla de máximos conserva el menor.
4. Cada comparación usa **diez pares** de máximos. La diferencia se calcula como `variante − E0` para la misma semilla.
5. Se aplica Wilcoxon pareado bilateral. Se reportan los valores p individuales de las cuatro comparaciones contra E0, sin ajuste por comparaciones múltiples.

**Semillas en el orden de la tabla y los gráficos**  
`915851, 884951, 687121, 664211, 616111, 999853, 999863, 999883, 999907, 999917`

## Cómo ejecutar el proyecto en Jupyter

### Reproducir el resumen estadístico

Abrir [`Resultados/Test_Wilcoxon.ipynb`](Resultados/Test_Wilcoxon.ipynb) con el kernel de R y ejecutar la celda de análisis. El notebook lleva incorporados los 50 máximos y usa R base. El archivo [`Kaggle/maximos_por_experimento_y_semilla.csv`](Kaggle/maximos_por_experimento_y_semilla.csv) permite contrastar esos valores con los datos consolidados.

La salida configurada es `resultados/resumen_wilcoxon.csv`, relativa al directorio de trabajo. La carpeta versionada se llama `Resultados/`, con mayúscula inicial. Para escribir allí desde la raíz del repositorio, cambiar la ruta en las líneas de creación de carpeta y exportación. El cálculo usa rangos promedio en empates y enumera las asignaciones de signo. En E4, las diferencias de +0,581 y −0,581 empatan en magnitud a la precisión publicada.

### Volver a entrenar y generar envíos

1. **Entorno y datos.** Usar Jupyter en Google Cloud con kernel de R. Los notebooks de modelado usan `data.table`, `R.utils`, `mice`, `lightgbm` y `yaml`. Para los lanzadores se requiere Bash y `jupyter nbconvert`. Conseguir el archivo autorizado `analistajr_competencia_2026.csv.gz`, que el código busca en `/content/datasets/`. El archivo de datos no está en la carpeta fuente del repositorio. Adaptar esa ruta y el directorio de trabajo `/content/buckets/b1/exp/` a la instancia donde se ejecute.
2. **Revisar los parámetros de corrida.** Los lanzadores pasan las variables de entorno `SEMILLA` y `EXPERIMENTO`. En las copias de E0 a E3, los notebooks de modelado todavía usan valores fijos para ambos parámetros. E4 sí los toma del entorno. Para ejecutar E0 a E3 por lotes, adaptar esas asignaciones con el patrón de E4 y usar identificadores únicos por variante y semilla. Los lanzadores traen cinco semillas activas, mientras que el análisis final usa diez. Verificar el vector antes de lanzar el lote completo.
3. **Ejecutar E0 y luego las variantes.** Abrir cada lanzador desde el directorio del notebook de modelado correspondiente y revisar `notebook_base`, el prefijo y la ruta del registro. En E0, el archivo guardado lleva el sufijo ` (1)` y el lanzador apunta al nombre sin ese sufijo. La celda auxiliar de lectura del registro apunta a `corridas_9400` en los cinco lanzadores. Usar la carpeta `corridas_<prefijo>/` que corresponda.
4. **Revisar la generación y el envío.** Los notebooks crean salidas de entrenamiento, predicciones y los siete archivos de envío por corrida. La sección final llama a `kaggle competitions submit` para `labo-1-virtual-2026-analista-jr`. Si solo se necesitan los archivos locales, desactivar esa llamada antes de ejecutar. Para enviar, configurar previamente el acceso a la competencia en el entorno.
5. **Consolidar y comparar.** Registrar la ganancia de cada envío junto a su experimento, semilla y corte. Verificar los siete cortes por semilla y una fila por combinación. Calcular el máximo por experimento y semilla, y actualizar la tabla de entrada de `Test_Wilcoxon.ipynb` si se reemplazan los resultados históricos.

Mantener las credenciales de Kaggle y los datos de acceso restringido fuera del repositorio.

## Resultados

### Consolidación de resultados

El archivo consolidado reúne cinco experimentos, diez semillas y siete cortes por semilla. Se eliminaron 22 registros idénticos repetidos de E3. La tabla de máximos conserva una fila por experimento y semilla, y coincide con los 50 valores usados para Wilcoxon.

### Ganancia máxima por semilla

Cada celda muestra el mayor valor observado entre los siete cortes. La escala es la misma que en los resultados del experimento.

| Semilla | E0 · Baseline | E1 · Serie 9200 | E2 · Tendencias | E3 · Medias móviles | E4 · Todo junto |
| --- | ---: | ---: | ---: | ---: | ---: |
| 915851 | 83,655 | 81,578 | 88,889 | 79,335 | 84,236 |
| 884951 | 90,301 | 83,406 | 79,667 | 81,495 | 80,249 |
| 687121 | 89,138 | 80,747 | 86,562 | 81,661 | 83,572 |
| 664211 | 86,729 | 83,323 | 80,913 | 81,993 | 81,080 |
| 616111 | 86,147 | 88,722 | 84,652 | 87,310 | 81,412 |
| 999853 | 80,415 | 81,993 | 79,086 | 84,319 | 76,178 |
| 999863 | 89,968 | 80,996 | 80,996 | 87,892 | 75,347 |
| 999883 | 84,319 | 87,809 | 82,575 | 80,664 | 83,738 |
| 999907 | 84,070 | 82,243 | 76,926 | 83,821 | 78,255 |
| 999917 | 90,633 | 84,403 | 87,393 | 87,144 | 78,255 |
| **Media de los máximos** | **86,54** | **83,52** | **82,77** | **83,56** | **80,23** |

### Wilcoxon pareado contra E0

| Variante | Mediana de las diferencias | Semillas que mejoraron | Valor p bilateral |
| --- | ---: | ---: | ---: |
| E1 | −2,74 | 3 de 10 | 0,1055 |
| E2 | −2,91 | 1 de 10 | 0,0273 |
| E3 | −3,57 | 2 de 10 | 0,0488 |
| E4 | −5,61 | 1 de 10 | 0,0059 |

E0 obtuvo el mayor promedio y quedó primero entre las cinco variantes en **6 de las 10 semillas**. En las pruebas individuales al 5 %, se observó una diferencia significativa para E2, E3 y E4. Las diferencias medianas de las tres variantes fueron negativas. E4 presentó la mayor caída mediana frente a E0, de 5,61 puntos en la escala reportada.

La comparación resume el **mejor corte observado** de cada semilla en un mismo período de evaluación. Para estimar el rendimiento de una política de contacto con corte fijo, hay que fijar el corte antes de evaluarla en un período independiente. La ganancia de Kaggle y el costo de cómputo son medidas separadas.
