# PDFs: como usar `entidades.ArmadoPdfTablas` y `entidades.areaYContenedorPdf`
Esta guia esta pensada para un agente de IA que necesita generar PDFs usando las clases de `entidades` basadas en `ArmadoPdf`/`hojaPdf`.

Codigo fuente (en el repo compartido `Clases`):
- `D:\Desarrollo\Clases\clases\imprenta\ArmadoPdfTablas.vb` (namespace `ArmadoPdfTablas`)
- `D:\Desarrollo\Clases\clases\imprenta\ArmadoPdf.vb` (clase `areaYContenedorPdf`)

## 1) `entidades.areaYContenedorPdf`

### Que es
`areaYContenedorPdf` representa un rectangulo (area) y la hoja (`hojaPdf`) donde ese rectangulo vive.
Se usa como "handle" para dibujar dentro de un area sin perder de vista en que pagina hay que agregar el contenido.

### API (segun `ArmadoPdf.vb`)
- `Public Property rectangulo As rectanguloPdf`
- `Public Property hoja As hojaPdf`
- `Clone() As Object`: clona el objeto (clona el rectangulo; mantiene referencia a la hoja).
- `posicionX(relativaX As Integer) As Integer`: `rectangulo.margenInferiorIzquierdoX + relativaX`.
- `posicionY(relativaY As Integer) As Integer`: `rectangulo.margenInferiorIzquierdoY + relativaY`.
- `centroX() As Integer`: centro en X.
- `centroY() As Integer`: centro en Y.

### Reglas para un agente
- Siempre escribir/dibujar contra `area.hoja` (no contra una variable `hoja` vieja):
  - Texto: `area.hoja.cuerpoTextoPdf.Add(renglon)`
  - Rectangulos: `area.hoja.cuerpoRectanguloPdf.Add(rct)`
  - Lineas/circulos/imagenes: donde corresponda.
- Usar `posicionX/posicionY` cuando quieras offsets dentro del area y `centroX/centroY` cuando quieras centrar.

Ejemplo (centrar un titulo dentro de un recuadro):
```vbnet
Dim area As areaYContenedorPdf = tabla.ObtenerRecuadro(0, 0, 1, 1)

Dim r As renglonPdf = itemEstandar.Clone()
r.tipoDeAlineacion = renglonPdf.alineacion.centrado
r.negrita = True
r.texto = "ORDEN DE VENTA"
r.posicionX = area.centroX()
r.posicionY = area.rectangulo.margenInferiorIzquierdoY + area.rectangulo.alto - r.tamanio - 4

area.hoja.cuerpoTextoPdf.Add(r)
```

## 2) `entidades.ArmadoPdfTablas`
`ArmadoPdfTablas` es un conjunto de clases para armar tablas sobre `ArmadoPdf` de forma declarativa (columnas por porcentaje, alturas de filas, estilos, bordes) y con soporte de salto de pagina.

### Componentes principales
- `TablaPdf`: la tabla (renderiza, pagina, expone recuadros).
- `columnaTablaPdf`: define columnas (ancho %, titulo, estilos por celda).
- `iDatosTablaPdf` + `DatosTabla`: stream de celdas (texto + estilo por celda).
- `hojaPdfSetup`: tamanio de hoja, margenes y cursores (`curX/curY`).
- `Toolbox01`: helpers que dibujan dentro de un `areaYContenedorPdf` (checkbox/cinta).
- `areaYContenedorPdfExtensions`: extension method para crear una tabla dentro de un area.

## 3) Patron base para crear y escribir una tabla

Checklist:
1. Tener una `hojaPdf` actual.
2. Crear `TablaPdf` (idealmente con el constructor que mantiene el `ArmadoPdf` real si esperas saltos).
3. Configurar `HojaSetup` (margenes, `curX`, `curY`, tamanio), `Ancho`, alturas y estilos.
4. Definir columnas (porcentajes que sumen 100).
5. Cargar datos en `DatosTabla` en orden de lectura por filas.
6. Llamar `tabla.Escribir()`.
7. Actualizar tu cursor externo con `tabla.HojaSetup.curY`.

Ejemplo (basado en `OrdenDeVenta_00632.vb`):
```vbnet
Dim datos As New DatosTabla()
Dim tabla As New TablaPdf(hoja)

tabla.HojaSetup = New hojaPdfSetup(hojaPdf.tipoDeHoja.A4, MARGEN, MARGEN, MARGEN, MARGEN)
tabla.TitulosAlto = 0
tabla.TitulosBordes = False
tabla.DatosAlto = ALTORENGLON
tabla.DatosBordes = True

tabla.HojaSetup.curX = MARGEN
tabla.HojaSetup.curY = y
tabla.Ancho = CInt(tabla.HojaSetup.AnchoUtil * 0.27)

tabla.Columnas.Add(New columnaTablaPdf With {.Ancho = 40})
tabla.Columnas.Add(New columnaTablaPdf With {.Ancho = 60})

' Completar celdas en orden. Si queres reservar, usa AgregarVacios.
datos.AgregarVacios(4)
datos.Agregar("Fecha")
datos.Agregar(operacion.fechaInicio.ToShortDateString())

tabla.Datos = datos
hoja = tabla.Escribir()
y = tabla.HojaSetup.curY
```

### Notas de paginacion (muy importante)
- `TablaPdf.Escribir()` puede generar nuevas hojas si no hay espacio (`HojaSetup.HayEspacio` / `HojaSetup.SaltoDePagina`).
- Si el codigo llamador va a seguir dibujando despues de una tabla, debe seguir usando la hoja devuelta por `Escribir()` (o `tabla.Hojas.Last`).
- Si creas una tabla con `New TablaPdf(hoja)` (sin pasar `ArmadoPdf`), el propio comentario del codigo advierte que si hay saltos y el llamador no los maneja, las paginas siguientes pueden quedar en un `ArmadoPdf` temporal.

## 4) `ObtenerRecuadro` / `UnirCeldas`: como dibujar contenido custom dentro de celdas

### `ObtenerRecuadro`
`TablaPdf` expone:
- `ObtenerRecuadro(fila, columna) As areaYContenedorPdf`
- `ObtenerRecuadroFila(fila) As areaYContenedorPdf`
- `ObtenerRecuadro(filaInicio, colInicio, filaFin, colFin) As areaYContenedorPdf`

Uso:
- Primero renderizas la tabla (o al menos llamas `Recalcular()`), y despues pedis el area de una celda o bloque.
- Si el bloque cruza paginas, la funcion lanza exception: el area debe pertenecer a una sola hoja.

### `UnirCeldas`
`UnirCeldas` sirve para:
- Calcular el area unida.
- Pintar el fondo (agrega un `rectanguloPdf` al `cuerpoRectanguloPdf` de la hoja).
- Opcionalmente escribir texto dentro del area (si pasas `contenido`).

Patron:
1. Renderizar tabla.
2. `tabla.UnirCeldas(...)`.
3. `area = tabla.ObtenerRecuadro(...)`.
4. Agregar tu contenido custom a `area.hoja`.

Usos comunes de este mecanismo:
1. Encapsular secciones: definir `Sub dibujarX(area As areaYContenedorPdf)` y dentro dibujar fondo, titulo y contenido usando `area.hoja` y `area.rectangulo`.
2. Mapa de layout: guardar `tabla.ObtenerRecuadro(...).rectangulo` en un diccionario (clave semantica -> `rectanguloPdf`) para luego recorrer y dibujar bordes/fondos o ubicar texto.
3. Posicionamiento relativo: si ya tenes un `rectanguloPdf` base, usar `contiguoX(n)` / `contiguoY(n)` para mover el mismo rectangulo por su propio ancho/alto.
4. Uso utilitario: si solo necesitas `TablaPdf.renglonEnRecuadro(...)` como helper, podes instanciar una `TablaPdf` temporal y usar el overload que recibe `rectanguloPdf`/`hojaPdf`.

## 5) `hojaPdfSetup` (margenes, tamanio, cursores)
`hojaPdfSetup` se usa para mantener coordenadas y saber si entra mas contenido:
- `curX`, `curY`: cursor actual.
- `HayEspacio(curY, espacioRequerido)`: decide si entra una fila/bloque.
- `SaltoDePagina(pdf, hojaActual)`: crea una hoja nueva y resetea cursores.

Regla:
- Para tablas, `HojaSetup.curY` es el punto de arranque en Y (arriba del proximo bloque), y se va decrementando.

## 6) `DatosTabla` (como cargar datos)
`DatosTabla` implementa `iDatosTablaPdf` y es lo mas simple para alimentar una tabla.

Puntos:
- Cada `Agregar(...)` suma una celda (no una fila).
- La cantidad de celdas define cuantas filas hay via `RowCount(cantidadDeColumnas)`.
- `AgregarVacios(n)` es util para reservar lugares.
- Si queres estilos por celda, usa `Agregar(texto, estilo)`.

## 7) Helpers: `Toolbox01`
`Toolbox01` dibuja dentro de un `areaYContenedorPdf`:
- `Checkbox01(contenedor, checked, checkedChar := "X", tipoDeMargenes := margenMinimo)`
- `Cinta01(contenedor, tipoDeMargenes := margenMinimo)`

Regla:
- Estas funciones agregan shapes/texto directamente a `contenedor.hoja`, asi que funcionan bien incluso si el area esta en otra pagina.

Uso comun tipo formulario:
1. Calcular una grilla con `TablaPdf`.
2. Tildar celdas puntuales con `toolbox.Checkbox01(tabla.ObtenerRecuadro(fila, columna), condicion)`.

## 8) Extension method: crear tabla dentro de un area
`areaYContenedorPdfExtensions.CrearTablaEnBaseA(area, tablaDeReferencia, cantidadDeColumnas)`:
- Crea una `TablaPdf` dentro del rectangulo `area`.
- Copia configuracion base (setup/estilos/altos) desde `tablaDeReferencia`.
- Distribuye el ancho 100% entre `cantidadDeColumnas`.

Uso recomendado para un agente:
- Tenes una tabla "template" ya configurada.
- Necesitas una subtabla dentro de una celda/recuadro: pedis el `areaYContenedorPdf` y creas una tabla nueva en ese area sin recalcular todo manual.

Usos comunes:
1. Subtablas para listas: tomar un recuadro de varias filas/columnas y crear una tabla chica adentro para cargar una lista.
2. Subtablas para pares clave/valor: crear una tabla de 2 columnas dentro de un bloque y escribir ahi.
3. Ajuste posterior: despues de crear la subtabla, setear `DatosBordes`, `DatosEstilo` y ajustar `Columnas(i).Ancho` antes de `Escribir()`.