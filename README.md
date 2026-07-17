

# EcoLogísticaDB — Data Platform para Analítica de Vuelos y Sostenibilidad

Proyecto de ingeniería de datos end-to-end aplicado a la industria de la aviación comercial: desde el modelado relacional hasta un asistente de IA conversacional sobre la base de datos, analizando ~225,000 vuelos comerciales (dataset del Bureau of Transportation Statistics, DOT, 2018–2020).

## Objetivo

La industria de la aviación comercial ha dejado de ser únicamente un sector de transporte para convertirse en un ecosistema impulsado por datos. Este proyecto transforma grandes volúmenes de datos planos de vuelos en una arquitectura de información estructurada y estratégica, integrando desde la planificación y estándares IATA hasta el análisis de sostenibilidad ambiental — abordando preguntas como: ¿existen rutas sistemáticamente ineficientes?, ¿qué modelos de aeronave optimizan mejor el consumo de combustible?, ¿cómo afectan los retrasos a la huella de carbono total?

## Highlights técnicos

- 🧠 **LLM fine-tuneado propio:** especialicé **Qwen2.5-Coder-14B** con **QLoRA** (Unsloth) para traducir lenguaje natural a SQL sobre el esquema de la base de datos, mejorando la tasa de consultas ejecutables sin error de **55%** (modelo base) a **95%**, con inferencia 100% local vía Ollama (sin exponer datos a APIs externas). El prompt de inferencia se redujo de ~6,000 a menos de 800 tokens mediante una estrategia de prompt-dropout para internalizar el esquema en los pesos del modelo.
- 🔍 **Detección y corrección de errores de datos:** identifiqué que la variable de consumo de combustible (`fuel_burn`) del dataset original no tenía relación física real con la distancia ni el tiempo de vuelo (correlación cercana a cero), y la reconstruí con un estimador físico basado en la metodología de ciclo LTO + crucero (ICAO/EUROCONTROL) — habilitando un modelo predictivo confiable.
- 🤖 **Modelo predictivo de emisiones:** comparé XGBoost, LightGBM y CatBoost mediante validación cruzada, seleccionando **CatBoost** por su mejor desempeño (**R² = 0.98**, error absoluto medio de 1,300 lbs sobre ~183,500 vuelos).
- 🗄️ **Diseño de base de datos:** modelo entidad-relación de **18 entidades**, normalizado hasta **3FN**, implementado en SQL Server con integridad referencial completa (FOREIGN KEY activas).
- ⚙️ **Pipeline automatizado:** ETL en Python (SQLAlchemy + pandas) que limpia, deduplica (226,979 → 225,346 registros) y carga el dataset completo en 3 fases secuenciales respetando el orden de dependencia entre tablas.
- 📊 **25 consultas SQL avanzadas** organizadas en 5 misiones temáticas (funciones de ventana, PIVOT, acumulados YTD/MTD, clasificación de riesgo por ruta) vinculadas a requerimientos funcionales y no funcionales documentados.
- 📈 **Dashboard interactivo en Streamlit:** 5 módulos — dashboard operativo, panel de huella de carbono, simulador predictivo de emisiones, repositorio de consultas SQL y asistente de IA conversacional — con gráficos dinámicos en Plotly.

## Arquitectura de datos

El modelo relacional integra 18 entidades organizadas en 6 grandes grupos: identificación temporal, información de aerolíneas, rutas/aeropuertos, tiempos y retrasos, estado e incidentes, y aeronave/emisiones ambientales. El dataset fuente contiene 226,979 registros de vuelos (2018–2020) con 66 variables, cubriendo 28 aerolíneas, 379–380 aeropuertos, 153 modelos de aeronave y 5 continentes.

El proceso de normalización siguió las tres formas normales (1NF, 2NF, 3FN), eliminando grupos repetitivos (pares origen/destino en tablas geográficas), separando atributos por dependencia (extracción de la entidad `TAXI`) y aislando dependencias transitivas (creación de `MODELO_DE_AVION` separado de `AERONAVE`).

## Módulos de la aplicación (Streamlit)

1. **Dashboard Operativo:** KPIs en tiempo real (total de vuelos, cancelaciones, retraso promedio) con filtros dinámicos por mes y aerolínea.
2. **Panel EcoLogístico:** emisiones totales de CO2 (10,565,417,789 lbs), huella de carbono por modelo de aeronave y matriz de eficiencia consumo vs. distancia.
3. **Simulador Predictivo (CatBoost):** estimación de combustible y CO2 de un vuelo antes del despegue a partir de modelo de aeronave, distancia, tiempo estimado y mes.
4. **Misiones de Análisis:** las 25 consultas SQL avanzadas integradas como reportes interactivos con tabla, gráfico y código SQL desplegable.
5. **Asistente IA:** consultas en lenguaje natural sobre la base de datos, traducidas a T-SQL y ejecutadas en tiempo real, con inferencia 100% local.

## Conclusiones y hallazgos clave

- La validez física de los datos precede a la sofisticación analítica: una variable sin relación física invalida cualquier indicador agregado, sin importar qué tan correcto sea el modelo dimensional.
- Las uniones sobre claves no únicas son un riesgo silencioso que genera cruces muchos-a-muchos y corrompe los hechos; siempre debe usarse la clave natural completa.
- Validar el conjunto de entrenamiento de un LLM contra el esquema vivo de la base de datos es tan decisivo para la calidad final como el propio fine-tuning.
- La caída de vuelos en Q1-2020 (>40% en algunas aerolíneas) es claramente identificable en los datos, consistente con el inicio de la pandemia de COVID-19.

## Stack

SQL Server · Python (pandas, SQLAlchemy, scikit-learn, CatBoost) · Streamlit · Plotly · Ollama · QLoRA (Unsloth) · Qwen2.5-Coder-14B

