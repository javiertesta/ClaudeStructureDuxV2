# YNC001 - Tapas de Carpeta

## Descripción
YNC001 devuelve PDFs con tapas de carpeta de las operaciones de comercio exterior solicitadas. Una tapa de carpeta es una impresión de una página con la información
básica de una operación. Puede variar según cada cliente. Hay tapas genéricas y tapas personalizadas para los clientes que las solicitan.

## Build Commands

Build the solution using Visual Studio or MSBuild:
```bash
msbuild YNC001.sln /p:Configuration=Debug
```

## Architecture

### Solution Structure

- **YNC001N/** - Business logic library (VB.NET class library producing YNC001N.dll)
- **YNC001P/** - Web presentation layer (ASP.NET Web Forms website)

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

Files follow numeric naming convention `YNC001N###.vb`:
- `YNC001N001.vb` - Report listing and PDF concatenation logic
- Other files contain client-specific report implementations (e.g., `Cliente00001TapaCarpetaImpo_AresEnderiz`)

Key patterns:
- Client-specific report classes for different customs brokers

### Presentation Layer (YNC001P)

- **MasterPage.master** / **MasterPage3.master** / **ResponsiveMasterPage.master** - Page templates
- **App_Code/paginaBase.vb** - Base page class inheriting from `globalizacion`, provides session handling
- **controles/** - Reusable ASCX user controls (modals, menus, input helpers)
- **Bin/** - Compiled DLLs from the entire Dux system

Page naming convention: `[MODULE][NUMBER]P[PAGE].aspx` (e.g., `ABC902P002.aspx`, `YNC001P001.aspx`)

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
