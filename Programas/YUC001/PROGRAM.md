# YUC001 - Seguimiento de operaciones

## Descripción
YUC001 presenta una consulta con listado de operaciones de comercio exterior. Forma parte del sistema Fux. Tiene una pantalla popup grande donde se aplican filtros y se seleccionan las columnas a mostrar.
Por otro lado, tiene la posibilidad de grabar una determinada consulta en lo que llamamos "plantillas". El usuario puede cargar una plantilla para mostrar inmediatamente la consulta, y grabarla tanto
de manera privada (para el usuario únicamente) o pública (para todos los usuarios del sistema) Un programa importante para YUC001, que contiene toda la parte de la consulta SQL, es YMA127.

## Build Commands

Build the solution using Visual Studio or MSBuild:
```bash
msbuild YUC001.sln /p:Configuration=Debug
```

## Architecture

### Solution Structure

- **YUC001N/** - Business logic library (VB.NET class library producing YUC001N.dll)
- **YUC001P/** - Web presentation layer (ASP.NET Web Forms website)

### Key Dependencies (External Projects)

The solution references these projects from parent directories:
- `../Clases/Clases.vbproj` - Shared entity classes (`entidades` namespace)
  Tener en cuenta que dentro del repo `Clases`, ubicado en `../Clases` hay una subcarpeta que también se llama `clases`.
  `../Clases/clases` es además importante; contiene gran parte del contenido del repo.

External DLLs from `../../DLLS/Sistema/`:
- `entidades.dll` - Core entity classes with `entidades.Factories.Factory` singleton
- `itextsharp.dll` - PDF generation
- `Newtonsoft.Json.dll` - JSON serialization

### Business Logic Layer (YNC001N)

Files follow numeric naming convention `YUC001N###.vb`

### Presentation Layer (YUC001P)

- **MasterPage.master** / **MasterPage3.master** / **ResponsiveMasterPage.master** - Page templates
- **App_Code/paginaBase.vb** - Base page class inheriting from `globalizacion`, provides session handling
- **controles/** - Reusable ASCX user controls (modals, menus, input helpers)
- **Bin/** - Compiled DLLs from the entire Dux system

Page naming convention: `[MODULE][NUMBER]P[PAGE].aspx` (e.g., `ABC902P002.aspx`, `YUC001P001.aspx`)

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
