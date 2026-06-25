# Guías MicroBlaze V

Este sitio forma parte del material desarrollado en el trabajo de título "Diseño e implementación de sistemas de cómputo heterogéneo basados en procesador RISC-V y coprocesadores en lógica reconfigurable", realizado por Vicente Ramírez para optar al grado de Ingeniero Civil Electrónico de la Universidad Técnica Federico Santa María.

El objetivo de este repositorio es proporcionar una ruta de aprendizaje reproducible para el desarrollo de sistemas embebidos y plataformas de cómputo heterogéneo sobre FPGA utilizando MicroBlaze V. Las actividades fueron diseñadas con complejidad incremental, permitiendo al lector avanzar desde la creación de un sistema base hasta la integración y depuración de aceleradores hardware desarrollados mediante HDL y High-Level Synthesis (HLS).

En esta primera instancia del repositorio se incluyen 8 guías con un nivel progresivo de complejidad.

  - [Guía 1](Ejemplo_1_Sistema_base/Ejemplo_1.md): Sistema base MicroBlaze V.
  - [Guía 2](Ejemplo_2_Depurador/Ejemplo_2.md): Uso de Depurador en Vitis.
  - [Guía 3](Ejemplo_3_Interrupts/Ejemplo_3.md): Uso de periféricos e interrupciones.
  - [Guía 4](Ejemplo_4_Verilog/Ejemplo_4.md): Integración de periféricos diseñados en HDL.
  - [Guía 5](Ejemplo_5_HLS/Ejemplo_5.md): Integración de periféricos diseñados en HLS.
  - [Guía 6](Ejemplo_6_DDR/Ejemplo_6.md): Uso de memoria externa.
  - [Guía 7](Ejemplo_7_VGA/Ejemplo_7.md): Sistema de Video con acceso directo a memoria.
  - [Guía 8](Ejemplo_8_System_ILA/Ejemplo_8.md): Uso de System ILA para depuración.

Los insumos para cada una de las guías se encuentran en el siguiente repositorio:

[https://github.com/vicenteramirezp/Tutorial_MicroBlazeV](https://github.com/vicenteramirezp/Tutorial_MicroBlazeV)

## Plataforma utilizada

Estas guías se desarrollaron para el flujo embebido de AMD. Particularmente se hizo uso de las herramientas:

- Vivado 2025.2
- Vitis 2025.2
- Python 3.10

Y en todas las guías se hizo uso de la placa  "Nexys A7-100T".