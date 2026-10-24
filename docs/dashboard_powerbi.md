# Guía de construcción del dashboard – Power BI

**Fuente:** `data/hr_attrition_clean.csv` (lo genera el notebook). Tabla: `RRHH`.

## 1. Carga
1. Power BI Desktop → *Obtener datos* → *Texto/CSV* → `hr_attrition_clean.csv` → **Transformar datos**.
2. En Power Query revisa los tipos: `Rotacion`, `MonthlyIncome`, `YearsAtCompany` e `IndiceSatisfaccion` deben quedar como **Número**; las columnas `_Txt` y `Rango*`, como **Texto**.
3. Cambia el nombre de la consulta a `RRHH` → *Cerrar y aplicar*.
4. Orden de los rangos: selecciona `RangoAntiguedad` → *Herramientas de columnas* → *Ordenar por columna*. Para eso crea antes una columna de orden, por ejemplo:
   ```DAX
   OrdenAntig = SWITCH(RRHH[RangoAntiguedad],"0-2 años",1,"3-5 años",2,"6-10 años",3,4)
   ```
   Haz lo mismo con `JobSatisfaction_Txt`, ordenándola por `JobSatisfaction`.

## 2. Medidas DAX (crear una tabla vacía `_Medidas` para agruparlas)
```DAX
Empleados = COUNTROWS(RRHH)
Salidas = SUM(RRHH[Rotacion])
Activos = [Empleados] - [Salidas]

-- KPI 1
Tasa Rotacion = DIVIDE([Salidas], [Empleados])

-- KPI 2
Rotacion Horas Extra = CALCULATE([Tasa Rotacion], RRHH[OverTime] = "Yes")
Rotacion Sin Horas Extra = CALCULATE([Tasa Rotacion], RRHH[OverTime] = "No")
Multiplicador Horas Extra = DIVIDE([Rotacion Horas Extra], [Rotacion Sin Horas Extra])

-- KPI 3
Ingreso Mediano Activos = CALCULATE(MEDIAN(RRHH[MonthlyIncome]), RRHH[Rotacion] = 0)
Ingreso Mediano Salidos = CALCULATE(MEDIAN(RRHH[MonthlyIncome]), RRHH[Rotacion] = 1)
Brecha Salarial = DIVIDE([Ingreso Mediano Activos], [Ingreso Mediano Salidos]) - 1

-- KPI 4
Pct Satisfaccion Baja = DIVIDE(CALCULATE([Empleados], RRHH[JobSatisfaction] = 1), [Empleados])

-- KPI 5
Rotacion Temprana = CALCULATE([Tasa Rotacion], RRHH[YearsAtCompany] <= 2)

-- KPI 6 y apoyo
Indice Satisfaccion = AVERAGE(RRHH[IndiceSatisfaccion])
Antiguedad Media = AVERAGE(RRHH[YearsAtCompany])
Meta Rotacion = 0.12
Semaforo Rotacion = IF([Tasa Rotacion] > [Meta Rotacion], "#C0392B", "#27AE60")
```
Valores esperados sin filtros: Tasa 16,1 % · Horas extra 30,5 % vs 10,4 % · Brecha 62,5 % · Satisfacción baja 19,7 % · Rotación temprana 29,8 % · Índice 2,73.

## 3. Páginas

### Página 1 – Dashboard ejecutivo (obligatoria)
| Visual | Campos | Propósito |
|---|---|---|
| 5 **tarjetas KPI** | Tasa Rotacion, Rotacion Horas Extra, Brecha Salarial, Pct Satisfaccion Baja, Rotacion Temprana | Resumen en 5 segundos |
| **Medidor (gauge)** | Valor: Tasa Rotacion · Destino: Meta Rotacion | Distancia a la meta del 12 % |
| **Barras horizontales** | Eje: Department_Txt · Valor: Tasa Rotacion | Dónde está el problema |
| **Columnas agrupadas** | Eje: OverTime_Txt · Valor: Tasa Rotacion | Hallazgo principal (H1) |
| **Anillo (donut)** | Leyenda: Attrition_Txt · Valor: Empleados | Proporción de activos y salidos |
| Cuadro de texto | 3 insights clave | Narrativa |

### Página 2 – Perfil de rotación
- **Barras** de Tasa Rotacion por `JobRole`, ordenadas de mayor a menor (Sales Representative ≈ 40 %).
- **Matriz/heatmap**: filas `Department_Txt`, columnas `OverTime_Txt`, valor Tasa Rotacion con formato condicional de escala de color.
- **Columnas** de Tasa Rotacion por `RangoEdad` y por `MaritalStatus_Txt`.
- **Treemap**: Grupo `JobRole`, valores `Salidas`.

### Página 3 – Compensación y antigüedad (H2, H5)
- **Diagrama de caja**, si importas el visual *Box and Whisker* desde AppSource, o en su defecto un **gráfico de dispersión** con eje X `YearsAtCompany`, eje Y `MonthlyIncome` y leyenda `Attrition_Txt`.
- **Columnas** de Tasa Rotacion por `RangoIngreso` (Q1 ≈ 29 % vs Q4 ≈ 10 %).
- **Líneas** de Tasa Rotacion por `YearsAtCompany` (pico en los primeros años).

### Página 4 – Clima laboral (H3)
- **Barras apiladas al 100 %**: eje `JobSatisfaction_Txt`, leyenda `Attrition_Txt`.
- **Barras** de Tasa Rotacion por `WorkLifeBalance_Txt` y por `EnvironmentSatisfaction_Txt`.
- **Tabla** con los resultados de las pruebas estadísticas (copiar del notebook): hipótesis, prueba, p-valor y decisión.

## 4. Filtros e interactividad
- **Segmentaciones** (en todas las páginas, sincronizadas con *Vista → Sincronizar segmentaciones*): Department_Txt, Gender_Txt, RangoEdad, OverTime_Txt, JobLevel.
- **Interacción cruzada:** al hacer clic en una barra de departamento se filtran los demás visuales (*Formato → Editar interacciones*).
- **Obtener detalles (drill-through):** crea una página "Detalle empleado" con una tabla (EmployeeNumber, JobRole, MonthlyIncome, YearsAtCompany, OverTime, IndiceSatisfaccion) y agrega `JobRole` como campo de drill-through.
- **Tooltips** personalizados con Empleados, Salidas e Ingreso mediano.
- **Botones de navegación** entre páginas (Insertar → Botones → Navegador de páginas).

## 5. Buenas prácticas de diseño
- Paleta reducida: rojo solo para "Se fue" y riesgo; gris o azul para "Activo". Aplica esto en todas las páginas.
- KPIs arriba a la izquierda (lectura en Z); de lo general a lo particular.
- Títulos que digan el hallazgo, por ejemplo *"Las horas extra triplican la rotación"*, y no solo *"Rotación por OverTime"*.
- Sin efectos 3D, con pocas líneas de cuadrícula y etiquetas de datos en porcentaje con 1 decimal.
- Fondo neutro, tipografía Segoe UI y tamaño de lienzo 16:9.
- Guarda el archivo como `dashboard/rotacion_rrhh.pbix`. Si lo hacen en Tableau, guárdenlo como `.twbx`.
