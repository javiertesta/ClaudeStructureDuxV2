# Sistema Dux/Fux - Información General

## Características generales de los repos de las versiones 2 de los sistemas de la empresa
- ASP.NET Web Forms.
- Target .NET Framework 4.7.2.
- VB.NET
- VS 2022 (probando VS 2026)

## Definiciones
- Cada sistema se compone de "puntos del sistema" o "programas".
- Cada "punto del sistema" o "programa" está contenido en una carpeta repo de Tortoise SVN.
- Localmente todos los repos se ubican en "/mnt/d/Desarrollo".
- Cada repo contiene a su vez una subcarpeta con un proyecto de DLL de negocio identificada con la letra N, y una subcarpeta identificada con la letra P
  que contiene el "programa en sí", un sitio web con el front. Si bien cada punto del sistema tiene su proyecto de negocio, a veces el "programa" (o sitio
  web del front) contiene código de negocio. Esto no es deseable pero sucede.
- La funcionalidad de base se maneja desde la biblioteca entidades.dll. El proyecto de VS se llama Clases.
- El sistema se configura en cada cliente con una serie de elementos, de los cuales dos importantes son:
  - La tabla "sympaa", que contiene registros tipo nro, detalle y valor, y se lee mediante Factory.getInstance().getParametro(nroParametro)
  - El archivo parametrizacion.xml, que se encuentra en la carpeta raíz del sitio.
- Cuando quiero comenzar a trabajar con un pedido que está asociado a un repo, tenés que resolver el problema en ese repo y no en otro, excepto que en VS
  esté cargado el proyecto de otro repo, o copiados sus archivos .aspx y .aspx.vb. No empieces a buscar cosas en /mnt/d o /mnt/d/Desarrollo sin tener en cuenta esto.

## Acerca del sistema
Dux y Fux son dos sistemas pertenecientes a la misma empresa que se encargan de gestionar cuestiones y trámites de despachantes de aduanas (en el caso de Dux)
y forwarders (en el caso de Fux). Dentro del ecosistema hay otros sistemas de la misma empresa, pero no vienen al caso nombrar. Todos usan una base de código común que
por lo general está contenida dentro del proyecto de DLL llamado Clases (espacio de nombres "entidades" con salida a "entidades.dll") y en lo que llamamos
"Plantilla", que es un repo especial que bajamos y usamos para copiar a cada "repo de programa" el resto de los archivos del sitio web -comunes-
que completan la estructura para poder compilar y ejecutar localmente. Para que se entienda: el sitio web productivo contiene absolutamente todos los programas,
pero cada carpeta P del "repo de programa" contiene inicialmente solo las páginas del mismo -salvo excepciones- y muy pocos archivos comunes.
Inicialmente casi no tiene páginas de otros programas salvo excepciones, o salvo que hayan sido agregadas posteriormente y de manera manual por algún desarrollador.
Por todo lo dicho, cuando te encuentres con un repo ya bajado, muy probablemente vas a ver en la carpeta P un sitio web funcional y ejecutable.
Si lo ves incompleto, es probable que le falte copiar la plantilla.

Las versiones 2 del sistema están en su etapa final. Como tiene años en el mercado, vas a encontrar diversas maneras de
encarar las cosas. Podés usar Tortoise SVN para Windows para revertir cambios, pero consultame antes de accionar al respecto.
Nunca hagas commits a menos que te lo pida muy expresamente.

## Para comenzar a trabajar sobre un pedido

**IMPORTANTE: Antes de analizar el código del pedido, PRIMERO modificar Factory.vb según se indica abajo.**

Si te pido que analices un pedido para empezar a trabajar sobre él, lo que tenés que hacer es lo siguiente:
- Te tengo que dar la siguiente información. Si no te doy algo de lo siguiente, me lo preguntás explícitamente:
  - 1. Número de pedido.
  - 2. Número de cliente para el cual se está realizando el pedido (el cliente nuestro; no el usuario que opera ni algún cliente de nuestro cliente).
  - 3. Texto del pedido.
  - 4. Vas a encontrar imágenes que se adjuntaron al pedido en "/mnt/d" con nombre en formato "capturaXXpedidoXXXXX"
- Tenés que indicarme cuál es la zona del código donde tenemos que actuar.
- Tenés que preguntarme, siempre que sea posible, los valores de parámetros que encontrás en el camino (acordate de la función getParametro, en Factory.vb).
  Necesitamos saber cuáles son esos valores en la base del cliente porque los que pasa la función getParametro por lo general son los de alguna base de testing.
  Lo que hacemos es editar la función getParametro y pisar los valores que se traen desde la BD (que probablemente no sea la de producción del cliente) con los
  valores que realmente tiene la base del cliente.
- Con lo anterior, tenés que modificar la función getParametro(ByVal id As Long) que está en Factory.vb (dentro del repo Clases, en la subcarpeta clases/Factories).
  Fijate que ya tiene otros ejemplos de otros pedidos con sus parámetros. Hay un Case que deriva según número de pedido, y otro Case que deriva por número de parámetro.
  Armá la estructura correspondiente para agregarla donde corresponda con el fin de pisar los valores de testing con los del cliente.
  - El parámetro 0 colocalo siempre con el valor del número de cliente. Aunque no se use en el código, ponelo.
  - La estructura que tenés que modificar es la que tiene la forma:
  			Dim idPedido As Integer = [Número de pedido sobre el cual estamos trabajando]
				Select Case idPedido
				  Case [Número de pedido sobre el cuál estamos trabajando] ' [Comentario mínimo indicador de lo que hace el pedido, de muy pocas palabras, muy conciso]
						Select Case parametro.nro
							Case 0
								parametro.valor = "[Número de cliente]"
							Case [Parámetro 1]
								parametro.valor = "[Valor del parámetro 1]"
						End Select
				End Select