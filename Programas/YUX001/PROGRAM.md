# YUX001 - Seguimiento de operaciones

## Descripción
YUX001 gestiona las cotizaciones del sistema Fux. Contiene YUX001P001, una pantalla de consulta con el listado de las cotizaciones de comercio exterior,
y contiene YUX001P002, un ABM para gestionar cada cotización. La pantalla del listado tiene una pantalla popup grande donde se aplican filtros
y se seleccionan las columnas a mostrar. Por otro lado, tiene la posibilidad de grabar una determinada consulta en lo que se denominan "plantillas".
El usuario puede cargar una plantilla para mostrar inmediatamente la consulta, y grabarla la estructura de la consulta tanto de manera privada
(para el usuario únicamente) como pública (para todos los usuarios del sistema)

## Build Commands

Build the solution using Visual Studio or MSBuild:
```bash
msbuild YUX001.sln /p:Configuration=Debug
```

## Architecture

### Solution Structure

- **YUX001N\** - Business logic library (VB.NET class library producing YUX001N.dll)
- **YUX001P\** - Web presentation layer (ASP.NET Web Forms website)

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

Files follow numeric naming convention `YUX001N###.vb`

### Presentation Layer (YUX001P)

- **MasterPage.master** / **MasterPage3.master** / **ResponsiveMasterPage.master** - Page templates
- **App_Code\paginaBase.vb** - Base page class inheriting from `globalizacion`, provides session handling
- **controles\** - Reusable ASCX user controls (modals, menus, input helpers)
- **Bin\** - Compiled DLLs from the entire Dux system

Page naming convention: `[MODULE][NUMBER]P[PAGE].aspx` (e.g., `ABC902P002.aspx`, `YUX001P001.aspx`)

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
- Date formatting via `Funciones.formatearFecha()`, `Funciones.toDate()`
