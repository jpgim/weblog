+++
title = 'Entre Ríos De Noche'
date = '2026-07-22T14:54:16-03:00'
draft = false
showtoc = true
tocopen = true
tags = ["NASA", "EOG", "GDAL", "GIS"]
+++

## Introducción.
Hace unos años se volvió viral una imagen de la [Tierra de noche](https://science.nasa.gov/earth/earth-observatory/earth-at-night/maps/#2012-color-original). Pero sobre todo llamo mucho la atención los pesqueros que usan luces para atraer y [pescar calamares](https://science.nasa.gov/earth/earth-observatory/something-fishy-in-the-atlantic-night/)

Y si bien están disponibles imágenes para la descarga siempre me pregunte que tan fácil (o difícil) será obtener una imagen nocturna reciente de Entre Rios, Argentina utilizando herramientas de linea de comandos.

NASA pone a disposición del público una gran cantidad de productos científicos relacionados con la observación de la Tierra. Uno de ellos es [Black Marble](https://www.earthdata.nasa.gov/data/projects/black-marble), un proyecto que utiliza las observaciones de la banda Day/Night Band (DNB) del instrumento VIIRS (Visible Infrared Imaging Radiometer Suite) instalado a bordo de los satélites Suomi NPP, NOAA-20 y NOAA-21. Estos productos generan un registro de largo plazo de la iluminación nocturna de la superficie terrestre y son capaces de detectar niveles muy bajos de luz, desde la iluminación de una ciudad hasta el brillo producido por un barco en el océano. A partir de esto es posible obtener indicadores de actividad humana, como contaminación lumínica, actividad pesquera o evolución del crecimiento de las ciudades.

Esto puedo usarlo para obtener una imagen nocturna de cuanta "luz" se emite de noche en una region del mundo determinada, es decir, me interesa obtener una imagen de la radiancia nocturna de una región concreta. El inconveniente es que Black Marble está orientado principalmente a la investigación científica y sus productos no siempre resultan sencillos de utilizar para quien solo necesita descargar una imagen y procesarla.

Una alternativa mucho más cómoda son los productos derivados publicados por el [Earth Observation Group (EOG)](https://payneinstitute.mines.edu/eog/), perteneciente al Payne Institute de la Colorado School of Mines. A partir de las observaciones de VIIRS, el EOG genera compuestos mensuales libres de nubes (Monthly Cloud-free DNB Composites) que pueden descargarse, previo registro gratuito, en formato GeoTIFF. Cada píxel almacena la radiancia medida en nW/cm²/sr (nanovatios por centímetro cuadrado por estereorradián) y los archivos se distribuyen con una resolución espacial de 15 segundos de arco que los convierte en un excelente punto de partida para lo que quiero realizar.

## 1. Obteniendo imágenes.
EOG publica tanto imágenes globales como imágenes divididas en teselas geográficas. Cada tesela cubre una región fija de la superficie terrestre y puede descargarse de forma independiente.
Las imágenes divididas en 6 teselas y son:
* Tesela 1: 180° Oeste - 75° Norte
* Tesela 2: 60° Oeste -  75° Norte
* Tesela 3: 60° Este -   75° Norte
* Tesela 4: 180° Oeste - 00° Norte
* Tesela 5: 60° Oeste -  00° Norte
* Tesela 6: 60° Este -   00° Norte

_nota: las coordenadas corresponden a esquina superior derecha de cada tesela_


Como Entre Rios esta atravesada por el meridiano 60° Oeste, voy a necesitar descargar las teselas 4 y 5. Así obtengo los archivos

 `SVDNB_npp_20260601-20260630_00N180W_vcmcfg_v10_c20260` y 

 `SVDNB_npp_20260601-20260630_00N060W_vcmcfg_v10_c202607102200.tgz`.
 
Los nombres indican que son datos del satelite Suomi NPP del 01 de junio de 2026 al 30 de junio de 2026.

Descomprimo para obtener los GeoTIFF:

```console {linenos=false}
$ gzip -d SVDNB_npp_20260601-20260630_00N060W_vcmcfg_v10_c202607102200.tgz 
```
```console {linenos=false}
$ gzip -d SVDNB_npp_20260601-20260630_00N180W_vcmcfg_v10_c202607102200.tgz 
```
```console {linenos=false}
$ tar xvf SVDNB_npp_20260601-20260630_00N060W_vcmcfg_v10_c202607102200.tar 
SVDNB_npp_20260601-20260630_00N060W_vcmcfg_v10_c202607102200.avg_rade9h.tif
SVDNB_npp_20260601-20260630_00N060W_vcmcfg_v10_c202607102200.cf_cvg.tif
SVDNB_npp_20260601-20260630_00N060W_vcmcfg_v10_c202607102200.cvg.tif
```
```console {linenos=false}
$ tar xvf SVDNB_npp_20260601-20260630_00N180W_vcmcfg_v10_c202607102200.tar 
SVDNB_npp_20260601-20260630_00N180W_vcmcfg_v10_c202607102200.avg_rade9h.tif
SVDNB_npp_20260601-20260630_00N180W_vcmcfg_v10_c202607102200.cf_cvg.tif
SVDNB_npp_20260601-20260630_00N180W_vcmcfg_v10_c202607102200.cvg.tif
```

y como solo necesito la radiancia, utilizao los archivos `avg_rade9h`  que son los que contienen radiancia media redondeada a dos decimales (nW/cm²/sr). Puedo eliminar el resto.

## 2. Cookie cutter.
A partir de este punto voy a usar las herramientas de la suite [GDAL](https://gdal.org). GDAL es la librería de referencia para trabajar con información geográfica desde la línea de comandos y con ella se pueden leer, convertir y editar archivos ráster y vectoriales

Como ya tengo los datos de una region grande del globo pero los necesito localizado a la provincia de Entre Ríos, me voy a apoyar en los geoservicios del Instituto Geográfico Nacional [IGN](https://www.ign.gob.ar/) para conseguir el contorno de la provincia.

Consulto que capas vectoriales hay en el servicio WFS del IGN y veo cuál me puede servir para "obtener Entre Rios"

```console {linenos=false}
$ ogrinfo WFS:"https://wms.ign.gob.ar/geoserver/ows" | grep -i "provincia"
83: ign:hitos_interprovinciales (title: Hitos interprovinciales)
97: ign:linea_de_limite_070111 (title: Límite interprovincial)
135: ign:provincia (title: Provincia)
150: ign:red_provincial (title: Red geodésica Provincial)
158: ign:vial_provincial (title: Red vial provincial)
```

Con esta información veo que la capa que necesito es `provincia`. Consulto detalles adicionales:

```console {linenos=false}
$ ogrinfo -so WFS:"https://wms.ign.gob.ar/geoserver/ows" ign:provincia
INFO: Open of `WFS:https://wms.ign.gob.ar/geoserver/ows'
      using driver `WFS' successful.
Metadata:
  ABSTRACT=Servicio WFS del Instituto Geográfico Nacional de la República Argentina basado en la implementación del standard por el software Geoserver.
  PROVIDER_NAME=Instituto Geográfico Nacional
  TITLE=WFS Instituto Geográfico Nacional

Layer name: ign:provincia
Metadata:
  ABSTRACT=División político territorial de primer orden. Incluye la Ciudad Autónoma de Buenos Aires (CABA).
  KEYWORD_1=features
  KEYWORD_2=provincia
  TITLE=Provincia
Geometry: Unknown (any)
Feature Count: 24
Extent: (-74.000000, -90.000000) - (-25.000000, -21.780857)
Layer SRS WKT:
GEOGCRS["WGS 84",
    ENSEMBLE["World Geodetic System 1984 ensemble",
        MEMBER["World Geodetic System 1984 (Transit)"],
        MEMBER["World Geodetic System 1984 (G730)"],
        MEMBER["World Geodetic System 1984 (G873)"],
        MEMBER["World Geodetic System 1984 (G1150)"],
        MEMBER["World Geodetic System 1984 (G1674)"],
        MEMBER["World Geodetic System 1984 (G1762)"],
        MEMBER["World Geodetic System 1984 (G2139)"],
        ELLIPSOID["WGS 84",6378137,298.257223563,
            LENGTHUNIT["metre",1]],
        ENSEMBLEACCURACY[2.0]],
    PRIMEM["Greenwich",0,
        ANGLEUNIT["degree",0.0174532925199433]],
    CS[ellipsoidal,2],
        AXIS["geodetic latitude (Lat)",north,
            ORDER[1],
            ANGLEUNIT["degree",0.0174532925199433]],
        AXIS["geodetic longitude (Lon)",east,
            ORDER[2],
            ANGLEUNIT["degree",0.0174532925199433]],
    USAGE[
        SCOPE["Horizontal component of 3D system."],
        AREA["World."],
        BBOX[-90,-180,90,180]],
    ID["EPSG",4326]]
Data axis to CRS axis mapping: 2,1
Geometry Column = geom
gml_id: String (0.0) NOT NULL
gid: Integer (0.0)
entidad: Integer64 (0.0)
fna: String (0.0)
gna: String (0.0)
nam: String (0.0)
in1: String (0.0)
fdc: String (0.0)
sag: String (0.0)
```

Descargo la capa vectorial que representa la provincia de Entre Ríos:

```console {linenos=false}
$ ogr2ogr -f "GPKG" entre_rios.gpkg WFS:"https://wms.ign.gob.ar/geoserver/ows" ign:provincia -where "nam LIKE 'Entre R%'"
```


Como necesité descargar dos teselas para abarcar la provincia de Entre Ríos, ahora conviene crear un raster virtual con `gdalbuildvrt`.

```console {linenos=false}
$ gdalbuildvrt rastersud.vrt SVDNB_npp_20260601-20260630_00N060W_vcmcfg_v10_c202607102200.avg_rade9h.tif SVDNB_npp_20260601-20260630_00N180W_vcmcfg_v10_c202607102200.avg_rade9h.tif 
0...10...20...30...40...50...60...70...80...90...100 - done.
``` 

Y ahora uso la capa vectorial de IGN para usarla como cortante de galletas, el "cookie cutter". `gdalwarp` es la herramienta usada para ello.

```console {linenos=false}
$ gdalwarp -cutline entre_rios.gpkg -crop_to_cutline -dstnodata -9999 rastersud.vrt radiancia_entre_rios.tif
Creating output file that is 714P x 931L.
Processing rastersud.vrt [1/1] : 0...10...20...30...40...50...60...70...80...90...100 - done.
```
## 3. Mejorar la visualización.
Hay un par de acciones para lograr que el resultado sea una imagen más fácil de interpretar. Usé el mismo criterio que EOG para sus productos destinados a los artistas que es limitar la radiancia al intervalo 0–75 nW/cm²/sr y aplicar una transformación de raíz cuadrada. Esto reduce el efecto de unas pocas luces muy intensas y hace más visibles las zonas con baja radiancia, sin perder la sensación de continuidad en la imagen.
 

```console {linenos=false}
$ gdal_calc.py -A radiancia_entre_rios.tif --outfile radiancia_entre_rios_sqrt.tif --calc "sqrt(clip(A,0,75))" --type Float32
0.. 0.. 0.. 0.. 0.. 1.. 1.. 1.. 1.. 1.. 2.. 2.. 2.. 2.. 3.. 3.. 3.. 3.. 3.. 4.. 4.. 4.. 4.. 4.. 5.. 5.. 5.. 5.. 6.. 6.. 6.. 6.. 6.. 7.. 7.. 7.. 7.. 7.. 8.. 8.. 8.. 8.. 9.. 9.. 9.. 9.. 9.. 10.. 10.. 10.. 10.. 10.. 11.. 11.. 11.. 11.. 12.. 12.. 12.. 12.. 12.. 13.. 13.. 13.. 13.. 13.. 14.. 14.. 14.. 14.. 15.. 15.. 15.. 15.. 15.. 16.. 16.. 16.. 16.. 16.. 17.. 17.. 17.. 17.. 18.. 18.. 18.. 18.. 18.. 19.. 19.. 19.. 19.. 19.. 20.. 20.. 20.. 20.. 21.. 21.. 21.. 21.. 21.. 22.. 22.. 22.. 22.. 22.. 23.. 23.. 23.. 23.. 24.. 24.. 24.. 24.. 24.. 25.. 25.. 25.. 25.. 25.. 26.. 26.. 26.. 26.. 27.. 27.. 27.. 27.. 27.. 28.. 28.. 28.. 28.. 28.. 29.. 29.. 29.. 29.. 30.. 30.. 30.. 30.. 30.. 31.. 31.. 31.. 31.. 31.. 32.. 32.. 32.. 32.. 33.. 33.. 33.. 33.. 33.. 34.. 34.. 34.. 34.. 34.. 35.. 35.. 35.. 35.. 36.. 36.. 36.. 36.. 36.. 37.. 37.. 37.. 37.. 37.. 38.. 38.. 38.. 38.. 39.. 39.. 39.. 39.. 39.. 40.. 40.. 40.. 40.. 40.. 41.. 41.. 41.. 41.. 42.. 42.. 42.. 42.. 42.. 43.. 43.. 43.. 43.. 43.. 44.. 44.. 44.. 44.. 45.. 45.. 45.. 45.. 45.. 46.. 46.. 46.. 46.. 46.. 47.. 47.. 47.. 47.. 48.. 48.. 48.. 48.. 48.. 49.. 49.. 49.. 49.. 50.. 50.. 50.. 50.. 50.. 51.. 51.. 51.. 51.. 51.. 52.. 52.. 52.. 52.. 53.. 53.. 53.. 53.. 53.. 54.. 54.. 54.. 54.. 54.. 55.. 55.. 55.. 55.. 56.. 56.. 56.. 56.. 56.. 57.. 57.. 57.. 57.. 57.. 58.. 58.. 58.. 58.. 59.. 59.. 59.. 59.. 59.. 60.. 60.. 60.. 60.. 60.. 61.. 61.. 61.. 61.. 62.. 62.. 62.. 62.. 62.. 63.. 63.. 63.. 63.. 63.. 64.. 64.. 64.. 64.. 65.. 65.. 65.. 65.. 65.. 66.. 66.. 66.. 66.. 66.. 67.. 67.. 67.. 67.. 68.. 68.. 68.. 68.. 68.. 69.. 69.. 69.. 69.. 69.. 70.. 70.. 70.. 70.. 71.. 71.. 71.. 71.. 71.. 72.. 72.. 72.. 72.. 72.. 73.. 73.. 73.. 73.. 74.. 74.. 74.. 74.. 74.. 75.. 75.. 75.. 75.. 75.. 76.. 76.. 76.. 76.. 77.. 77.. 77.. 77.. 77.. 78.. 78.. 78.. 78.. 78.. 79.. 79.. 79.. 79.. 80.. 80.. 80.. 80.. 80.. 81.. 81.. 81.. 81.. 81.. 82.. 82.. 82.. 82.. 83.. 83.. 83.. 83.. 83.. 84.. 84.. 84.. 84.. 84.. 85.. 85.. 85.. 85.. 86.. 86.. 86.. 86.. 86.. 87.. 87.. 87.. 87.. 87.. 88.. 88.. 88.. 88.. 89.. 89.. 89.. 89.. 89.. 90.. 90.. 90.. 90.. 90.. 91.. 91.. 91.. 91.. 92.. 92.. 92.. 92.. 92.. 93.. 93.. 93.. 93.. 93.. 94.. 94.. 94.. 94.. 95.. 95.. 95.. 95.. 95.. 96.. 96.. 96.. 96.. 96.. 97.. 97.. 97.. 97.. 98.. 98.. 98.. 98.. 98.. 99.. 99.. 99.. 99.. 100 - Done
```

Otra acción es elegir una paleta de colores adecuada. Me interesa que los colores de la imagen respondan a la los datos por lo que conviene  elegirlos según como se distribuyen las mediciones. Y para ver eso uso `gdalinfo -hist radiancia_entre_rios_sqrt.tif` Dey la salida de ese comando la parte relevante en este caso es la siguiente:
```
Band 1 Block=714x2 Type=Float32, ColorInterp=Gray
  Min=0.000 Max=8.660 
  Minimum=0.000, Maximum=8.660, Mean=0.673, StdDev=0.578
  256 buckets from -0.0169809 to 8.67724:
  1270 0 0 1 1 8 5 8 18 36 76 361 1571 6281 18791 41428 88414 90515 62827 35868 23558 12566 7555 5135 4472 3304 2077 2300 1634 1589 1322 1126 987 928 817 726 630 585 498 485 486 432 381 358 308 305 307 273 251 226 254 232 241 194 145 174 160 182 148 141 147 122 145 131 105 95 89 83 103 87 88 92 77 88 80 84 80 84 75 79 76 62 68 69 55 82 58 55 72 59 56 53 35 53 47 54 57 53 48 38 41 44 29 30 39 48 38 44 49 34 52 39 38 41 35 24 21 24 30 27 31 45 31 34 33 37 28 32 24 26 31 24 28 32 28 28 18 28 22 28 26 33 25 25 22 32 16 20 15 28 19 22 28 33 26 16 25 32 29 15 22 25 20 20 21 17 11 29 20 22 14 19 17 17 17 13 15 16 17 14 16 19 27 13 18 14 12 17 10 15 23 18 19 5 15 16 13 5 20 8 8 8 8 18 12 17 7 10 8 16 10 11 7 12 9 6 15 9 12 11 7 12 15 13 13 11 11 14 12 18 12 13 8 8 11 7 9 10 11 17 9 3 13 5 8 7 5 10 8 10 3 7 4 7 5 187 
  NoData Value=3.4028234663852886e+38
  Metadata:
    STATISTICS_APPROXIMATE=YES
    STATISTICS_MAXIMUM=8.6602544784546
    STATISTICS_MEAN=0.67281373245889
    STATISTICS_MINIMUM=0
    STATISTICS_STDDEV=0.5783040484076
    STATISTICS_VALID_PERCENT=62.45
```

En este punto creo que conviene aclarar algunas cuestiones.

Este GeoTIFF es un archivo ráster que lleva incrustados los metadatos geográficos necesarios para ubicarlo sobre la Tierra: el sistema de referencia de coordenadas y la transformación que asigna a cada píxel su posición y tamaño real en el terreno. A diferencia de una imagen convencional en  donde los píxeles codifican color, este archivo contiene una única banda de valores Float32 que representan directamente una medición física de radiancia lumínicé medido desde el espacio con los instrunentos del satelite. 

Observando los "buckets" que informa `gdalinfo` se nota que la distribución de esos valores no sigue una campana de Gauss, está fuertemente sesgada a la derecha, donde la gran mayoría presenta valores cercanos a cero y solo un pequeño porcentaje concentra valores que representan luces intensas. Por esta razón, es mejor evaluar la estructura de la luminosidad mediante percentiles P50 a P99.99.

Un pequeño script en `python`con `gdal` y `numpy` es útil para esta tarea:


```python console {linenos=false}
from osgeo import gdal
import numpy as np

raster = gdal.Open("radiancia_entre_rios_sqrt.tif")
banda = raster.GetRasterBand(1)
radiancia = banda.ReadAsArray()

valor_nodata = banda.GetNoDataValue() #limpiar no-data
if valor_nodata is not None:
    radiancia = radiancia[radiancia != valor_nodata]

for percentil in (50, 75, 90, 95, 99, 99.5, 99.9):
    print(f"P{percentil:>4}: {np.percentile(radiancia, percentil):8.2f}")

```
```console console {linenos=false}
$ python3 percentiles.py 
P  50:     0.58
P  75:     0.64
P  90:     0.75
P  95:     0.96
P  99:     2.72
P99.5:     4.41
P99.9:     7.72
```

Estos valores de percentiles son los que uso para crear una paleta de colores que luego uso con `gdaldem color-relief` 
```
nv      0    0      0      # no-data
0.00    0    0      5      # Negro
0.58    0    0     30      # Azul muy oscuro
0.64    0   15     70      # Azul oscuro
0.75    0   60    160      # Azul
0.96    0  140    255      # Celeste intenso
2.72  245  248    255      # Blanco azulado
4.41  255  235    180      # Blanco cálido
7.72  255  250    245      # Blanco muy brillante
8.66  255  255    255      # Blanco saturado
```
```console {linenos=false}
$ gdaldem color-relief radiancia_entre_rios_sqrt.tif paleta_colores.txt luces_entre_rios.tif
0...10...20...30...40...50...60...70...80...90...100 - done.
```
           
## 4. Imagen final.

Y para concluir, ahora obtengo la imagen final. Y si bien cualquier herramienta GIS puede visualizar este trabajo sin problemas, para agregarla a este post la convierto a PNG:

```console {linenos=false}
$ gdal_translate -of PNG -co ZLEVEL=9 luces_entre_rios.tif luces_entre_rios.png
Input file size is 714, 931
0...10...20...30...40...50...60...70...80...90...100 - done.
```
{{< figure
    src="luces_entre_rios.png"
    link="luces_entre_rios.png"
    alt="Radiancia nocturna de Entre Ríos"
    title="Radiancia Nocturna en Entre Ríos"
    caption="Radiancia nocturna de la provincia de Entre Ríos obtenida a partir del producto VIIRS Monthly Cloud-Free DNB Composite (VCMSLCFG). Visualización elaborada por el autor mediante GDAL a partir de datos del Earth Observation Group (EOG), Payne Institute for Public Policy, Colorado School of Mines. Licencia CC BY 4.0."
>}}



## Conclusión.
La idea de este post era poder explorar herramientas GIS en linea de comandos como GDAL y como pueden usarse para procesar datos que instituciones científicas ponen a disposicion del público. 


## Referencias:

Las siguientes referencias son las recomendadas por el Earth Observation Group (EOG) para citar los productos VIIRS Nighttime Lights utilizados en este artículo.

- Elvidge, C. D., Baugh, K., Zhizhin, M., Hsu, F. C., & Ghosh, T. *VIIRS night-time lights*. International Journal of Remote Sensing, 38, 5860–5879, 2017.
