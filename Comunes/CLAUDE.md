# CLAUDE.md
Este archivo contiene las instrucciones principales para trabajar en este repositorio con Claude y/o Codex CLI (si este archivo se usa como doc de instrucciones/fallback).

## Reglas fundamentales
1. **Commits**: NUNCA hacer commits sin autorización explícita.
2. **Edición de archivos**: Prohibido modificar archivos con herramientas por fuera del uso del MCP.
3. **Aplicación de patches (file.apply_patch_only)**:
     - El parchador interno es `patch.exe`, de Git for Windows.
     - La tool `file.apply_patch_only` espera **unified diff puro** (estilo git):
       - Encabezados `---` / `+++`
       - Hunks `@@ -start[,count] +start[,count] @@`
       - Líneas de hunk con prefijo: `' '` (contexto), `'+'` (alta), `'-'` (baja)
     - **No usar** wrappers tipo:
       - `*** Begin Patch`
       - `*** Update File`
       - `*** End Patch`
       (ese formato corresponde a otra herramienta, no a `file.apply_patch_only`).
     - Mantener contexto exacto (tabs/espacios) en líneas de contexto y borrado.
     - No partir líneas largas manualmente dentro del diff.
     - El diff debe terminar con newline (`\n`).
4. Por fuera del MCP sí está permitido crear archivos desde cero, copiarlos, moverlos, o eliminarlos.

## Herramientas de compilación
- Compilar con `C:\Program Files\Microsoft Visual Studio\18\Community\MSBuild\Current\Bin\amd64\MSBuild.exe`
- Usar `-` en lugar de `/` para los flags (Git Bash convierte `/flag` a rutas Windows)
- Compilar de a un proyecto por llamada.

## Documentación
- **docs** - Información específica del repo que se debe leer obligadamente.

## Sistema Dux/Fux - Información general

### Características generales de los repos de las versiones 2 de los sistemas de la empresa
- ASP.NET Web Forms.
- .NET Framework 4.7.2.
- VB.NET
- VS 2026
- MySQL
- Tortoise SVN

### Definiciones
- Cada sistema se compone de "puntos del sistema" o "programas".
- Cada "punto del sistema" o "programa" es un repo SVN.
- Localmente todos los repos de los puntos del sistema se ubican en "D:\Desarrollo".
- Si bien cada punto del sistema tiene su proyecto de negocio y así se refleja en el contenido de la solución de VS, a veces la misma contiene archivos y referencias a proyectos de otros repos.
- La funcionalidad de base, mapeado de entidades y funciones básicas, se manejan desde la biblioteca entidades.dll. El proyecto de VS se llama Clases y el espacio de nombres es "entidades".
- Cada cliente se parametriza de dos modos, entre otros de menor importancia:
  - La tabla "sympaa", que contiene registros tipo nro, detalle y valor, y se lee mediante Factory.getInstance().getParametro(nroParametro)
  - En menor medida el archivo parametrizacion.xml, que se encuentra en la carpeta raíz del sitio web.

### Entidades / Estructura de capas / Funcionalidad básica
- A. Obtención de SQLs y mapeos: mapper.vb/mapperFux.vb. Por lo general son funciones Friend.
- B. Capa pública del manejo de entidades: Factory.vb/FactoryFux.vb.
- C. Kernel de negocio: Funcionalidad.vb/FuncionalidadFux.vb.
- D. Repo del programa -> Negocio: Proyecto DLL VB, nombrado como un código que contiene la letra N.
- E. Repo del programa -> Front (el programa en sí): Sitio web, nombrado como un código que contiene la letra P.

- Las funciones que comienzan con mapperQuery[...] generan los SQL para cada entidad.
- La función fillObject pasa un DataRow a un Object (Friend)
- Las funciones que comienzan con fill[...] completan un objeto de una clase específica con el DataRow obtenido desde la base. (Friend)
- La función get[...] devuelve una entidad específica desde la BD (Pública)
- La función fillListObject pasa un DataSet a una lista. (mapper / Es Friend)
- La función getListObject pasa un DataSet a una lista (Factory / Pública)
- Funcionalidad clase con la primera capa de negocio, que incluye las funciones más abarcativas.
- En caso de ser necesario y si no hay nada predefinido, se puede consultar la base mediante Factory.getInstance().getQuery().

- Los datos desde la base se leen con DBFunctions.DBToObject().
- Los datos desde que se van a persistir en la base se escriben con DBFunctions.objectToDB().
- El conjunto fecha/hora que viene del front se concetena con Funciones.concatenarFechaYHora().
- Las fechas se formatean de manera estadarizada con Funciones.formatearFecha().
- Los decimales se formatean a través de Funciones.formatearDecimales().
- Disponemos de Funciones.devolverCurrentCultureEstandarDeDux().
- La operación es la principal entidad del sistema. Dispone de soyImportacion() y soyExportacion().

- Las entidades herendan por lo general de "base". Una manera simple te gestionarlas es a través de:
  - Factory.getInstance().marcarParaAlta()
  - Factory.getInstance().marcarParaBorrar()
  - Factory.getInstance().marcarParaBorrarFisico()
  - Factory.getInstance().marcarParaReplace()
  - Se persiste con Factory.getInstance().persistir()

### Acerca del sistema
Dux y Fux son dos sistemas pertenecientes a la misma empresa que se encargan de gestionar cuestiones y trámites de despachantes de aduanas (en el caso de Dux)
y forwarders (en el caso de Fux). Dentro del ecosistema hay otros sistemas de la misma empresa, pero no vienen al caso nombrar. Todos usan una base de código común que
por lo general está contenida dentro del proyecto de DLL llamado Clases (espacio de nombres "entidades" con salida a "entidades.dll") y en lo que llamamos
"Plantilla", que es un repo especial que bajamos y usamos para copiar a cada "repo de programa" el resto de los archivos del sitio web -comunes-
que completan la estructura para poder compilar y ejecutar localmente. Para que se entienda: el sitio web productivo contiene absolutamente todos los programas,
pero cada carpeta P del "repo de programa" contiene inicialmente solo las páginas del mismo -salvo excepciones- y muy pocos archivos comunes.
Inicialmente casi no tiene páginas de otros programas salvo excepciones, o salvo que hayan sido agregadas posteriormente y de manera manual por algún desarrollador.
Por todo lo dicho, cuando te encuentres con un repo ya bajado, muy probablemente vas a ver en la carpeta P un sitio web funcional y ejecutable.
Si lo ves incompleto, es probable que le falte copiar la plantilla.

Las versiones 2 del sistema están en su etapa final. Como tiene años en el mercado vas a encontrar diversas maneras de
encarar las cosas. Podés usar SVN para Windows para revertir cambios pero consultame antes de hacerlo.
Nunca hagas commits.