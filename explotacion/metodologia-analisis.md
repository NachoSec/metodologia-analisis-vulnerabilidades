Explotación de vulnerabilidad en FreeFloat FTP Server
---
Introducción
---
Una vez preparado el entorno de laboratorio y analizadas las aplicaciones vulnerables, el siguiente paso consiste en realizar el proceso completo de explotación de una vulnerabilidad presente en FreeFloat FTP Server.

El objetivo de esta fase es comprender cómo un atacante puede identificar un buffer overflow, controlar el flujo de ejecución del programa y redirigir la ejecución hacia código controlado por el atacante.

Durante este proceso se utilizan herramientas de análisis y depuración que permiten observar el comportamiento interno del programa mientras se envían datos maliciosos al servicio.

Entre las herramientas utilizadas destacan:

Immunity Debugger

IDA Pro

Mona.py

Scripts de explotación en Python

Estas herramientas permiten analizar el binario, observar el estado de la memoria del proceso y desarrollar un exploit paso a paso.


Análisis del binario vulnerable
---
Antes de comenzar la explotación es necesario analizar el binario del servidor FTP para comprender cómo procesa las peticiones que recibe.

Para ello se utiliza IDA Pro, una herramienta de ingeniería inversa que permite analizar el código del programa.

Dentro de IDA se abre el ejecutable FTPServer.exe y se accede a la vista de cadenas del programa mediante:

View → Open Subviews → Strings

Esta vista permite observar todas las cadenas de texto utilizadas por el programa.

Posteriormente se realiza una búsqueda de la cadena:

USER

Este comando forma parte del protocolo FTP y está relacionado con el proceso de autenticación del servidor.

Al localizar la cadena se utilizan las referencias cruzadas (Xrefs) para identificar qué función del programa utiliza esa cadena.

El análisis muestra que la cadena es utilizada dentro de la función ubicada en la dirección:

402910

Al abrir esta función en el pseudocódigo es posible observar cómo el programa procesa los comandos enviados por el cliente FTP.

Este análisis permite identificar el punto del programa donde posteriormente se producirá la vulnerabilidad.

Prueba de comunicación con el servidor
---
Una vez identificado el funcionamiento básico del servidor se procede a comprobar que es posible interactuar con él mediante scripts desarrollados para el laboratorio.

Para ello se ejecuta el primer script de Python, cuyo objetivo es establecer una conexión con el servidor FTP utilizando las credenciales:

USER anonymous
PASS anonymous

El script también intenta ejecutar el comando:

NOOP

Este comando no es procesado por el servidor debido a que no existe un handler que lo gestione dentro del código analizado.

Sin embargo, el servidor responde correctamente a la conexión, lo que confirma que es posible establecer comunicación con el servicio y enviar comandos al programa vulnerable.

Fuzzing del servicio
---
El siguiente paso consiste en realizar fuzzing sobre el servidor.

El fuzzing es una técnica utilizada en seguridad ofensiva que consiste en enviar grandes cantidades de datos a una aplicación con el objetivo de provocar errores en su ejecución.

En este caso el script comienza enviando buffers cada vez más grandes compuestos por caracteres repetidos.

El tamaño del buffer aumenta progresivamente:

100 bytes
200 bytes
300 bytes
400 bytes

Tras enviar una cantidad suficiente de datos, el programa deja de funcionar correctamente y se produce un fallo en memoria.

En Immunity Debugger se puede observar que el registro EIP ha sido sobrescrito con valores correspondientes al carácter A, lo que indica que el buffer enviado ha sobrepasado el tamaño esperado por el programa.

Este comportamiento confirma la existencia de una vulnerabilidad de buffer overflow.

Generación de un patrón cíclico
---
Una vez confirmada la vulnerabilidad es necesario determinar exactamente cuántos bytes son necesarios para sobrescribir el registro EIP.

Para ello se utiliza la herramienta Mona, integrada en Immunity Debugger.

Creamos un folder que se llame mona en C:\ 

!mona config -set workingfolder C:\Mona con esto conseguiremos que los archivos que se creen se dirigan a la carpeta creada para mona.

Mediante el comando:

!mona pattern_create 400

se genera un patrón único de 400 bytes.

Este patrón contiene una secuencia de caracteres diseñada para que cada posición del buffer sea única.

El patrón se guarda en el archivo:

pattern.txt

Este archivo será utilizado en el siguiente paso para identificar la posición exacta donde el buffer sobrescribe el registro EIP.

Descubrimiento del offset de EIP
---
El siguiente script del laboratorio envía el patrón generado anteriormente al servidor FTP.

Durante la ejecución del script ocurre lo siguiente:

El script establece conexión con el servidor.
Se envía el patrón de 400 bytes.
El servidor procesa los datos recibidos.
Se produce un buffer overflow.
El programa se detiene debido a una violación de acceso.

En Immunity Debugger se observa el siguiente error:

Access violation when executing

Además, el registro EIP contiene el valor:

41326941

Este valor forma parte del patrón cíclico enviado al servidor.

Cálculo del offset
---
Para determinar exactamente qué posición del patrón sobrescribe el registro EIP se utiliza el comando:

!mona pattern_offset 41326941

La herramienta Mona devuelve el resultado:

found in cyclic pattern at position 246

Esto significa que el byte número 246 del buffer sobrescribe el registro EIP.

Este valor es fundamental para construir el exploit.

Verificación del control del flujo de ejecución
---
Una vez conocido el offset exacto se construye un nuevo payload compuesto por:

246 bytes de relleno
seguido del valor BBBB

Los caracteres BBBB corresponden al valor hexadecimal:

42424242

Al ejecutar el script se observa en Immunity Debugger que el registro EIP contiene exactamente ese valor.

Esto confirma que es posible controlar completamente el flujo de ejecución del programa.

En otras palabras, el atacante puede decidir qué dirección de memoria será ejecutada por el programa.

Identificación de bad characters
---
Antes de introducir shellcode en el exploit es necesario identificar los bad characters.

Los bad characters son bytes que el programa no procesa correctamente y que pueden romper el exploit.

Para identificarlos se utiliza el comando:

!mona bytearray

Este comando genera un array con todos los posibles valores de bytes.

Posteriormente se envían estos bytes al servidor y se analizan en memoria utilizando Immunity Debugger.

Tras varias iteraciones se identifican los siguientes bad characters:

\x00
\x0a
\x0d

Estos valores deberán excluirse del payload final.

Búsqueda de una instrucción JMP ESP
---
El siguiente paso consiste en localizar una instrucción que permita redirigir la ejecución hacia la pila.

Para ello se busca una instrucción JMP ESP, que salta directamente a la dirección almacenada en el registro ESP.

Se utiliza el comando:

!mona find -s "\xff\xe4" -cpb "\x00\x0a\x0d"

Este comando busca direcciones de memoria que contengan la instrucción JMP ESP y que no incluyan los bad characters identificados anteriormente.

Una de las direcciones encontradas es:

0x75A8E647

Esta dirección se utilizará para sobrescribir el registro EIP.

Ejecución del exploit
---
En esta fase final se construye y ejecuta el payload completo del exploit para comprobar que es posible controlar el flujo de ejecución del programa vulnerable.

El payload se compone de los siguientes elementos:

Relleno de 246 bytes (A's) para alcanzar el registro EIP

Dirección de memoria que contiene un JMP ESP

NOP sled

Shellcode

Durante el análisis se observa que las direcciones de memoria pueden cambiar entre ejecuciones debido a que el sistema tiene ASLR activado. Esto implica que las direcciones no serán siempre las mismas cada vez que se ejecute el binario.

Por este motivo, en el momento del análisis se selecciona una dirección válida desde Immunity Debugger, copiando la dirección de memoria mostrada en el depurador, por ejemplo:

75A8E647

Posteriormente se utiliza la opción "Go to expression" para localizar dicha dirección en memoria y verificar que en esa posición existe una instrucción JMP ESP.

Aunque en futuras ejecuciones esta dirección pueda cambiar debido al ASLR, en ese instante sabemos que esa dirección contiene un salto hacia el registro ESP, lo que nos permitirá redirigir el flujo de ejecución hacia los datos que hemos colocado en la pila.

Una vez localizada la dirección adecuada, se modifica el script del exploit para sustituir el valor del EIP overwrite por la dirección que contiene el JMP ESP.

De esta forma, el payload queda estructurado de la siguiente manera:

Se envían 246 bytes de relleno (A's) para sobrescribir el buffer.

Se sobrescribe el registro EIP con la dirección que contiene el JMP ESP.

Cuando el programa intenta continuar su ejecución, el salto redirige el flujo hacia la pila.

En la pila se encuentra el NOP sled, seguido del shellcode que será ejecutado.

A continuación se ejecuta el script 06, que envía el payload al servicio vulnerable.

Al analizar la ejecución en Immunity Debugger, se puede observar en la ventana de registros que el flujo del programa ha sido redirigido correctamente hacia la pila, lo que confirma que el control del EIP ha sido logrado.

En este punto se ha conseguido ejecutar instrucciones controladas por el atacante (NOPs y breakpoints), demostrando que el exploit funciona correctamente.

El siguiente paso del proceso consistirá en generar una shellcode real y modificar el script para incluirla en el payload, lo que permitirá finalmente obtener ejecución de código dentro del proceso vulnerable.
