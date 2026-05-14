# analisis_SEPA_canasta_geo

El proyecto busca procesar la base SEPA de precios minoristas para construir el **Índice de Consumo Masivo UADE (ICM-UADE)** a nivel de sucursal individual y generar un mapa interactivo georreferenciado con todas las sucursales del país, mostrando el valor de la canasta de 30 productos en cada punto de venta.

---

# 📌 Objetivo

A diferencia del análisis agregado por provincia o región, este proyecto construye el ICM-UADE **a nivel de sucursal individual**, permitiendo:

- Visualizar geográficamente la dispersión de precios dentro de una misma jurisdicción
- Identificar diferencias por cadena comercial en cada zona del país
- Mostrar el detalle de los 30 productos de la canasta para cada punto de venta
- Construir rankings de cadenas según el valor de la canasta a nivel nacional y por región

---

# ⚙️ Funcionalidades principales

## Procesamiento de datos

- Lectura de archivos `.csv.gz` semestrales del SEPA (formato wide, una columna por día)
- Filtrado por códigos EAN correspondientes a los 30 productos de la canasta
- Conversión de formato wide → long para el promedio mensual
- Detección automática del factor de escala de precios
- Filtrado de placeholders (precios menores a $5) y duplicados por archivos solapados

## Enriquecimiento

Cruce con:
- Maestro de productos (176.702 registros): marca, rubro, categoría, proveedor
- Maestro de sucursales (3.611 registros): coordenadas geográficas, provincia, región, localidad, barrio, tipo de sucursal

## Identificación de cadenas

Mapeo verificado por `(id_comercio, id_bandera)` que distingue las distintas banderas dentro de cada empresa:

- **Cencosud (id_comercio = 9)**: Vea, Disco, Jumbo
- **Carrefour (id_comercio = 10)**: Carrefour, Carrefour Market, Carrefour Express
- **ChangoMas / DORINKA (id_comercio = 11)**: ChangoMas, Hiper ChangoMas, Mi ChangoMas
- **Libertad (id_comercio = 16)**: Hipermercado Libertad, Mini Libertad
- Y cadenas con bandera única: DIA, Coto, La Anónima, Cooperativa Obrera, Toledo, LAR, Pasamonte, entre otras

Se filtran cadenas no representativas de consumo masivo (FULL, Easy, Mercado Libre).

## Construcción de la canasta por sucursal

### Promedio mensual por producto-sucursal
Para cada combinación (sucursal, producto), se calcula el precio promedio del mes a partir de las observaciones diarias.

### Canasta de 30 productos por sucursal
La canasta total resulta de la suma:
Canasta = Σ (precio_promedio_mensual_producto × cantidad_mensual)

para los 30 productos definidos. Si una sucursal no comercializa algún producto, se imputa con el promedio nacional de ese producto para preservar la comparabilidad.

### Filtro de representatividad
Solo se incluyen sucursales que reportan al menos 20 de los 30 productos propios, garantizando que la canasta no esté dominada por valores imputados.

## Visualización geográfica

### Mapa interactivo HTML (Folium)
Mapa con todas las sucursales del país representadas como círculos coloreados según el valor de la canasta:

- **Verde**: canasta más barata
- **Rojo**: canasta más cara
- Escala continua basada en los percentiles 5 y 95 de la distribución

### Tooltip y popup detallado
Al pasar el mouse sobre cada sucursal se muestra cadena + precio. Al hacer click se despliega un panel con:

- Cadena y nombre de sucursal
- Ubicación (barrio/localidad, provincia, región)
- Canasta total destacada
- Cantidad de productos propios vs imputados
- **Tabla detallada de los 30 productos** agrupados por categoría (Lácteos, Almacén, Bebidas, Limpieza, Higiene, Snacks) con precio unitario, cantidad mensual y subtotal

### Corrección cartográfica
Las Islas Malvinas se muestran etiquetadas como "Islas Malvinas (ARG)" mediante un overlay sobre el tile base.

---

# 🧺 Canasta utilizada

Mismos 30 productos que el análisis longitudinal del ICM-UADE, distribuidos en seis categorías:

- **Lácteos** (5): leche entera, yogur, queso untable, manteca, leche chocolatada
- **Almacén** (8): aceite, arroz, fideos, harina, yerba, café, galletitas, sal
- **Bebidas** (5): gaseosa cola, gaseosa sin azúcar, agua saborizada, cerveza, vino
- **Limpieza** (3): lavandina, detergente, limpiador
- **Higiene** (7): shampoo, acondicionador, jabón, antitranspirante, hilo dental, toallas femeninas, papel higiénico
- **Snacks** (2): confites de chocolate, snacks salados

Cada producto tiene EAN, descripción, cantidad fija mensual y categoría.

---

# 📂 Inputs requeridos

El notebook espera encontrar en `/content/`:

```text
/content/
│
├── 042026_pais_parte1COMPLETO.csv.gz     (días 1-15 del mes)
├── 042026_pais_parte2COMPLETO.csv.gz     (días 16-30 del mes)
│
├── Maestro de Productos Interno.xlsx
├── maestro_sucursales_completo.xlsx
```

---

# 📤 Outputs generados

- `mapa_canasta_pais_abril2026_detalle.html` — Mapa interactivo georreferenciado con detalle de productos por sucursal (~15-25 MB)
- `ranking_cadenas_nacional.png` — Gráfico de ranking de cadenas a nivel nacional
- `ranking_cadenas_amba.png` — Gráfico de ranking de cadenas en AMBA (CABA + PBA)

---

# 🔍 Verificación del mapeo de cadenas

El proyecto incluye bloques de verificación que validan la consistencia del mapeo `(id_comercio, id_bandera) → cadena` mediante cinco checks:

1. **Cantidad de sucursales por cadena** vs valores esperados del mercado real
2. **Distribución geográfica** (La Anónima en Patagonia, Toledo en costa atlántica, etc.)
3. **Coherencia económica del ranking de precios**
4. **Ausencia de ubicaciones duplicadas** entre cadenas distintas
5. **Productos que cada cadena no comercializa** (validación cualitativa)

---

# 📊 Hallazgos principales (abril 2026)

- **1.829 sucursales** del país con canasta válida (≥20 productos propios de 30)
- **Carrefour** y **Hipermercado Libertad** lideran el ranking de cadenas más baratas a nivel nacional
- **Cooperativa Obrera** es la cadena más barata en AMBA, por su modelo de cooperativa de consumo
- **La Anónima** y **Disco** son las más caras del país, esta última por su posicionamiento premium
- La dispersión de precios entre cadenas es **menor en AMBA (5,4%) que a nivel nacional (12,1%)**, reflejando mayor competencia comercial en el área metropolitana

---

# ⚠️ Limitaciones

- El análisis corresponde al **promedio mensual** del último mes vencido, no captura variaciones intra-mensuales
- Las sucursales con **coordenadas inválidas** o fuera del territorio argentino son descartadas (~5% del total)
- Los precios corresponden al precio de **góndola sin descuentos**; el gasto efectivo puede ser menor con promociones o medios de pago específicos
- Algunas cadenas con muy pocos productos relevados (FULL, Easy, Mercado Libre) se filtran del análisis por no ser representativas del consumo masivo
- La imputación de productos faltantes con el promedio nacional puede atenuar las diferencias reales entre sucursales con muy poca cobertura

---

# 🛠️ Stack técnico

- **Python 3** ejecutado en Google Colab
- **pandas** + **numpy** para procesamiento
- **folium** + **branca** para el mapa interactivo HTML
- **matplotlib** para gráficos de ranking
- Tiempo de procesamiento: ~5-10 minutos en Colab gratuito

---

# 📚 Fuentes

- **SEPA**: datos.produccion.gob.ar/dataset/sepa-precios — Ministerio de Economía de la Nación
- **Marco normativo**: Resolución 12/2016 de la ex Secretaría de Comercio

---

# 👥 Autores

INECO — Instituto de Economía, UADE  
Santiago Riverti
