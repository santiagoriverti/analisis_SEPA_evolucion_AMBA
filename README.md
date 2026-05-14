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
