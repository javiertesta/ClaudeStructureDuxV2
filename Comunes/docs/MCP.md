# MCP - ClaudeMcpHost.exe

## Principio
**Modificar archivos existentes SIEMPRE por unified diff. Sin excepciones.**
**ClaudeMcpHost.exe lo vas a encontrar ubicado en /mnt/d/Desarrollo**

Este archivo se lee junto con **CLAUDE.md** (ahí van las reglas de “cómo generar” el diff). Acá queda el **contrato** del MCP.

## Comandos
**Leer archivo**
- `ClaudeMcpHost.exe read "<ruta>"`
  - Devuelve contenido + `-----HASH-----` (tolerante, usar este) + `-----HASH-STRICT-----` (bytes).
  - Si hay U+FFFD o U+FEFF → WARNING/ERROR: **parar**.

**Leer rango (para copiar literal)**
- `ClaudeMcpHost.exe read-range "<ruta>" <ini> <fin>`
  - Devuelve líneas numeradas + hash. Usalo para sacar contexto exacto.

**Aplicar patch**
- `ClaudeMcpHost.exe apply-patch "<ruta-archivo-destino>" <hash> "<ruta-al-archivo.diff>"`
- Para diffs grandes: `--large` (solo con autorización explícita).

⚠️ **NUNCA** pasar el contenido del diff como argumento de línea de comandos.
El tercer parámetro es una **RUTA a un archivo .diff**, NO el contenido del diff.

## Reglas del diff (innegociables)
1. **Formato**: unified diff git-style con hunks `@@ -X,Y +X,Y @@`
2. **Prefijos**: cada línea del hunk empieza con ` ` / `-` / `+`
3. **Contexto literal**: líneas ` ` y `-` deben coincidir **tal cual** con el archivo (mismos tabs/espacios).
4. **NO partir líneas**: si una línea tiene excesivos chars, va completa. **Nunca wrap.**
5. **Cambio mínimo**: no refactor, no "limpieza", no reindent fuera del cambio.
6. **Hunks mínimos**: 1 cambio por hunk, 2–6 líneas reales de contexto, copiadas del `read-range`.
7. **El hash tiene que ser completo (64 hex). No usar el hash 'truncado con …'.**
8. **Si querés tolerancia a espacios/tabs, usar el hash de -----HASH----- (whitespace-normalized), no necesariamente el -----HASH-STRICT-----.**

> Nota: el MCP normaliza LF/CRLF internamente y re-escribe con el newline original. No uses unix2dos/sed/printf.

## Flujo obligatorio
1) `read` (o `read-range`) → obtener hash / contexto real
2) Crear archivo .diff con Write tool (ej. `Write("/mnt/d/Desarrollo/patch.diff")`)
3) `apply-patch "<archivo-destino>" <hash> "<ruta-patch.diff>"`
4) Opcionalmente borrar el archivo .diff después

## Errores: qué hacer
- **Hash no coincide** → re-`read` y usar el hash nuevo.
- **Contexto / línea a eliminar no coincide** → `read-range` del bloque y regenerar diff **copiando literal** (tabs, sin wrap).
- **U+FFFD / U+FEFF** → **PARAR. NO REINTENTAR.** Ir a modo manual.
- **Diff grande** → pedir autorización para `--large` o dividir en varios patches. Si se elige esta opción, preguntar al usuario por cada uno.

## Modo manual (cuando no se puede aplicar)
1) No escribir.  
2) `read-range` del bloque exacto.  
3) Explicar “línea por línea” qué se cambia.  
4) El usuario aplica manualmente.

## Fuera del MCP
- Crear / borrar / renombrar / mover archivos: mecanismo estándar del modelo.
- **Modificar existente**: **MCP obligatorio**.

Reglas para mantener la fortaleza:
- Buscar coincidencia dentro de una **ventana acotada** (ej. ±300 líneas; opcional fallback global).
- Aceptar SOLO si hay **una coincidencia única**. Si hay 2+, **rechazar** (evita aplicar en lugar equivocado).
- “Fuzz” controlado: recortar máx. 1–2 líneas de **contexto** (` `) al inicio/fin del hunk.
  - **Nunca** fuzz sobre líneas `-` ni `+`.
- Mantener límites existentes (p.ej. touched-lines / tamaño / --large).