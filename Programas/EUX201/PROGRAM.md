# EUX201 - ABM de operaciones HCME (House Carga Marítima de Exportación)

## Descripción
EUX201 es uno de los tantos ABMs de operaciones de comercio exterior para forwarders. Específicamente este ABM es para cargas house (consolidadas o cargas directas) 
marítimas de exportación. Forma parte del sistema Fux, nuestro sistema de gestión para forwarders.

Este repo EUX201 no lo vamos a usar en este caso para resolver ningún pedido de la empresa sino para crear nuevas clases que generen de mejor manera las impresiones
de BLs de las operaciones.

Tenés que estar atento a los archivos ArmadoPdf.vb, ArmadoPdfTablas.vb, ArmadoPdfFormularios.vb, ArmadoPdfFunciones.vb y BL00390a. La cosa es así:
- ArmadoPdf se monta sobre itextsharp y es la base sobre la cual se monta casi todo el resto. Igualmente en las nuevas clases quizás hagamos consultas directas
  a las clases de itextsharp.
- ArmadoPdfTablas se monta sobre ArmadoPdf para generar tablas de manera más fácil. No tiene que ver tanto con las nuevas clases para BLs, pero lo comento porque forma
  parte del ecosistema de PDFs.
- ArmadoPdfFormularios contiene las clases de la nueva funcionalidad, y la que quiero terminar de desarrollar.
- ArmadoPdfFunciones contiene funciones helper, mayormente para estas nuevas clases de BLs.
- También vas a ver BL00390a.vb que es el archivo que contiene el BL específicamente el cliente 00390. Como es el primer formulario que se creó para ese cliente,
  tiene la letra a.

Lo que busco es crear un sistema de clases que represente un "formulario PDF" sobre las cuales vamos a desarrollar clases que representen "BLs", y sobre las cuales
creemos BLs para distintos clientes (BL00390a.vb es un ejemplo)
las clases de formularios tienen que manejar de manera exacta las dimensiones de cada "campo" que va a contener el formulario, sabiendo alinear horizontalmente
(no me acuerdo si vertical también) y sabiendo escribir un campo a través de varias páginas.
Por el momento el contenido de cada campo va a ser texto, o simplemente la gestión del espacio: devolver el espacio para dárselo a una tercera función que
gestione su contenido. Tiene que tener también la posibilidad de que esa tercera función le pida a esta clase de formularios un "salto de página",
en cuyo caso la funcionalidad de formularios tiene que asignar un nuevo espacio en la página siguiente y devolverlo para que la tercera función trabaje.
Dejé varias cosas encaradas en este trabajo, y me acuerdo que estaba pensando en pensar bien la estructura correcta para que a futuro no me dé problemas.
Estaba pensando qué clase es la que debe tener la funcionalidad de escribir en el campo, si el mismo campo, o la función "ejecutar" perteneciente a las clases
de formularios.
¿Me dijeron que tenés un modo de trabajo que se llama "plan" o algo así? ¿Me podés dar un panorama de lo que ves al recorrer las clases y la estructura existente?

## Build Commands

Build the solution using Visual Studio or MSBuild:
```bash
msbuild EUX201.sln /p:Configuration=Debug
```

## Architecture

### Solution Structure

- **EUX201N\** - Business logic library (VB.NET class library producing EUX201N.dll)
- **EUX201P\** - Web presentation layer (ASP.NET Web Forms website)

### Key Dependencies (External Projects)

The solution references these projects from parent directories:
- `..\Clases\Clases.vbproj` - Shared entity classes (`entidades` namespace)
  Tener en cuenta que dentro del repo `Clases`, ubicado en `..\Clases` hay una subcarpeta que también se llama `clases`.
  `..\Clases\clases` es además importante; contiene gran parte del contenido del repo.

External DLLs from `..\..\DLLS\Sistema\`:
- `entidades.dll` - Core entity classes with `entidades.Factories.Factory` singleton
- `itextsharp.dll` - PDF generation
- `Newtonsoft.Json.dll` - JSON serialization

### Business Logic Layer (YNC001N)

Files follow numeric naming convention `EUX201N###.vb`

### Presentation Layer (EUX201P)

- **MasterPage.master** / **MasterPage3.master** / **ResponsiveMasterPage.master** - Page templates
- **App_Code\paginaBase.vb** - Base page class inheriting from `globalizacion`, provides session handling
- **controles\** - Reusable ASCX user controls (modals, menus, input helpers)
- **Bin\** - Compiled DLLs from the entire Dux system

Page naming convention: `[MODULE][NUMBER]P[PAGE].aspx` (e.g., `ABC902P002.aspx`, `EUX201P001.aspx`)

### Session Management

Pages use `utilidadSession.miSession` (shadowed `Session` property) instead of standard ASP.NET session.

### Database Access

Uses `entidades.Factories.Factory.getInstance()` singleton pattern:
- `Factory.getInstance.getListObject(Type, whereClause)` - Query entities
- `Factory.getInstance.getOperacion(numero)` - Get specific operation
- `Funciones.objectToDB()` - SQL value formatting

### Configuration

- `Web.config` - IIS configuration with Windows authentication
- `parametrizacion.xml` - Application parameters for each specific client
- `MenuMymtec*.xml` - Navigation menu definitions

## Code Conventions

- Spanish variable/method names throughout
- Import/export operations: "Impo" (importacion) / "Expo" (exportacion)
- Report classes named by client ID: `Cliente#####TapaCarpeta[Impo|Expo]_[ClientName]`
- Date formatting via `Funciones.formatearFecha()`, `Funciones.toDate()`
