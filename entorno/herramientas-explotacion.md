# Herramientas de explotación

Durante el laboratorio se utilizaron herramientas diseñadas para facilitar el desarrollo de exploits y el análisis de vulnerabilidades relacionadas con el manejo incorrecto de memoria.

Estas herramientas permiten automatizar diferentes tareas que normalmente formarían parte del proceso manual de análisis.

## Mona

Mona es un plugin que se integra con **Immunity Debugger** y está especialmente diseñado para facilitar el desarrollo de exploits.

Esta herramienta automatiza muchas tareas que resultan habituales durante el proceso de explotación de vulnerabilidades.

Entre sus principales funcionalidades destacan:

- Generación de patrones para identificar offsets en la memoria
- Búsqueda de direcciones útiles dentro de los módulos cargados
- Identificación de módulos que no utilizan protecciones de seguridad
- Análisis de estructuras de memoria utilizadas por el programa

El uso de Mona permite simplificar considerablemente el proceso de desarrollo de exploits, especialmente cuando se trabaja con vulnerabilidades como los **buffer overflows**.
