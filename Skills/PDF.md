# PDFs: Explicación general
Esta guia esta pensada para un agente de IA que necesita generar PDFs, con foco en el uso de tablas.

Codigo fuente (en el repo compartido `Clases`):
- `D:\Desarrollo\Clases\clases\imprenta\ArmadoPdf.vb` (namespace `ArmadoPdf`)
- `D:\Desarrollo\Clases\clases\imprenta\ArmadoPdfTablas.vb` (namespace `ArmadoPdfTablas`)
- `D:\Desarrollo\Clases\clases\imprenta\ArmadoPdfFormularios.vb` (NO SE USA POR EL MOMENTO)
- `D:\Desarrollo\Clases\clases\imprenta\ArmadoPdfFunciones.vb` (NO SE USA POR EL MOMENTO)

Prioridad de utilización:
- Salvo excepciones, en la mayoría de PDFs no es tan importante la velocidad procesamiento. Por esto, se prefiere ganar en claridad.
- Para datos que tengan una estructura de celdas y puedan ser representadas mediante tablas, se deben priorizar primero las clases de ArmadoPdfTablas.
- NO DEBE usarse el sistema de tablas de ArmadoPdf; el sistema de tablas vigente está en ArmadoPdfTablas. En tablas, ArmadoPdf contiene a lo sumo código accesorio y secundario.
- En muchos casos es útil crear una tabla invisible, sin contenido ni bordes, para obtener automáticamente áreas de celdas y ubicar capas secundarias de datos.
- En ArmadoPdfTablas, al generar los objetos internos para que trabaje la capa subyacente ArmadoPDf, el código pisa "estilo.texto". Tener cuidado con esto y evaluar el uso conveniente de Clone().
- Para datos sueltos, mínimos, o estructuras de líneas o formas que no vayan con la noción de tabla, usar directamente las clases de ArmadoPdf.
- Evitar el uso directo de iTextSharp.

## 1) Clase ArmadoPdf
- Vas a encontrar que se crean muchos objetos de las clases renglonPdf, lineaPdf, etc., escribiendo las propiedades de manera manual, una por una, por cada instancia.
  Esto ahora no es nacesario. Recientemente hicimos que muchas clases implementen IClone, por lo que ahora se puede crear un objeto e ir clonándolo.

### Secuencia de renderización y renderización por bloques
- Existe el modo de renderización por bloques, con el agregado de bloques mediante el método nuevoBloque(). El primero se crea automáticamente.
  Este modo está deshabilitado por defecto; se habilita mediante "pdf.switchs = renderizaPorBloques". Por lo general este modo no es necesario.
  La secuencia de procesamiento de un bloque, que es además la secuencia tradicional cuando este modo no está habilitado, es la siguiente:
    1. hoja.cuerpoMarcaDeAgua (marcas de agua)
    2. hoja.cuerpoRectanguloPdf con atras=True (rectángulos “atrás”)
    3. hoja.cuerpoLineaPdf (líneas)
    4. hoja.cuerpoRectanguloPdf “medio” (los que no son adelante ni atras)
    5. hoja.cuerpoCirculoPdf (círculos)
    6. hoja.cuerpoImagenPdf (imágenes)
    7. hoja.cuerpoRectanguloPdf con adelante=True (rectángulos “adelante”)
    8. hoja.cuerpoTextoPdf (renglones/texto)
    9. hoja.codigoDeBarrasPdf (códigos de barras)
    10. (opcional) mostrarPlantilla (la grilla/plantilla si está habilitada)
    Cada bloque agregado se imprime por sobre el anterior.

### `entidades.areaYContenedorPdf`

#### Que es
`areaYContenedorPdf` representa un rectangulo (area) y la hoja (`hojaPdf`) donde ese rectangulo vive.
Se usa como "handle" para dibujar dentro de un area sin perder de vista en qué pagina hay que agregar el contenido.

#### API
- `Public Property rectangulo As rectanguloPdf`
- `Public Property hoja As hojaPdf`
- `Clone() As Object`: clona el objeto (clona el rectangulo; mantiene referencia a la hoja).
- `posicionX(relativaX As Integer) As Integer`: `rectangulo.margenInferiorIzquierdoX + relativaX`.
- `posicionY(relativaY As Integer) As Integer`: `rectangulo.margenInferiorIzquierdoY + relativaY`.
- `centroX() As Integer`: centro en X.
- `centroY() As Integer`: centro en Y.

## 2) `entidades.ArmadoPdfTablas`
`ArmadoPdfTablas` es un conjunto de clases para armar tablas sobre `ArmadoPdf` de forma declarativa (columnas por porcentaje, alturas de filas, estilos, bordes) y con soporte de salto de página.

### Componentes principales
- `TablaPdf`: la tabla (renderiza, pagina, expone recuadros).
- `columnaTablaPdf`: define columnas (ancho %, título, estilos por celda).
- `iDatosTablaPdf` + `DatosTabla`: stream de celdas (texto + estilo por celda).
- `hojaPdfSetup`: tamaño de hoja, márgenes y cursores (`curX/curY`).
- `Toolbox01`: helpers que dibujan dentro de un `areaYContenedorPdf` (checkbox/cinta).
- `areaYContenedorPdfExtensions`: extension method para crear una tabla dentro de un área.

#### Patrón base para crear y escribir una tabla

Checklist:
1. Tener una `hojaPdf` actual.
2. Crear `TablaPdf` (idealmente con el constructor que mantiene el `ArmadoPdf` real si esperas saltos).
3. Configurar `HojaSetup` (margenes, `curX`, `curY`, tamanio), `Ancho`, alturas y estilos.
4. Definir columnas (porcentajes que sumen 100).
5. Cargar datos en `DatosTabla` en orden de lectura por filas.
6. Llamar `tabla.Escribir()`.
7. Actualizar tu cursor externo con `tabla.HojaSetup.curY`.

### Notas de paginacion
- `TablaPdf.Escribir()` puede generar nuevas hojas si no hay espacio (`HojaSetup.HayEspacio` / `HojaSetup.SaltoDePagina`).
- Si el código llamador va a seguir dibujando después de una tabla, debe seguir usando la hoja devuelta por `Escribir()` (o `tabla.Hojas.Last`).
- Si creás una tabla con `New TablaPdf(hoja)` (sin pasar `ArmadoPdf`), el propio comentario del codigo advierte que si hay saltos de página y el llamador no los maneja, las páginas siguientes quedarán en un `ArmadoPdf` temporal.

- Datos
      1. Estilo de la celda (Datos.GetEstilo, prioritario)
      2. Estilo de la columna para datos (col.DatoEstilo)
      3. Estilo general para celdas de datos (DatosEstilo)
  - Títulos
      1. Estilo de la columna para títulos (col.TituloEstilo, prioritario)
      2. Estilo general para celdas de título (TitulosEstilo)
  
### Bordes de celdas
- DatosBordes/TitulosBordes controlan las líneas de la grilla (verticales/horizontales).
- anchoBordeEnTablas controla el borde del rectángulo de fondo que se pinta "debajo" del texto en esa celda.
- Podés tener: grilla apagada (DatosBordes=False) y aun así "celdas con borde" usando un estilo con "anchoBordeEnTablas=1".
- Recomendación: cuando uses esto, pasar estilos clonados por celda o asegurarte de no reutilizar la misma instancia con cambios porque "DatosTabla.Agregar(texto, estilo)" pisa "estilo.texto".

### Texto multi-línea dentro de celdas (vbCrLf) y alineación vertical
- En "ArmadoPdfTablas.vb", "celdaPosicionY" activa "estilo.puedeCrecer" automáticamente si el texto contiene vbCrLf, y usa "tipoDeAlineacionVerticalEnTablas" + "interlineadoPuedeCrecer".

### `ObtenerRecuadro` / `UnirCeldas`: como dibujar contenido custom dentro de celdas

#### `ObtenerRecuadro`
Obtiene el área correspondiente a las celdas solicitadas.
`TablaPdf` expone:
- `ObtenerRecuadro(fila, columna) As areaYContenedorPdf`
- `ObtenerRecuadroFila(fila) As areaYContenedorPdf`
- `ObtenerRecuadro(filaInicio, colInicio, filaFin, colFin) As areaYContenedorPdf`

Uso:
- Primero renderizás la tabla (o al menos llamas `Recalcular()`), y despues pedís el área de una celda o bloque.
- Si el bloque cruza paginas, la función lanza Exception: el área debe pertenecer a una sola hoja.
- En la implementación, "fila=0" es "títulos" si "TitulosAlto<>0", si no, es primera fila de datos; y las filas de datos se ajustan con "filaDatos = fila - 1" cuando hay títulos.
- ObtenerRecuadroFila simplifica el uso de ObtenerRecuadro, cuando los parámetros indican filas enteras.

#### `UnirCeldas`
Obtiene el área correspondiente a las celdas solicitadas y pinta su interior de blanco simulando ser una única celda.
Opcionalmente escribir texto dentro del área (si pasás `contenido`).

Uso:
- Primero renderizás la tabla (o al menos llamas `Recalcular()`), y después pedís qué area de una celda o bloque querés unir.
- Si el bloque cruza paginas, la función lanza Exception: el área debe pertenecer a una sola hoja.
- UnirFila es un atajo de UnirCeldas, cuando los parámetros indican filas enteras.

### Tips
1. Se pueden encapsular secciones: definir `Sub dibujarX(area As areaYContenedorPdf)` y dentro dibujar fondo, titulo y contenido usando `area.hoja` y `area.rectangulo`.
2. Posicionamiento relativo: con `ares.rectangulo`, podés usar `contiguoX(n)` / `contiguoY(n)` para mover el mismo rectángulo por su propio ancho/alto.
3. Uso utilitario: si solo necesitás `TablaPdf.renglonEnRecuadro(...)` como helper, podés instanciar una `TablaPdf` temporal y usar el overload que recibe `rectanguloPdf`/`hojaPdf`.

### `hojaPdfSetup` (márgenes, tamaño, cursores)
`hojaPdfSetup` se usa para conocer el tamaño de la página, mantener coordenadas, y saber si entra más contenido:
- `curX`, `curY`: cursor actual.
- `HayEspacio(curY, espacioRequerido)`: decide si entra una fila/bloque.
- `SaltoDePagina(pdf, hojaActual)`: crea una hoja nueva y resetea cursores.

Regla:
- Para tablas, `HojaSetup.curY` es el punto de arranque en Y (arriba del proximo bloque), y se va decrementando.

### `DatosTabla`
`DatosTabla` implementa `iDatosTablaPdf` y es lo mas simple para alimentar una tabla.

Puntos:
- Cada `Agregar(...)` suma una celda (no una fila).
- La cantidad de celdas define cuantas filas hay via `RowCount(cantidadDeColumnas)`.
- `AgregarVacios(n)` es util para agregar celdas sin contenido. Se suele utilizar para agregar filas vacías enteras, agregando múltiplos de la cantidad de columnas.
- Si querés estilos por celda, usá `Agregar(texto, estilo)`.

### Helpers
`Toolbox01` dibuja dentro de un `areaYContenedorPdf`:
- `Checkbox01(contenedor, checked, checkedChar := "X", tipoDeMargenes := margenMinimo)`
  Checkbox: dibuja la casilla de verificación (recuadro, tilde -opcional-, etiqueta -puede ser cadena vacía-)
- `Cinta01(contenedor, tipoDeMargenes := margenMinimo)`
  Cinta: dibuja una cinta con bordes redondeados que puede servir como título de alguna sección. Por lo general no se prefiere este mecanismo, pero para algunos pocos casos queda muy bien.

Regla:
- Estas funciones agregan shapes/texto directamente a `contenedor.hoja`, asi que funcionan bien incluso si el area esta en otra pagina.

### Extension method: crear tabla dentro de un área
`areaYContenedorPdfExtensions.CrearTablaEnBaseA(area, tablaDeReferencia, cantidadDeColumnas)`:
- Crea una `TablaPdf` dentro del rectángulo `area`.
- Copia configuración base (setup/estilos/altos) desde `tablaDeReferencia`.
- Distribuye 100% del ancho entre `cantidadDeColumnas`.
- Si la tabla de referencia se creó con New "TablaPdf(hoja)" (Genera un ArmadoPdf temporal), la subtabla setea alineaciones al centro. Si querés conservar dichas alineaciones, re-setearlas posteriormente.

Uso recomendado para un agente:
- Tenés una tabla "template" ya configurada.
  a. Si tenés una tabla y querés escribir contenido con otro formato.
  b. Si tenés una tabla que se use sólo para los fines de posicionar subelementos (una tabla invisible, sin textos ni bordes)
- Necesitás una subtabla dentro de una celda/recuadro: pedís el `areaYContenedorPdf` y creás una tabla nueva en esa área sin recalcular todo manual.
- Las columnas van a tener el mismo ancho o casi (digo `casi` porque puede ajustar alguna para que la suma de anchos coincida con el ancho total).

Tips:
1. Tomar un recuadro de varias filas/columnas y crear una tabla con otros tamaños o formatos.
2. Puede ser útil que las subtablas se dibujen en funciones aparte, para modularizar mejor.
3. Ajuste posterior: despues de crear la subtabla, setear `DatosBordes`, `DatosEstilo` y ajustar `Columnas(i).Ancho` antes de `Escribir()`.