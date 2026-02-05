# ABC522 - Facturación

## Descripción
ABC522 es el programa que se encarga de la facturación. Forma parte de los sistemas Dux y Fux.

## Architecture

### Solution Structure

- **ABC522N\** - Business logic library (VB.NET class library producing ABC522N.dll)
- **ABC522P\** - Web presentation layer (ASP.NET Web Forms website)

### Key Dependencies (External Projects)

The solution references these projects from parent directories:
- `..\Clases\Clases.vbproj` - Shared entity classes (`entidades` namespace)
  Tener en cuenta que dentro del repo `Clases`, ubicado en `..\Clases` hay una subcarpeta que también se llama `clases`.
  `..\Clases\clases` es además importante; contiene gran parte del contenido del repo.

External DLLs from `..\..\DLLS\Sistema\`:
- `entidades.dll` - Core entity classes with `entidades.Factories.Factory` singleton
- `itextsharp.dll` - PDF generation
- `Newtonsoft.Json.dll` - JSON serialization

### Business Logic Layer (ABC522N)

Files follow numeric naming convention `ABC522N###.vb`

### Presentation Layer (ABC522P)

- **MasterPage.master** / **MasterPage3.master** / **ResponsiveMasterPage.master** - Page templates
- **App_Code\paginaBase.vb** - Base page class inheriting from `globalizacion`, provides session handling
- **controles\** - Reusable ASCX user controls (modals, menus, input helpers)
- **Bin\** - Compiled DLLs from the entire Dux system

Page naming convention: `[MODULE][NUMBER]P[PAGE].aspx` (e.g., `ABC902P002.aspx`, `ABC522P001.aspx`)

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
