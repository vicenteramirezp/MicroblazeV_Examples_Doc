# Guía 8 : Uso de System ILA


## Contexto


## Diseño de Hardware

Abra Vivado y genere un nuevo proyecto. 

En este genere un diagrama de bloques. En el diagrama de bloques haga un sistema con MicroBlaze V con interrupciones habilitadas y con un periférico Uart como se ve en la[](#fig-bd_no_adder_tree). Si no recuerda como realizarlo puede revisar la [sección de diseño de Hardware de la guía 3](../../Ejemplo_3_Interrupts/Ejemplo_3/#diseno-de-hardware).

![Diagrama de bloques de MicroBlaze V con periférico Uart](img/bd_no_adder_tree.png){ #fig-bd_no_adder_tree width="1000" }

Desde el panel lateral *Boards* agregue **16 LEDs** y **16 Switches** al diagrama de bloques como se ve en la []().


![](img/.png){ #fig width="1000" }

Luego presione *Run Connection Automation* y mantenga las conexiones predeterminadas. Su diagrama de bloques debería verse como en la [](). Note que los leds estan en GPI0 y los interruptores estan en GPI2.


![](img/.png){ #fig width="1000" }


Luego agregue al diagrama de bloques la IP *Axi timer*, la cual es usada para medir tiempo o generar eventos periodicos. Tras agregar la IP corra *Run Connection Automation* para integrar esta IP a la interconexion AXI del sistema. Debería verse como en la []().

![](img/.png){ #fig width="1000" }


Ahora integre la IP "System_ILA" como se ve en la [](). Esta es una IP de depurado de Xillinx pata el monitoreo de señales a nivel de interfaz dentro de un sistema, particularmente AXI, AXI-Stream y AXI4-Lite. Al tener conocimiento de la naturaleza de la conexion, agrupa todas las señales correspondientes a una misma interfaz en el visor de ondas.

![](img/.png){ #fig width="1000" }


Haga doble click en la IP para configurarla, en opciones generales estan las siguientes opciones:


- **Monitor Type**: elige que observa la ip.
	- **Native**: Monitorea señales particulares.
	- **Interface**: Monitorea interfaces compuestas por varias señales como los canales de AXI.
	- **Mix**: Puede monitorear ambos.
- **Number of Interface Slots** : cuantas conecciones va a monitorear esta IP.
- **Sample Data Depth** : Profundidad del buffer de captura, es decir cuantas muestras son guardadas por evento gatillante.
- **Number of Comparators**: Numero de comparadores independientes por señal a usar como gatillante de captura. A cada señal gatillante se le puede asignar una condicion por comparador.
- **Trigger Out Port**: Señal de salida opcional que se pone en alto cuando este bloque es gatillado.
- **Trigger In Port**: Señal de entrada opcional que gatilla la captura de señales de manera independiente a las señales elegidas como gatillantes de forma interna en el bloque.
- **Capture Control**: Habilita calificacion de almacenado. Cuando esta apagado, el ILA almacena cada ciclo de reloj dentro del bufffer interno cuando es gatillado. Cuando esta activado se puede señalar una condicion de captura de manera que  solo los ciclos donde la condicion es verdadera se consume espacio del bugger inteno
- **Advanced Trigger**: Permite generar  condiciones de captura mas avanzadas basadas en maquinas de estado.

Tambien se tiene que este bloque automaticamente realiza una estimacion acerca de los recursos que va a utilizar y la muestra en un porcentaje de uso de las BRAMS disponibles en la placa. El consumo de recursos es directamente proporcional a la cantidad de interfaces medidas y tamaño de muestras por capturas.

En esta guía se van a medir dos interfaces por lo que fije **Number of Interface Slots** y para que el tamaño de la transacción no sea un impedimento para la visualizacion de la forma de onda, fije **Sample Data Depth** en 4096 como se ve en la []().


![](img/.png){ #fig width="1000" }

Note que no esta presente el botón *Run connection Automation*, esto es debido a que el System ILA se puede conectar a distintas interfaces AXI dentro de un mismo sistema.

Realice las siguientes conexiones para integrar el bloque al esquema axi y medir las dos interfaces de interes:

- *clk_out1* de **Clocking Wizard** a *clk* de **System ILA**.
- *peripheral_aresetn* de **Processor System Reset** a *reset* de **System ILA**.
- *SLOT_0_AXI* de **System ILA** a *S_AXI* de **AXI Timer**
- *SLOT_1_AXI* de **System ILA** a *S_AXI* de **AXI GPIO**

Debería verse como la []().

![](img/.png){ #fig-conexion-ILA-SLAVE width="1000" }

Finalmente conecte *interrupt* de **AXI Timer** a *In0* de **Inline Concat** para habilitar las interrupciones del temporizador como se ve en []().

![](img/.png){ #fig width="1000" }

Valide su diagrama de bloques, genere los productos de salida y la envoltura HDL.

Después de correr la síntesis y la implementación, el autor obtuvo la utilización de recursos vista en la [](#tbl-resources-global).

<div markdown="1" style="text-align: center;">

Table: Utilización de recursos {#tbl-resources-global}

| Proceso  | LUT | FF | BRAM      | 
| ------- | ----- | --------- | ---------------- |
| `Sintesis`   | 5718     | 6644        | 57     | 
| `Implementacion`  | 5686    | 7197    | 32         |

</div>

Tras exportar el archivo xsa **no cierre Vivado** debido a que el depurado a través de System ILA se realiza en esta plataforma.

## Diseño de Firmware


Abra Vitis, fije un workspace, cree un nuevo componente de plataforma y uno de aplicación.

En el componente de aplicación importe el archivo Ejemplo_8.c de la carpeta Ejemplo_8, la que tiene el siguiente código:

```c
#include <stdio.h>
#include <xgpio.h>
#include "xparameters.h"
#include "xil_printf.h"
#include "xtmrctr.h"
#include "xil_types.h"

XTmrCtr   TimerInstance;
XGpio     GpioInstance;


#define TIMER_COUNTER_0  0
#define SWITCH_CHANNEL  2
#define LED_CHANNEL     1

int main() {
    int Status;
    u32 valor_switches;
    u32 TimerValue1, TimerValue2;
	//Inicializacion de perifércos
    Status = XGpio_Initialize(&GpioInstance, XPAR_AXI_GPIO_0_BASEADDR);
    if (Status != XST_SUCCESS) return XST_FAILURE;
    Status = XTmrCtr_Initialize(&TimerInstance,XPAR_AXI_TIMER_0_BASEADDR);
    if (Status != XST_SUCCESS) return XST_FAILURE;

	// Fija que canal de GPIO es entrada y cual es salida
    XGpio_SetDataDirection(&GpioInstance, SWITCH_CHANNEL, 0xFFFFFFFF);
    XGpio_SetDataDirection(&GpioInstance, LED_CHANNEL,    0x00000000);
    // Escritura  (Canales AW W B)
    XGpio_DiscreteWrite(&GpioInstance, LED_CHANNEL, 0x00005555);
    // Lectura (AR R)
    valor_switches=XGpio_DiscreteRead(&GpioInstance, SWITCH_CHANNEL);

    // Cambio dinamico de estado de registro
    XTmrCtr_Start(&TimerInstance, TIMER_COUNTER_0);
    TimerValue1=XTmrCtr_GetValue(&TimerInstance, TIMER_COUNTER_0);    
    TimerValue2=XTmrCtr_GetValue(&TimerInstance, TIMER_COUNTER_0);

    
    return 0;
}
```

En este archivo se añade la librería "xtmrctr.h" que define el manejo del periférico AXI Timer.
Este programa sigue la siguiente secuencia:

- Se inicializan los periféricos.
- Define las direcciones de los GPIO (Switches como entradas y LEDs como salidas).
- Se realiza una escritura en los 16 LEDs con un patrón 0x0101010101.
- Se realiza una lectura del valor actual de los 16 switches.
- Se inicia el timer.
- Se mide el valor actual del timer dos veces.

Compile la aplicación haciendo click en *Build* del panel lateral. El autor obtuvo un ELF con un tamaño de 11.6 KB como se ve en la [](#tbl-elf-size).

<div markdown="1" style="text-align: center;">

Table: Tamaño del ELF {#tbl-elf-size}

| text  | data | bss | dec      |
| ------- | ----- | --------- | ---------------- |
|  7664  | 342     | 3616        | 11621     |

</div>


## Verificación

Para poder hacer uso del System ILA hay que programar la placa a través de Vivado, para esto hay que hacer que Vitis no reprograme la placa a nivel de hardware y se encarge solamente de cargar la aplicación.

Para esto vaya al menu lateral y presione el boton de configuración,tiene que posicionar el mouse en *Run* o *Debug* para que aparezca como se ve en la []().

![](img/.png){ #fig width="1000" }

En la ventana de configuración desmarque la casilla *Program Device* como se ve en la []().

![Deshabilitar programado de la placa desde Vitis](img/Disable_programing.png){ #fig width="1000" }

Conecte la placa a la FPGA.

Luego vuelva a Vivado y programe la placa, para esto vaya al menu lateral de la izquierda y presione *Open Target* como se ve en []().Esto lanzara un botón que dice "Auto_connect" , presionelo para que detecte automaticamente la placa.

![Open_hardware_manager](img/.png){ #fig width="1000" }

Esto lo llevará a la perspectiva de depurado de Vivado, como se ve en la [](), están las dos interfaces definidas en la [](#fig-conexion-ILA-SLAVE) con  *slot_0* correspondiendo al timer y *slot_1* correspondiendo al modulo GPIO.


![](img/.png){ #fig width="1000" }

Al hacer click en cualquiera de las dos interfaces AXI se expande en los 5 canales de AXI4-Lite definidos en [la sección de contexto](#contexto) como se ve en la [](#fig-5-channels).

![5 Canales AXI4-Lite vistos en la perspectiva de depuracion de Vivado](img/5_Canales.png){ #fig-5-channels width="1000" }

Para señalarle al bloque *System ILA* como realizar la captura de los datos hay que agregar señales particulares que actuaran como los gatillos que le diran cuando terminar y empezar una captura. Para esto haga click en el botón "+" de la pestaña Trigger Setup como se ve en [](#fig-Add_trigger).

![](img/Add_trigger.png){ #fig-Add_trigger width="1000" }


Las señales que mas nos sirven por canal son:

- WVALID: Señal que se pone en alto cuando el procesador realiza una escritura correctamente en el periférico.
- RREADY: Señal que se pone en alto cuando el procesador realiza una transacción de lectura correctamente.

Añada estas señales para *slot_0* y *slot_1* de manera que se puedan analizar todas las transacciones de ambas interfaces como se ve en la []().

![](img/Señales_gatillantes.png){ #fig width="1000" }

Ya estando añadidas las señales, hay que añadirles una condición, en este caso en cada una de las señales haga click en la seccion *Value* y fijela en R (0-to-1-transition) de manera que cuando estas señales se pongan en alto se realice la captura. Debería quedar como en la []().

![](img/Señales_gatillantes_rise.png){ #fig width="1000" }

Ya que se tienen varias condiciones gatillantes, hay que asignarles una logica al conjunto para definir el comportamiento de la captura.  Haga click el boton de *Set Trigger Condition* visto en la [](#fig-Trigger_condition) y fijelo en "Set Trigger Condition to Global OR" de manera que todas las transiciones gatillen de forma individual la captura.

![](img/Trigger_condition.png){ #fig-Trigger_condition width="1000" }

Luego haga click en *Program Device* en el panel lateral visto en la []() para programar la tarjeta, luego haga click en el pop-up "xc7a100t_0" que hace referencia al chip de la placa.

![](img/Program_device.png){ #fig width="1000" }


Esto lanzará otra ventana emergente, donde se mostrará el archivo Bitstream (que posee la definicion de hardware de la placa) y el archivo *ltx* que contiene la definición del System ILA.

![](img/Bitstream_ltx.png){ #fig width="1000" }

Tras programar el hardware de la placa, hay que lanzar la aplicacion desde Vitis.

En vitis empiece una sesion de depurado, de manera que se puedan apreciar las transacciones linea a linea.



![](img/Sesion_depurado_Vitis.png){ #fig width="1000" }

Una vez este en la sesión de depurado de Vitis, vuelva a Vivado para armar el ILA. 

En Vivado presione el botón *auto-retrigger* de ILA de manera que cada vez que termine una captura, el ILA se prepare automaticamente para recibir otra. Luego prepare el Bloque para capturas haciendo click en el boton de  *Run Trigger* como se ve en la []()

![Auto_trigger](img/.png){ #fig width="1000" }

Luego vuelva a Vitis y presione *Step Over(F10)* hasta pasar la linea:

```c
Status = XTmrCtr_Initialize(&TimerInstance,XPAR_AXI_TIMER_0_BASEADDR);
```

Donde se realiza la inicializacion del periférico AXI Timer. Note que el periférico AXI GPIO no realiza ninguna escritura ni lectura de registros al momento de inicializarse.

Una vez que paso esa linea, vuelva a Vivado. Notará que ahora hay datos en el visor de ondas, esto es debido a que se realizaron escrituras en el modulo para inicializarlo.

Expanda los canales de escritura de *slot_0*: **AW Channel** y **W Channel** como se ve en la []()

![](img/.png){ #fig width="1000" }


Para verlo en mas detalle haga click en Zoom in como se ve en la [](). Note que el zoom se realizará enfocando el marcador amarillo el cual puede arrastrar con el mouse.

![](img/inicializacion_timer_zoom.png){ #fig width="1000" }


Para entender la secuencia hay que revisar el mapa de registros de AXI timer visto en la [](#Register_map) basado en la [documentacion oficial](https://docs.amd.com/v/u/en-US/axi_timer_ds764). Note que por defecto la IP internamente maneja dos temporizadores.

<div markdown="1" style="text-align: center;">

Table: Tamaño del ELF {#Register_map}

| Nombre Registro  | Direccion | Descripcion | 
| ------- | ----- | --------- |  
|  TCSR0  | 0x00     |    Registro de control de timer 0    | 
|  TLR0  | 0x04     |  Registro de carga de timer 0      | 
|  TCR0  | 0x08     |    Registro que guarda el valor actual del timer 0   | 
|  TCSR1  | 0x10     |    Registro de control de timer 1    | 
|  TLR1  | 0x14     |    Registro de carga de timer 1    | 
|  TCR1  | 0x18     | Registro que guarda el valor actual del timer 1     | 


</div>

Enfocandonos en la forma de onda vista en la [](#fig-Waveform_Inicializar_timer).

![Forma de onda de la inicializacion de AXI Timer](img/Waveform_Inicializar_timer.png){ #fig-Waveform_Inicializar_timer width="1000" }




En la forma de onda se realizan 6 operaciónes de escritura mostradas en la [](#tbl-init-timer), no se ira en mayor detalle a que corresponde cada bit en cada registro de control.


<div markdown="1" style="text-align: center;">

Table: Tamaño del ELF {#tbl-init-timer}

| AWADDR  | WDATA | Significado | 
| ------- | ----- | --------- |  
|  0x04  | 0x00000000     |     Se carga el valor 0 registro de carga al timer 0| 
|  0x00  | 0x00000120     |     Se fija el timer 0 al estado reset y se fija el valor actual del timer al valor del registro de carga del timer 0 | 
|  0x00  | 0x00000000     |     Se sale del estado de reset e inhabilita el timer 0 | 
|  0x14  | 0x00000000     |     Se carga el valor 0 registro de carga al timer 1| 
|  0x10  | 0x00000140     |     Se fija el timer 1 al estado reset y se fija el valor actual del timer al valor del registro de carga del timer 1 | 
|  0x10  | 0x00000000     |     Se sale del estado de reset e inhabilita el timer 1 | 
</div>

Esta secuencia esta descrita en la implementación de la función que inicializa el periférico :XTmrCtr_Initialize(), la cual hace un llamado a la funcion XTmrCtr_InitHw() donde esta la siguiente secuencia:

```c
/* Set the compare register to 0. */
XTmrCtr_WriteReg(InstancePtr->BaseAddress, TmrIndex,
            XTC_TLR_OFFSET, 0);
/* Reset the timer and the interrupt. */
XTmrCtr_WriteReg(InstancePtr->BaseAddress, TmrIndex,
            XTC_TCSR_OFFSET,
            XTC_CSR_INT_OCCURED_MASK | XTC_CSR_LOAD_MASK);
/* Release the reset. */
XTmrCtr_WriteReg(InstancePtr->BaseAddress, TmrIndex,
            XTC_TCSR_OFFSET, 0);
```

Note que son 3 escrituras en registros y al ser 2 temporizadores terminan siendo 6 escrituras.


![](img/.png){ #fig width="1000" }
