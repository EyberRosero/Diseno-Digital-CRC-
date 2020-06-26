## Código completo incluido funciones de ayuda
```
/* Includes */
#include <stddef.h>
#include "stm32l0xx.h"
#include "stdlib.h"
#include "stdint.h"
#include "stdbool.h"
/* Private macro */

/* Private variables */
bool SENDCOMPLETE = false;

/* Private function prototypes */
void uart2_send(uint8_t data);
void crc_init_8(void);
void crc_init_32(void);

/* Function main */

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

void crc_init_8(void) {

    RCC->AHBENR |= RCC_AHBENR_CRCEN;       // Habilitar reloj para CRC
    CRC->CR  |= CRC_CR_RESET;              // Reestablecer calculo
    CRC->POL  = 0xCB;                      // Se asigna el polinomio deseado
    CRC->CR   = 0x08;                // Establecer poly a 8 bits
    CRC->INIT = 0xFF;                      // Valor inicial también de 8 bits
    
}

void uart2_send(uint8_t data) {

 SENDCOMPLETE = false;
 USART2->TDR = data;
 while (SENDCOMPLETE == false);

}
```
