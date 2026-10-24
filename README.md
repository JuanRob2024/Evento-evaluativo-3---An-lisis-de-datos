# Análisis de Rotación de Personal – Proyecto BI (Equipo 5: Recursos Humanos)

**Institución Universitaria ITM · Ingeniería de Sistemas · Análisis de Datos · 2026-2**
**Docente:** Daniel Alexis Nieto Mora
**Integrantes:** Juan Pablo Robledo Urrego · Andrés Felipe Villa Calderón · María Camila Álvarez Cruz · Duban Guerrero Castro


---

## Objetivo
Identificar los factores asociados con la rotación de empleados y entregar a Recursos Humanos un dashboard e indicadores que permitan priorizar acciones de retención con base en evidencia estadística.

**Preguntas clave**
1. ¿Cuál es la tasa de rotación y cómo cambia por departamento, cargo y nivel?
2. ¿Hacer horas extra aumenta la probabilidad de salida?
3. ¿Los empleados que se van ganan menos?
4. ¿La satisfacción laboral se relaciona con la rotación?
5. ¿En qué momento de la antigüedad se va más gente?

## Dataset
- **IBM HR Analytics Employee Attrition & Performance**: 1.470 empleados y 35 variables (dataset ficticio creado por IBM).
- Fuente: https://www.kaggle.com/datasets/pavansubhasht/ibm-hr-analytics-attrition-dataset
- Archivo original: `data/WA_Fn-UseC_-HR-Employee-Attrition.csv` · Archivo limpio: `data/hr_attrition_clean.csv`

## Estructura del repositorio
```
├── data/
│   ├── WA_Fn-UseC_-HR-Employee-Attrition.csv   # original
│   └── hr_attrition_clean.csv                  # limpio, insumo de Power BI
├── notebooks/
│   ├── 01_limpieza_eda.ipynb                   # fases 1 y 3 + EDA
│   └── 02_hipotesis_kpis.ipynb                 # fases 2, 4 y 6
├── dashboard/
│   └── rotacion_rrhh.pbix                      # dashboard Power BI
├── docs/
│   ├── dashboard_powerbi.md                    # guía y medidas DAX
│ 
├── requirements.txt
└── README.md
```

## Metodología
1. **Comprensión del problema:** contexto, problema de negocio y preguntas clave.
2. **Hipótesis:** 5 hipótesis con H0 y H1, α = 0,05.
3. **Preparación:** validación de calidad (0 nulos, 0 duplicados), eliminación de 3 columnas constantes, traducción de escalas, variables derivadas (rangos de edad, ingreso y antigüedad; índice de satisfacción) y exportación de una tabla plana para BI.
4. **EDA:** tasas por segmento, distribuciones y matriz de correlación.
5. **Pruebas estadísticas:** chi-cuadrado de independencia (con V de Cramér) para variables categóricas y Mann-Whitney U (con r rank-biserial) para las numéricas, elegida después de que la prueba de Shapiro-Wilk rechazara la normalidad. Como validación multivariada se ajustó una regresión logística (AUC = 0,795).
6. **KPIs y dashboard:** 5 KPIs principales en Power BI, con 4 páginas, filtros sincronizados y drill-through.

**Reproducir:**
```bash
pip install -r requirements.txt
# Ejecutar en orden: 01_limpieza_eda.ipynb y luego 02_hipotesis_kpis.ipynb
jupyter notebook notebooks/
```

## Resultados

### Pruebas de hipótesis
| Hipótesis | Prueba | Estadístico | p-valor | Efecto | Decisión |
|---|---|---|---|---|---|
| H1 Horas extra ↔ rotación | Chi² | 87,56 | 8,2e-21 | V = 0,244 | Rechaza H0 |
| H2 Salidos ganan menos | Mann-Whitney | U = 100.620 | 1,5e-14 | r = 0,311 | Rechaza H0 |
| H3 Satisfacción ↔ rotación | Chi² | 17,51 | 5,6e-04 | V = 0,109 | Rechaza H0 |
| H4 Rotación difiere por depto. | Chi² | 10,80 | 4,5e-03 | V = 0,086 | Rechaza H0 |
| H5 Salidos con menos antigüedad | Mann-Whitney | U = 102.582 | 1,5e-13 | r = 0,298 | Rechaza H0 |

### KPIs
| KPI | Valor | Relación |
|---|---|---|
| Tasa de rotación | **16,1 %** (meta de referencia: 12 %) | General |
| Rotación con horas extra / sin horas extra | **30,5 % vs 10,4 %** (≈ 2,9×) | H1 |
| Brecha salarial (mediana activos vs salidos) | **+62,5 %** (USD 5.204 vs 3.202) | H2 |
| % empleados con satisfacción baja | **19,7 %** | H3 |
| Rotación temprana (≤ 2 años) | **29,8 %** | H5 |

### Hallazgos
- Las **horas extra** son el factor de mayor peso: tienen el odds ratio más alto del modelo logístico (≈ 1,9 por desviación estándar).
- Rotación por cuartil de ingreso: Q1 **29 %** vs Q4 **10 %**.
- Cargos críticos: **Sales Representative (39,8 %)**, Laboratory Technician (23,9 %) y Human Resources (23,1 %).
- Perfil de mayor riesgo: 18-25 años (35,8 %), solteros (25,5 %), viajan con frecuencia y llevan 0-2 años en la empresa.

## Conclusiones
1. Las cinco hipótesis se confirman con p < 0,05, así que la rotación no es aleatoria: se concentra en perfiles identificables.
2. **Recomendaciones:**
   - Controlar y compensar las horas extra.
   - Revisar las bandas salariales de los niveles 1 y 2, sobre todo en ventas y laboratorio.
   - Hacer un onboarding y seguimiento intensivo durante los primeros 24 meses.
   - Aplicar una encuesta de clima trimestral con alertas.
   - Usar el modelo logístico como score de riesgo para la retención preventiva.
3. **Limitaciones:**
   - El dataset es ficticio.
   - Al ser un corte transversal, muestra asociación y no causalidad.
   - Las clases están desbalanceadas (16/84).
   - Las encuestas son autorreportadas.
   - No distingue entre salida voluntaria e involuntaria.
