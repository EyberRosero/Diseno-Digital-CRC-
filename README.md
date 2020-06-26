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

## Algoritmo CRC 

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

## Registros del periférico CRC

El periférico CRC en la serie **L** contiene un total de 5 registros que se describen a continuación.

1. `CRC_DR` : el **CRC data register**. Al escribir en el registro de datos, le está dando al motor CRC los datos de entrada sobre los cuales calculará el CRC. Cuando lea del registro de datos obtendrá el resultado de ese cálculo.

2. `CRC_IDR` : el **CRC idr** almacena datos que realmente no tiene absolutamente ningún propósito en el cálculo de CRC, el manual dice que quizás puede usarlo para almacenar un byte de datos.

3. `CRC_CR` : el **CRC control register** nos permite configurar el CRC para invertir los datos de entrada o los datos de salida. Le permite seleccionar el tamaño polinómico. Y le permite restablecer todo el cálculo

4. `CRC_INIT` : el **CRC init regiter** le permite establecer el valor inicial de CRC, de forma predeterminada se establece en `0xFFFF FFFF`

5. `CRC_POL` : el **CRC poli register** le permite elegir el coeficiente polinómico utilizado en los cálculos. El valor predeterminado es `0x04C1 1DB7`

## Código:

A continuación hay una funcion separada que inicializa el CRC para usar un polinomio de 8 bits.

```
void crc_init_8(void) {

    RCC->AHBENR |= RCC_AHBENR_CRCEN;       // Habilitar reloj para CRC
    CRC->CR  |= CRC_CR_RESET;              // Reestablecer calculo
    CRC->POL  = 0xCB;                      // Se asigna el polinomio deseado
    CRC->CR   = 0x08;                // Establecer poly a 8 bits
    CRC->INIT = 0xFF;                      // Valor inicial también de 8 bits
    
}
```

Ahora aquí se ejecutan todos los datos a través del CRC y se obtiene el código CRC final, el cual será usado para la suma de verificación.

```
int main(void) {
    // datos ficticios para probar
    uint8_t data_array[4] = { 0x41, 0x20, 0xff, 0x10 };
  
    while (1) {
        
        //EX 1: Poner 4 bytes a través de CRC uno por uno, obtenga el código CRC al final
        crc_init_8();
        uint8_t i;
        for (i = 0; i < 4; i++) {
            CRC->DR = data_array[i];
        }
        uart2_send((uint8_t) (CRC->DR & 0xFF));
    }
    return 0;
}
```

El código CRC del código anterior en un monitor de puerto generado es:

EX 1: `0xDB`

Tenga en cuenta que no está limitado a utilizar un valor `POLY` de 8 bits y un `CRC_INIT` de 8 bits para sus datos de 8 bits, puede configurarlo para el modo de 32 bits y ejecutar sus datos de 8 bits a través de él. Luego obtendrá un código CRC de 32 bits único para sus datos de 8 bits si así lo prefiere. 

Puede ver el codigo completo incluido funciones de ayuda [aquí](https://github.com/EyberRosero/Diseno-Digital-CRC-/blob/master/code_8bits_complete.md).

## Actividad propuesta:

1. Crear una función `void crc_init_32(void)` en la cual se asigne y establezca un polinomio de 32 bits y un valor inicial tambien de 32 bits. 

2. En la función `int main(void)`crear un ciclo en el cual se pongan 4 bytes en el registro de datos de 32 bits a la vez, y se obtenga el código CRC al final.

Puede ver el código solución de la actividad propuesta [aquí](https://github.com/EyberRosero/Diseno-Digital-CRC-/blob/master/solucion_actividad.md).
