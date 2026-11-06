# proyecto-final-herramientas-computacionales-
# Diagnóstico de la cobertura taxonómica, geográfica y de esfuerzo de recolección de especímenes en Colombia

**Curso:** Herramientas Computacionales 2026-2 · Proyecto final integrador
**Equipo:** Marvin Santiago Londoño Serna, Nicolás Blandób Berrío y Alejandro Muñoz Carmona
**Lenguaje:** R (base + ggplot2)

## 1. Contexto y pregunta

La base de datos "Distribución y recolección de especímenes" reúne 2.836 registros de puntos de recolección de organismos (animales, plantas, hongos y microorganismos) reportados por 13 solicitantes en 29 expedientes (proyectos). Una base de este tipo sirve para decidir dónde y qué se ha recolectado, pero su utilidad depende de qué tan bien identificados están los nombres, de cómo se reparte el esfuerzo en el territorio y de qué tan completo es el muestreo de cada grupo.

El proyecto responde cuatro preguntas con una sola cadena de procesamiento:

1. **Calidad taxonómica:** ¿qué proporción de los registros llega a nivel de especie, y cómo varía por grupo biológico, solicitante y expediente?
2. **Esfuerzo y riqueza espaciales:** ¿dónde se concentra la recolección y dónde se acumula más riqueza registrada?
3. **Completitud del muestreo:** ¿las curvas de acumulación de cada grupo se acercan a estabilizarse?
4. **Perfil por expediente:** ¿cómo difieren los proyectos en composición por grupo y en diversidad (Shannon y Simpson)?

**Alcance:** descriptivo y exploratorio. No se estima la biodiversidad real del país ni se compara entre regiones con criterios estadísticos formales.

## 2. Datos

| Archivo | Descripción |
|---|---|
| `datos/distribucion_recoleccion_especimenes.csv` | Tabla de trabajo (CSV, UTF-8), exportada del Excel original sin modificar su contenido. |
| `datos/distribucion_recoleccion_especimenes_original.xlsx` | Archivo original, solo como respaldo. |

**Fuente:** [completar: entidad que publica la base de datos, enlace y fecha de descarga].

Columnas originales (12): `Expediente`, `Solicitante`, `Proyecto`, `Grupo Biológico`, `Especie`, `Observación`, `Este`, `Norte`, `OBJECTID`, `ID Punto`, `x`, `y`. El script usa `x` (longitud) e `y` (latitud) en grados; no usa `Este` y `Norte`, por lo que no depende de asumir su sistema de referencia.

**Una fila no es un sitio.** Hay 2.836 filas pero solo 636 coordenadas distintas; por ejemplo, un único punto reúne 734 filas de un mismo taxón. Por eso los análisis cuentan **puntos únicos** y pares únicos (punto, taxón), no filas.

## 3. Estructura de la carpeta

```
2026_2-HC_TrabajoFinal-Equipo#/
├── README.md
├── analisis_especimenes.R
├── datos/
│   ├── distribucion_recoleccion_especimenes.csv
│   └── distribucion_recoleccion_especimenes_original.xlsx
└── resultados/            (se crea al ejecutar)
    ├── tablas/
    ├── figuras/
    └── sessionInfo.txt
```

## 4. Requisitos

- R 4.1 o superior.
- Paquete **ggplot2** (obligatorio): `install.packages("ggplot2")`.
- Paquete **maps** (opcional): solo dibuja el contorno de Colombia en los mapas. Sin él, el script funciona igual y avisa con un mensaje.

El resto del código usa únicamente R base.

## 5. Cómo ejecutar

1. Descomprime la carpeta y abre R (o RStudio) con la carpeta del proyecto como directorio de trabajo: `setwd("ruta/a/la/carpeta")`.
2. Ejecuta:

```r
source("analisis_especimenes.R")
```

o, desde la terminal, dentro de la carpeta: `Rscript analisis_especimenes.R`.

El script corre de principio a fin sin intervención, imprime los resultados principales en consola y guarda tablas y figuras en `resultados/`. Para repetir el análisis con otros criterios, cambia los parámetros del bloque **0. Configuración**:

| Parámetro | Valor por defecto | Efecto |
|---|---|---|
| `NIVEL_ANALISIS` | `"especie"` | `"genero"` incluye también los nombres solo a nivel de género, agrupados por género. |
| `PASO_GRADOS` | `0.5` | Tamaño de la celda de la cuadrícula, en grados. |
| `N_PERMUTACIONES` | `200` | Reordenamientos aleatorios de cada curva de acumulación. |
| `MIN_PUNTOS_CURVA` | `10` | Los grupos con menos puntos no se grafican. |
| `MIN_TAXONES_INDICE` | `5` | Por debajo, los índices de diversidad se marcan "con cautela". |
| `SEMILLA` | `2026` | Hace reproducibles las curvas. |

## 6. Qué hace el script

| Sección | Contenido | Salidas |
|---|---|---|
| 1 | Funciones propias: interpretación de nombres, cuadrícula, curvas de acumulación, índices de diversidad. | — |
| 2 | Pruebas con resultados verificables a mano (por ejemplo, Shannon de dos taxones iguales = ln 2). | Mensaje en consola |
| 3 | Lectura, renombrado de columnas, limpieza de texto y validación de coordenadas. | — |
| 4 | **Línea 1.** Normaliza los nombres y clasifica su resolución: especie, género, familia o superior, sin identificar. | Figura 01; tablas 01 a 04 |
| 5 | Construye la tabla de presencias únicas (punto, taxón). | — |
| 6 | **Línea 2.** Cuadrícula de celdas: puntos únicos y taxones por celda; concentración del esfuerzo. | Figuras 02 a 04; tabla 05 |
| 7 | **Línea 3.** Curvas de acumulación por grupo con permutaciones aleatorias. | Figura 05; tablas 06 y 07 |
| 8 | **Línea 4.** Perfil por expediente: composición, Shannon y Simpson. | Figuras 06 y 07; tabla 08 |
| 9 | Guarda las tablas y el registro de la sesión. | `resultados/` |

## 7. Decisiones metodológicas

- **Normalización de nombres.** La columna `Especie` mezcla niveles y formatos: "sp.", "spp.", guiones bajos (`Eunicea_asperula`), mayúsculas inconsistentes, códigos de laboratorio (`ATPH 55 Rhizopus sp.`), autores y saltos de línea. `interpretar_nombre()` los unifica y deja `resultados/tablas/01_normalizacion_nombres.csv` para auditar cada decisión. Los nombres con "cf." o "aff." se conservan marcados como tentativos.
- **Nivel de análisis.** Por defecto se usan solo nombres a nivel de especie, para que "riqueza de especies" signifique lo mismo en todos los grupos. Esto excluye registros: pueden ser muchos en microorganismos y todos los de "Varios Grupos". Esa pérdida es en sí un resultado de la línea 1.
- **Sin abundancias.** La base no registra cuántos individuos hay. Los índices de Shannon y Simpson se calculan con **incidencia** (número de puntos únicos donde aparece cada taxón en el expediente), no con abundancias.
- **Curvas de acumulación.** La unidad es el punto único. Cada curva promedia reordenamientos aleatorios de los puntos; la banda es el rango del 95 % de esos reordenamientos, no un intervalo de confianza poblacional. Los umbrales para interpretar la pendiente final son didácticos.
- **Cuadrícula.** Celdas de 0,5° en grados, sin reproyectar: su área varía levemente con la latitud, aceptable para una exploración descriptiva.

## 8. Limitaciones

- Los registros reflejan dónde se solicitaron y reportaron recolecciones, no un muestreo sistemático; el mapa de riqueza refleja sobre todo el esfuerzo de recolección.
- Sin un polígono oficial del territorio no se puede afirmar qué regiones quedan sin cubrir; solo se describe dónde hay y dónde no hay registros.
- Los nombres no se validan contra un catálogo taxonómico: puede haber sinónimos, errores tipográficos o epítetos mal escritos que cuentan como taxones distintos.
- Con pocos puntos (por ejemplo, hongos) las curvas no son confiables; esos grupos se omiten o se señalan.
- Shannon y Simpson basados en incidencia no son comparables con valores calculados con abundancias.

## 9. Verificación

- La sección 2 del script comprueba las funciones con casos conocidos antes de analizar los datos.
- `resultados/tablas/01_normalizacion_nombres.csv` permite revisar a mano la clasificación de cada nombre.
- `resultados/sessionInfo.txt` guarda las versiones de R y de los paquetes usados.

## 10. Uso de herramientas de inteligencia artificial

[Completar con la declaración del equipo, por ejemplo: "Se usó Claude (Anthropic) como apoyo para proponer la estructura del proyecto y generar un borrador del código. El equipo revisó, ejecutó, verificó y puede explicar cada parte". Indicar qué partes se modificaron o escribieron por el equipo.]

## 11. Referencias

- R Core Team. *R: A Language and Environment for Statistical Computing.* R Foundation for Statistical Computing. https://www.R-project.org/
- Wickham, H. (2016). *ggplot2: Elegant Graphics for Data Analysis.* Springer-Verlag New York.
- Paquete `maps` (opcional), CRAN: https://CRAN.R-project.org/package=maps
- Material del curso Herramientas Computacionales 2026-2: notebooks 03 (funciones), 04 (lectura y análisis exploratorio), 05 y 06 (gráficos con R base y ggplot2) y 07 (malla espacial).
- Base de datos: [completar con la fuente original].
