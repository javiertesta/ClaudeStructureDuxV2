# FFA001 - Clientes

## Descripción
FFA001 es un ABM que se encarga de la gestión de clientes. Forma parte de los sistemas Dux y Fux.

## Architecture

### Solution Structure

- **FFA001N\** - Business logic library (VB.NET class library producing FFA001N.dll)
- **FFA001P\** - Web presentation layer (ASP.NET Web Forms website)

### Key Dependencies (External Projects)

The solution references these projects from parent directories:
- `..\Clases\Clases.vbproj` - Shared entity classes (`entidades` namespace)
Tener en cuenta que dentro del repo `Clases`, ubicado en `..\Clases`, hay una subcarpeta que también se llama `clases`.
  
External DLLs from `..\..\DLLS\Sistema\`:
- `entidades.dll` - Core entity classes (`entidades` namespace)
- `Newtonsoft.Json.dll` - JSON serialization

### Business Logic Layer (FFA001N)

Files follow numeric naming convention `FFA001N###.vb`

### Presentation Layer (FFA001P)

- **MasterPage.master** / **MasterPage3.master** / **ResponsiveMasterPage.master** - Page templates
- **App_Code\paginaBase.vb** - Base page class inheriting from `globalizacion`, provides session handling
- **controles\** - Reusable ASCX user controls (modals, menus, input helpers)
- **Bin\** - Compiled DLLs from the entire Dux system

Page naming convention: `[MODULE][NUMBER]P[PAGE].aspx` (e.g., `ABC902P002.aspx`, `FFA001P001.aspx`)

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
