.. -*- coding: utf-8 -*-

.. _rcs_subversion:

Clase 03 - PIII 2015 - 19-08-2015
=================================


Manejo del oscilador
====================

- 3 osciladores primarios (XTL, XT y HS)
	XTL: Con cristal de cuarzo o resonador cerámico (200kHz y 4MHz)
	XT: Cristal o resonador (4MHz a 10MHz)
	HS: Sólo cristal (10MHz a 25MHz)

	- Pines OSC1 y OSC2

- 1 secundario (LP)
	LP: Cristal o cerámico a 32kHz.
	- Pines SOSC1 y SOSC2

- 2 internos (FRC y LPRC): Sensibles a la temperatura y voltaje al cual trabaje el dispositivo
	FRC (Fast RC): Trabaja a 8MHz sin necesidad de conectar un cristal
	LPRC (Low Power RC): 512kHz. 

- 1 externo (ERC): Sensible a la temperatura y voltaje.
	ERC: Hasta 4MHz. Necesita de una resistencia y un condensador en OSC1. También se puede conectar a una señal de reloj externa (modo EC).

- PLL para multiplicar la frecuencia interna
	

.. figure:: /images/oscilador.png
   :target: http://ww1.microchip.com/downloads/en/DeviceDoc/70046E.pdf


    Licencia Creative Commons Atribución-CompartirIgual 2.5 Argentina (CC BY-SA 2.5 AR)
	
	

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














Cálculo de Fcy (Frequency Cycle)

- Fcy es utilizada para calcular el ciclo de máquina (número de instrucciones por segundo)
- Se deriva de la frecuencia de oscilación Fosc	

 

- Para dsPIC30F4013

Fcy=  Fosc/4=  (Frec Oscilador x PLL)/4                                   Fcy=  10MHz/4=2,5MHz                              Tcy=  1/2,5MHz=400nseg


- Para dsPIC30FJ32MC202
Fcy=  Fosc/2=  (Frec Oscilador x PLL)/2


Ejercicio 1:

- Definir las siguientes funciones:

	void retardarUnSegundo();

	void retardo(int segundos)

	- Con la siguiente línea consumimos un ciclo de instrucción sin hacer nada:
	
		asm nop;
	

Manejo de temporizadores (para dsPIC30F4013)

- Los dsPIC30F tienen temporizadores de 16 bits aunque se pueden combinar para tener 32 bits.
- Proporcionan una base de tiempo

	- TMRx - Registro contador del temporizador
	- PRx - Registro de períodos
	- TxCON - Registros de control (contiene los bits TCS, TSYNC y TGATE)
	
- Además, cada uno de los timers tiene bits para control de interrupciones.

	- TxIE - Bit de control de la interrupción Timer
	- TxIF - Bit de estado de desbordamiento
	- TxIP<2:2> - Prioridad de la interrupción

 


 


 







Ejemplo:
- Hacerlo parpadear cada cierto números de ciclos haciendo uso de la interrupción del timer.

 

- Primero configuramos el servicio en la interrupción del timer.

void detectarIntT1() org 0x001a  {
    LATBbits.LATB0 = !LATBbits.LATB0;
    IFS0bits.T1IF=0;  // Borramos la bandera de interrupción T1
}

void main(){
  TRISBbits.TRISB0 = 0;
  LATBbits.LATB0 = 0;

  // Modo de operación Timer1
  T1CON=0x0000;

  // Modo operación Timer1: reloj interno, escala 1:1, empieza cuenta en 0
  TMR1=0;

  // Cuenta 500 ciclos
  PR1=500;

  // Interrupciones Timer1, borra Bandera de interrupción
  IFS0bits.T1IF=0;

  // Habilita interrupción
  IEC0bits.T1IE=1;

  // Arranca Timer1
  T1CONbits.TON=1;

  while(1)
    asm nop;
}




















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

- IFS0<15:0>, IFS1<15:0>, IFS2<15:0>
	- Banderas de solicitud de interrupción. (el software debe borrarlo - hay que hacerlo sino sigue levantando la interrupción).

- IEC0<15:0>, IEC1<15:0>, IEC2<15:0>
	- Bits de control de habilitación de interrupción.

- IPC0<15:0>... IPC10<7:0>
	- Prioridades

- INTCON1<15:0>, INTCON2<15:0>
	- Control de interrupciones.
		- INTCON1 contiene el control y los indicadores de estado. 
		- INTCON2 controla la señal de petición de interrupción externa y el uso de la tabla AIVT.


Secuencia de interrupción
+++++++++++++++++++++++++

- Las banderas de interrupción se muestrean en el comienzo de cada ciclo de instrucción por los registros IFSx. 
- Una solicitud de interrupción pendiente (IRQ) se indica mediante la bandera en '1' en un registro IFSx. 
- La IRQ provoca una interrupción si se encuentra habilitado con IECx. 
- El IVT contiene las direcciones iniciales de las rutinas de interrupción para cada fuente de interrupción.

Interrupciones externas INT0 INT1 y INT2

.. code-block::

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











