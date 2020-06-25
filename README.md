# **GUÍA:** *Cyclic redundancy check calculation unit (CRC)- STM32LXX*
Para esta guía suponga que está enviando paquetes de datos a un dispositivo crítico, los paquetes contienen información que le indica al dispositivo que continúe operando o de lo contrario prodria causar accidentes fatales. Ahora imagine que los datos se corrompen, por medio de EMI (Electro Magnetic Interference) o cualquier forma de interferencia. Ahora es posible que la corrupción haya modificado el comando para que el dispositivo ocasione accidentes fatales. Sería conveniente si hubiera una manera de prevenir la corrupción de datos por completo, para poder generar una alarma o apagado del dispositivos para evitar resultados no deseados. Uno de los métodos para prevenir la corrupcion de datos es la verificación de redundancia cíclica (CRC).

## Características principales de CRC:

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


La unidad de cálculo CRC se utiliza para obtener un código CRC de una palabra de datos de 8, 16 o 32 bits y un polinomio generador.

```
as
```
