# **GUÍA:** *Cyclic redundancy check (CRC)- STM32LXX*
Para esta guía suponga que está enviando paquetes de datos a un dispositivo crítico, los paquetes contienen información que le indica al dispositivo que continúe operando o de lo contrario prodria causar accidentes fatales. Ahora imagine que los datos se corrompen, por medio de EMI (Electro Magnetic Interference) o cualquier forma de interferencia. Ahora es posible que la corrupción haya modificado el comando para que el dispositivo ocasione accidentes fatales. Sería conveniente si hubiera una manera de prevenir la corrupción de datos por completo, para poder generar una alarma o apagado del dispositivos para evitar resultados no deseados. Uno de los métodos para prevenir la corrupcion de datos es la verificación de redundancia cíclica (CRC).

## Características principales de CRC para microcontroladores STM32LXX:

- Utiliza el polinomio CRC-32 (Ethernet): 0x4C11DB7

<a href="https://www.codecogs.com/eqnedit.php?latex=\small&space;X^{32}&plus;X^{26}&plus;X^{23}&plus;X^{22}&plus;X^{16}&plus;X^{12}&plus;X^{11}&plus;X^{10}&plus;X^{8}&plus;X^{7}&plus;X^{5}&plus;X^{4}&plus;X^{2}&plus;X&plus;1" target="_blank"><img src="https://latex.codecogs.com/gif.latex?\small&space;X^{32}&plus;X^{26}&plus;X^{23}&plus;X^{22}&plus;X^{16}&plus;X^{12}&plus;X^{11}&plus;X^{10}&plus;X^{8}&plus;X^{7}&plus;X^{5}&plus;X^{4}&plus;X^{2}&plus;X&plus;1" title="\small X^{32}+X^{26}+X^{23}+X^{22}+X^{16}+X^{12}+X^{11}+X^{10}+X^{8}+X^{7}+X^{5}+X^{4}+X^{2}+X+1" /></a>

- Alternativamente, utiliza un polinomio totalmente programable con un tamaño programable (7, 8, 16, 32 bits)
- Maneja tamaño de datos de 8, 16, 32 bits
- Valor inicial de CRC programable
- Registro de datos de 32 bits de entrada/salida única
- Búfer de entrada para evitar la parada del bus durante el cálculo
- Cálculo de CRC realizado en 4 ciclos de reloj AHB (HCLK) para el tamaño de datos de 32 bits
- Registro de 8 bits de uso general (se puede usar para almacenamiento temporal)
- Opción de reversibilidad en datos de E/S

La siguiente imagen muestra la ejecución del algoritmo CRC paso a paso para las siguientes condiciones:

- `Input_Data = 0xC1`
- `POLY = 0xCB`
- `Initial_Crc = 0xFF`

![alt text](https://github.com/EyberRosero/Diseno-Digital-CRC-/blob/master/Step-by-step%20CRC%20computing%20example.JPG)

La idea detrás del algoritmo CRC es esta: 

1. Comience con un valor CRC inicial, en este caso es `0xFF`.

2. Luego se hace XOR con los datos de entrada.

3. Si en el resultado del paso 2 el bit más significativo es 0, entonces simplemente desplaza ese número a la izquierda por 1.

4. Si el resultado del paso 2 tiene un 1 en el bit mas significativo, entonces también se realiza un desplazamiento en 1, pero también se hace XOR con un polinomio elegido.

5. Al realizar estos pasos, también se debe realizar un seguimiento de su "bindex", que no es más que un contador para realizar un seguimiento de cuántas veces ha cambiado. Este contador debe ser igual en magnitud al número de bits en sus datos. El ejemplo anterior va de 0 a 7, en otras palabras, 8 iteraciones para un número/dato de 8 bits. El punto es cambiar cada bit.

6. El polinomio que se eligió no puede cambiar durante el proceso. Como puede ver arriba, el `POLY` es `0xCB` y cada vez que se hace XOR con un `POLY` siempre es `0xCB`.

Finalmente, terminará con un valor final y este valor es su suma de verificación o **valor CRC**. Tenga en cuenta que si mantiene todos los parámetros iguales (`Initial_Crc`, `Input_Data`, `POLY`) siempre obtendrá el mismo valor de retorno. Esto está bien, porque lo que se esta haciendo es verificación de errores. Una cosa más a tener en cuenta es que CRC también se utiliza para verificar los datos escritos en la memoria. Por ejemplo, si está guardando en una tarjeta SD u otro dispositivo y desea verificar los datos escritos, ejecute un CRC en los datos originales y la copia para ver si coinciden.

## Ejemplo de aplicación: 

El escenario es el siguiente: se tienen dos dispositivos, uno es un receptor al que llamaremos RX y el otro es un transmisor al que llamaremos TX. El TX desea enviar datos al RX pero también quiere asegurarse de que los **datos enviados se ignoren si están dañados**. Por lo tanto, el TX ejecutará los datos que desea enviar a través de un algoritmo CRC. Luego tendrá un **código CRC** que será único para esos datos. Entonces **TX envía sus datos junto con el código CRC** al RX.
Ahora RX recibe los datos y el código CRC. RX ejecutará los datos a través del mismo algoritmo CRC. RX ahora habrá calculado su propio código CRC. **RX ahora comparará el CRC que recibió con el que calculó** y si coinciden, entonces los datos no están dañados. Si los códigos CRC no coinciden, entonces algo está dañado y, por lo tanto, sabemos que el paquete recibido no es bueno y debe ser ignorado o manejado de alguna manera, tal vez pidiendo un reenvío de TX. Para que esto funcione, TX y RX deben usar exactamente el mismo algoritmo para la verificación de errores, por lo que todos los valores deben coincidir (`Initial_Crc`, `Input_Data`, `POLY`). Tenga en cuenta que si los códigos CRC no coinciden, esto podría significar que los datos estaban corruptos o tal vez el código CRC enviado estaba dañado, realmente no se podría saber, pero esto no cambia el hecho de que tenga datos corruptos.

```
as
```
