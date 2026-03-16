# Sistema operativo del entorno

El entorno de investigación se ha preparado sobre un sistema operativo **Windows**, ya que muchas de las herramientas utilizadas en el análisis de vulnerabilidades en aplicaciones de escritorio están diseñadas específicamente para esta plataforma.

En el ámbito del exploit development, especialmente cuando se trabaja con vulnerabilidades relacionadas con corrupción de memoria (como buffer overflows), resulta fundamental utilizar herramientas que permitan observar en detalle el comportamiento del programa durante su ejecución. En este contexto, Windows proporciona un entorno adecuado para utilizar herramientas como **Immunity Debugger** y el plugin **Mona**, ampliamente utilizadas en laboratorios de explotación.

Además, algunas de las aplicaciones vulnerables utilizadas durante el laboratorio, como **VulnServer** y **FreeFloat FTP Server**, están diseñadas específicamente para ejecutarse en entornos Windows. Estas aplicaciones permiten simular escenarios reales donde un servicio vulnerable recibe datos a través de la red y puede ser analizado mediante técnicas de debugging y análisis de memoria.

El uso de un entorno controlado basado en Windows permite reproducir de forma segura situaciones donde se pueden analizar fallos en el manejo de memoria, observar el comportamiento del stack y estudiar cómo una entrada maliciosa puede alterar el flujo de ejecución de un programa.

Por este motivo, la elección de Windows como sistema operativo principal del laboratorio facilita el aprendizaje práctico de técnicas de análisis y explotación utilizadas en investigaciones reales de vulnerabilidades.
