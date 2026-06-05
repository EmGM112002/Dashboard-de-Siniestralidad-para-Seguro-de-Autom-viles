# Portafolio Actuarial — Proyecto 02
## Dashboard de Siniestralidad para Seguro de Automóviles

![Power BI](https://img.shields.io/badge/Power%20BI-F2C811?style=flat&logo=powerbi&logoColor=black)
![DAX](https://img.shields.io/badge/DAX-2E6DB4?style=flat&logo=microsoft&logoColor=white)
![LaTeX](https://img.shields.io/badge/LaTeX-008080?style=flat&logo=latex&logoColor=white)
![Área](https://img.shields.io/badge/Área-Pricing%20No--Vida-1B3A6B?style=flat)
![Ramo](https://img.shields.io/badge/Ramo-Automóviles-2E6DB4?style=flat)

---

## ¿De qué trata este proyecto?

Dashboard interactivo en Power BI que analiza la siniestralidad de una cartera
de **5,000 pólizas de seguro de automóviles** en México (2022–2024).
El modelo permite identificar segmentos de riesgo, tendencias temporales y la
suficiencia de la tarifa vigente a través de los indicadores clave del ramo:
loss ratio, frecuencia, severidad, prima pura e IBNR estimado.

| Métrica global | Valor |
|---|---|
| Pólizas emitidas | 5,000 |
| Siniestros totales | 690 |
| Siniestros pagados | 687 |
| Prima devengada total | $53.5 M MXN |
| Loss Ratio global | 95% |
| Frecuencia | 0.15 siniestros / vehículo-año |
| Severidad media | $73,610 |
| Prima pura | $10,940 |

---

## Fundamento teórico

### Modelo de riesgo colectivo

Los siniestros de la cartera se modelan como $S = X_1 + X_2 + \cdots + X_N$,
donde $N$ (frecuencia) y $X_i$ (severidad) son independientes.

$$\text{Prima pura} = f \times s = \frac{N}{E} \times \frac{\sum X_j}{N} = \frac{\sum X_j}{E}$$

La prima pura (o *burning cost*) es el costo esperado de siniestros por unidad
de exposición (Werner & Modlin, 2016).

### Loss Ratio

$$\text{Loss Ratio} = \frac{\text{Siniestros pagados}}{\text{Prima devengada}} \times 100\%$$

Rangos de referencia para el ramo de automóviles en México:
- ✅ **< 65%** — Cartera rentable
- ⚠️ **65–80%** — Zona de atención
- 🔴 **> 80%** — Revisión tarifaria requerida

El Loss Ratio global de la cartera es **95%**, lo que la sitúa claramente en zona
de acción requerida.

### IBNR estimado (factor de cola)

$$\widehat{\text{IBNR}} = \frac{\text{Siniestros reportados (últimos 90 días)}}{f_{\text{tail}}} - \text{Siniestros reportados}$$

Con $f_{\text{tail}} = 0.85$ (85% de siniestros reportados a 90 días de ocurrencia).

---

## Estructura del repositorio

```
02-PowerBI-DAX/
│
├── datos/
│   ├── polizas.csv           # 5,000 pólizas con vigencias y primas
│   ├── siniestros.csv        # ~690 siniestros con fechas y montos
│   ├── dim_cobertura.csv     # Catálogo de coberturas
│   └── dim_region.csv        # Catálogo de regiones con factor de riesgo
│
├── medidas_dax.dax           # 24 medidas DAX documentadas
├── tema_actuarial.json       # Tema visual minimalista financiero
├── Proyecto_2.pbix           # Dashboard ejecutable
├── Proyecto_2_PBI.pdf        # Exportación visual de las 4 páginas
├── Proyecto02_PowerBI_DAX_Siniestralidad.pdf   # Documento teórico
└── README.md
```

---

## Modelo de datos (Star Schema)

```
                 DimFecha
                    │
   DimCobertura ── FactSiniestros ── DimRegion
        │                                │
        └────────── FactPolizas ─────────┘
```

| Tabla | Tipo | Registros |
|---|---|---|
| `FactSiniestros` | Hechos | ~690 |
| `FactPolizas` | Hechos | 5,000 |
| `DimFecha` | Dimensión | 1,096 (2022–2024) |
| `DimCobertura` | Dimensión | 4 |
| `DimRegion` | Dimensión | 5 |

### Relaciones (dirección de filtro cruzado: única)

| Desde | Hacia | Estado |
|---|---|---|
| `DimFecha[Date]` | `FactSiniestros[Fecha_Ocurrencia]` | Activa |
| `DimFecha[Date]` | `FactSiniestros[Fecha_Reporte]` | Inactiva |
| `DimFecha[Date]` | `FactPolizas[Fecha_Inicio]` | Activa |
| `DimFecha[Date]` | `FactPolizas[Fecha_Fin]` | Inactiva |
| `DimCobertura[ID_Cobertura]` | `FactSiniestros[ID_Cobertura]` | Activa |
| `DimCobertura[ID_Cobertura]` | `FactPolizas[ID_Cobertura]` | Activa |
| `DimRegion[ID_Region]` | `FactSiniestros[ID_Region]` | Activa |
| `DimRegion[ID_Region]` | `FactPolizas[ID_Region]` | Activa |

> **Nota:** las tablas de hechos no se conectan entre sí. La dirección del filtro
> cruzado debe ser única (no bidireccional) — la dirección bidireccional rompe
> las funciones de inteligencia de tiempo.

---

## Páginas del dashboard (21 visuales)

### Página 1 — Resumen Ejecutivo *(6 visuales)*
Segmentador de año, semáforo de estado de la cartera, 4 tarjetas KPI (Loss Ratio,
Siniestros Pagados, Frecuencia, Severidad), 3 tarjetas secundarias (Prima Pura,
Num Pólizas, Variación vs año anterior) y 2 gráficos de barras del Loss Ratio por
Cobertura y por Región con línea de referencia en 80%. *Para la dirección general.*

### Página 2 — Análisis por Segmento *(6 visuales)*
2 segmentadores (rango de fecha y cobertura), 2 gráficos de barras apiladas
(siniestros por tipo y por región), dispersión Frecuencia × Severidad con tamaño
proporcional al volumen, y tabla resumen por región con formato condicional
semáforo en el Loss Ratio. *Para el equipo de pricing.*

### Página 3 — Tendencias Temporales *(5 visuales)*
3 tarjetas (Loss Ratio YTD 24%, Loss Ratio 12M 24%, Variación AA 34%), gráfico de
líneas con Loss Ratio vs móvil 12 meses, gráfico combinado de siniestros actuales
vs año anterior, columnas de estacionalidad mensual y barras de prima pura
trimestral. *Para el área técnica.*

### Página 4 — Reservas e IBNR *(4 visuales)*
4 tarjetas (IBNR Estimado $148.1k, RBNS $170.5k, Reserva Total $318.7k, Rezago
5.67 días), gráfico de columnas de siniestros en proceso por mes, gráfico de
columnas apiladas RBNS + IBNR por cobertura, y tabla de detalle de siniestros
pendientes filtrada por Estado = "En proceso". *Para el actuario de reservas.*

---

## Medidas DAX principales

El archivo `medidas_dax.dax` contiene **24 medidas** organizadas en 5 secciones:

| Sección | Medidas |
|---|---|
| Base | `Siniestros_Pagados`, `Num_Siniestros`, `Prima_Devengada`, `Exposicion_VA`, `Prima_Emitida` |
| Indicadores | `Loss_Ratio`, `Frecuencia`, `Severidad`, `Prima_Pura` |
| Time Intelligence | `Loss_Ratio_YTD`, `Loss_Ratio_12M`, `Loss_Ratio_AA`, `LR_Variacion_AA`, `Siniestros_AA` |
| Reservas | `IBNR_Estimado`, `RBNS`, `Reserva_Total`, `Rezago_Reporte_Dias`, `Siniestros_Ult90` |
| Auxiliares | `Semaforo_LR`, `Num_Polizas`, `Num_En_Proceso` |

### Prima Devengada (versión implementada)

```dax
Prima_Devengada =
SUMX(
    FactPolizas,
    FactPolizas[Prima_Emitida]
        * DIVIDE(
            DATEDIFF( FactPolizas[Fecha_Inicio], FactPolizas[Fecha_Fin], DAY ),
            365,
            0
        )
)
```

Pondera la prima emitida de cada póliza por su duración real en años, asumiendo
devengamiento uniforme a lo largo de la vigencia.

---

## Cómo reproducir el dashboard

**Requisitos:** Power BI Desktop (descarga gratuita en Microsoft Store)

1. Abrir Power BI Desktop.
2. **Obtener datos → Texto/CSV** → importar los 4 archivos de `/datos/`.
3. En vista de modelo, crear las relaciones según la tabla anterior (todas con dirección de filtro **Única**).
4. **Modelado → Nueva tabla** → pegar la fórmula `DimFecha` del archivo `.dax`.
5. Crear tabla `_Medidas` y agregar cada medida del archivo `medidas_dax.dax`.
6. **Vista → Temas → Buscar temas** → `tema_actuarial.json`.
7. Construir las 4 páginas del dashboard con los visuales descritos.

---

## Patrones de riesgo identificados

### Loss Ratio por cobertura

| Cobertura | Loss Ratio | Interpretación |
|---|---|---|
| Limitada | ~130% | Precio insuficiente — acción inmediata |
| Robo Total | ~110% | Deficitaria — revisión tarifaria |
| Amplia | ~85% | Por encima del umbral de atención |
| Responsabilidad Civil | ~30% | Muy rentable — oportunidad de crecimiento |

### Loss Ratio por región

| Región | Loss Ratio | Interpretación |
|---|---|---|
| Centro | ~110% | Cartera deficitaria |
| CDMX | ~100% | Mayor frecuencia de robo y accidentes |
| Occidente | ~95% | En zona de acción |
| Sur | ~80% | Límite de la zona de atención |
| Norte | ~80% | Segmento más controlado |

> Los valores se leen de los gráficos de barras del dashboard; las cifras exactas
> se obtienen al pasar el cursor sobre cada barra (tooltip con Loss Ratio,
> siniestros y frecuencia).

---

## Decisiones técnicas relevantes

Tres problemas no triviales resueltos durante la implementación:

- **Dirección de filtro cruzado:** las relaciones bidireccionales que Power BI
  crea por defecto rompen `DATESYTD`, `DATESINPERIOD` y `LASTDATE`. Se corrigieron
  a dirección única.
- **Time intelligence robusta:** `Loss_Ratio_YTD` y `Loss_Ratio_12M` filtran
  `FactSiniestros` directamente con `FILTER(ALL(FactSiniestros), ...)` en lugar de
  depender de la propagación a través de `DimFecha`.
- **IBNR con fecha de referencia:** se sustituyó `TODAY()` por `MAX(DimFecha[Date])`
  porque el dataset termina en 2024, y se usó `ALL(FactSiniestros)` para evitar
  conflictos con el filtro de página de la Página 4.

---

## Referencias

- Werner, G., & Modlin, C. (2016). *Basic Ratemaking* (5th ed.). Casualty Actuarial Society.
- England, P., & Verrall, R. (2002). *Stochastic Claims Reserving in General Insurance*. British Actuarial Journal.
- Ohlsson, E., & Johansson, B. (2010). *Non-Life Insurance Pricing with GLMs*. Springer.
- CNSF (2024). *Boletín Estadístico de Seguros*. [gob.mx/cnsf](https://www.gob.mx/cnsf)
- Ferrari, A., & Russo, M. (2019). *The Definitive Guide to DAX* (2nd ed.). Microsoft Press.

---

## Portafolio completo

| # | Proyecto | Herramientas | Estado |
|---|---|---|---|
| 01 | Calculadora de Primas de Vida | Excel · VBA | ✅ Completo |
| 02 | Dashboard de Siniestralidad | Power BI · DAX | ✅ Completo |
| 03 | Modelo GLM para Tarifación | Python · statsmodels | ✅ Completo |
| 04 | Modelo de Riesgo Colectivo | R · actuar | 🔜 Próximo |
| 05 | Análisis de Portafolio de Pólizas | SQL | 🔜 Próximo |
