.. -*- coding: utf-8 -*-

.. _rcs_subversion:

Clase 02 - PIII 2015
====================

Ejercicio 4 (clase pasada): Una opción para resolverlo.

.. code-block::

    int contadorRB0 = 0;
    int contadorRB1 = 0;

    void main()  {
        TRISBbits.TRISB0 = 0;
        TRISBbits.TRISB1 = 0;

        LATBbits.LATB0 = 1;
        LATBbits.LATB1 = 1;

        while(1)  {
            contadorRB0++;
            contadorRB1++;

            if (contadorRB0 >= 250)  {
                LATBbits.LATB0 = ~LATBbits.LATB0;
                contadorRB0 = 0;
            }
        
            if (contadorRB1 >= 133)  {
                LATBbits.LATB1 = ~LATBbits.LATB1;
                contadorRB1 = 0;
            }
        
            Delay_ms(1);
        }
    }


Interrupciones
==============

- Eventos que hacen al dsPIC dejar de realizar lo que está haciendo y pase a ejecutar otra tarea.
- Las causas pueden ser diferentes (hasta 45 fuentes): Interrupciones externas, Timers, ADC, UART, etc.
- 7 niveles de prioridad (1 a 7 a través de los registros IPCx). Con 0 se desactiva la interrupción.
- Existe una tabla de vector de interrupción (IVT) que nos dice dónde escribir nuestra función que atiende la interrupción.
- Cuando una interrupción es atendida, el PC (Program Counter) se carga con la dirección almacenada en la ubicación de vectores en la memoria del programa que corresponde a la interrupción.

- Escribir una rutina del servicio de interrupción (ISR)
	- Función void sin parámetros
	- No puede ser invocada

.. code-block::

	void interrupcionExterna()  org 0x0014  {

	}

- *IFS0<15:0>, IFS1<15:0>, IFS2<15:0>*
- Banderas de solicitud de interrupción. (el software debe borrarlo - hay que hacerlo sino sigue levantando la interrupción).

- IEC0<15:0>, IEC1<15:0>, IEC2<15:0>
	- Bits de control de habilitación de interrupción.

- IPC0<15:0>... IPC10<7:0>
	- Prioridades



- INTCON1<15:0>, INTCON2<15:0>
	- Control de interrupciones.
		- INTCON1 contiene el control y los indicadores de estado. 
		- INTCON2 controla la señal de petición de interrupción externa y el uso de la tabla AIVT.

	- El usuario debe asegurarse de poner en cero las banderas antes de activar una interrupción.


Secuencia de interrupción

- Las banderas de interrupción se muestrean en el comienzo de cada ciclo de instrucción por los registros IFSx. 
- Una solicitud de interrupción pendiente (IRQ) se indica mediante la bandera en '1' en un registro IFSx. 
- La IRQ provoca una interrupción si se encuentra habilitado con IECx. 



- El IVT contiene las direcciones iniciales de las rutinas de interrupción para cada fuente de interrupción.











Interrupciones externas INT0  INT1  INT2

	void detectarInt0() org 0x0014  {
								0x0014 - INT0  
								0x0034 - INT1
								0x0042 - INT2
	}

- Para elegir lanzar la interrupción con flanco ascendente o descendente hacemos:
	INTCON2bits.
			INT0EP 
			INT1EP
			INT2EP
					0 - Ascendente
					1 - Descendente

IFS0bits.INT0IF  --- Borramos la bandera

IEC0bits.INT0IE  --- Habilitamos la interrupción
			

Ejemplo: Cambia de estado un led en PORTD0 cada vez que se detecta un flanco descendente en INT0

void detectarInt0() org 0x0014  {
  IFS0bits.INT0IF = 0;
  LATDbits.LATD0 = ~LATDbits.LATD0;

}

void configuracionPuertos()  {

  TRISDbits.TRISD0 = 0;  // Para led Int0
}


void main()  {
    configuracionPuertos();

    INTCON2bits.INT0EP = 1;

    IEC0bits.INT0IE = 1;

    while(1)  {
    }
}


Ejemplo (para dsPIC30F4013):

El ejemplo muestra cómo dsPIC reacciona a un flanco de señal ascendente en el puerto RF6 (INT0). Para cada flanco ascendente el valor en el puerto D se incrementa en 1.

void deteccionDeInterrupcion() org 0x0014{ // Interrupción en INT0
  LATD++;		// Incrementamos el contador
  IFS0.F0 = 0;      // Decimos que ya atendimos la interrupción
}

void main(){
  TRISD = 0;      // Contador de eventos por interrupción
  TRISA = 0xFFFF; // PORTA para leer el pin RA11
  IFS0 = 0;       // Interrupción puesta en cero
  IEC0 = 1;       // Interrupción en el flanco ascendente de INT0 (RA11)
  while(1) 
    asm nop;
}






- Se utiliza el PORTD para mostrar el número de eventos de interrupción.
- PORTF como entrada para producir una interrupción cuando en INT0 (RA11) cambie de cero a 1. 
- En el registro IEC0, el bit menos significativo está en uno para interrumpir con INT0. 
- Cuando se produce una interrupción, la función deteccionDeInterrupcion se invoca
- Por la instrucción org en la tabla de vectores de interrupción se escribe la función en la posición de memoria 0x000014.
- Cuando en RA11 aparece un 1, se escribe un 1 en el bit menos significativo del registro IFS0. A continuación, se verifica si la interrupción INT0 está activado (el bit menos significativo de IEC0). 
- Se lee de la tabla de vectores de interrupción qué parte del programa se debe ejecutar. 
- En la posición 0x000014 está la función deteccionDeInterrupcion , se ejecuta y vuelve al main.
- Dentro de la función, el software debe poner a cero el bit menos significativo de IFS0. Si no, siempre pensará que hay interrupción.
- Luego incrementamos en 1 LATD.

Ejercicio:
- Realizar el mismo ejemplo para dsPIC30F3012 y grabarlo con PICKit2









Introducción
============

- Brinda al estudiante herramientas de programación de microcontroladores para el procesamiento digital de señales.
- Conocimientos sobre programación de hardware específico para tratamiento de señales.
- Complementa lo desarrollado en "Teoría de Señales y Sistemas Lineales" y "Tratamiento Digital de Señales". 


Familias de microcontroladores Microchip
----------------------------------------

*MCU (MicroController Unit)*
	- Microcontroladores clásicos
	- Las operaciones complejas las realiza en varios ciclos
	
*DSP (Digital Signal Processor)*
	- Microcontroladores para procesamiento de señales
	- Las operaciones complejas las realiza en un sólo ciclo.

*DSC (Digital Signal Controller)*
	- Híbrido MCU/DSP
	- Controlador digital de señales

*dsPIC (Nombre que utiliza Microchip para referirse a sus DSC)*
	- PIC de 16 bits (registros de 16 bits)
	- Estudiaremos las familias dsPIC30F y dsPIC33F
	- Se pueden conseguir en Córdoba los siguientes: 
	#. dsPIC30F4013 (40 pines)
 	#. dsPIC30F2010 (28 pines)
	#. dsPIC33FJ32MC202 (28 pines)

Softwares
---------
- Proteus
- mikroC para dsPIC

*Proteus*
	- Conjunto de programas para diseño y simulación
	- Desarrollado por Labcenter Electronics (http://www.labcenter.com)
	- Versión actual: 8.3
	- Versión 8.1 para compartir. Algunos problemas con Windows 7
	- Versión 7.9 para compartir. Estable para XP, Windows 7 y Windows 8
	- Herramientas principales: ISIS y ARES

*ISIS (Intelligent Schematic Input System - Sistema de Enrutado de Esquemas Inteligente)*
	- Permite diseñar el circuito con los componentes.
	- Permite el uso de microcontroladores grabados con nuestro propio programa.
	- Contiene herramientas de medición, fuentes de alimentación y generadores de señales.
	- Puede simular en tiempo real mediante VSM (Virtual System Modeling -Sistema Virtual de Modelado).

*ARES (Advanced Routing and Editing Software - Software de Edición y Ruteo Avanzado)*
	- Permite ubicar los componentes y rutea automáticamente para obtener el PCB (Printed Circuit Board).
	- Permite ver una visualización 3D de la placa con sus componentes.

*mikroC para dsPIC*
	- Compilador C para dsPIC
	- Incluye bibliotecas de programación
	- Última versión 6.2 (febrero 2014)
	- Desarrollado por MikroElektronika ( http://www.mikroe.com/mikroc/dspic )
	- MikroElektronika también dispone de placas de desarrollo como la Easy dsPIC que disponemos en el Lab


*Ejercicio 1*: Regulador de tensión para los dsPIC33F.
	- Alimentación desde un conector USB.
	- Utilizar herramientas de medición para asegurarse de los voltajes obtenidos.

*Ejercicio 2*: Alimentar el dsPIC33FJ32MC202.
	- Conectar el Master Clear
	- Utilizar capacitores de desacoplo
	- Conectar un cristal de cuarzo
	- Grabarle un programa simple (ver ejercicio 3)

*Ejercicio 3*: Crear un programa "Hola mundo" para el dsPIC33FJ32MC202.
	- Escribir una función void configuracionInicial() para configurar el puerto RB0 como salida
	- En la función main encender y apagar un LED en RB0 cada 1 segundo

*Ejercicio 4*: Programar en RB1 un segundo LED que encienda cada un determinado tiempo distinto al tiempo de RB0.
	- El LED en RB0 que encienda y apague cada 250 ms
	- El LED en RB1 que encienda y apague cada 133 ms


**Proteus (primer proyecto)**

- New Design
- Component mode (panel izquierdo)
- P (Pick Device) - permite seleccionar los componentes a utilizar en este proyecto
	- DSPIC33FJ32MC202
	- USBCONN
	- LM317L
	- A700 (es el prefijo de capacitores electrolíticos de alto valor)
	- CAP-ELEC - Capacitores electrolíticos generales
	- POT-HG - Potenciómetro
	- RES - Resistencia
	- LED-RED
	- CRYSTAL
- Terminals Mode - Permite agregar tierra, entrada, salida, etc.
	- GROUND


**Código ejemplo del Hola Mundo**

.. code-block::

	void main()  {
  	    TRISBbits.TRISB0 = 0;            
  	    LATBbits.LATB0 = 0;    

  	    while(1) {
    	        LATBbits.LATB0 = ~LATBbits.LATB0;       
    	        Delay_ms(1000);
  	    }
	}






