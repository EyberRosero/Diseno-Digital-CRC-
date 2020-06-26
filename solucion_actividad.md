## Solución actividad propuesta:

1. Crear una función `void crc_init_32(void)` en la cual se asigne y establezca un polinomio de 32 bits y un valor inicial tambien de 32 bits. 

```
void crc_init_32(void) {

    RCC->AHBENR |= RCC_AHBENR_CRCEN;       // Habilitar reloj para CRC
    CRC->CR  |= CRC_CR_RESET;              // Reestablecer calculo
    CRC->POL  = 0x04C11DB7;                      // Se asigna el polinomio deseado
    CRC->CR   = 0x00000000;                // Establecer poly a 8 bits
    CRC->INIT = 0xFFFFFFFF;                      // Valor inicial también de 8 bits
    
}
```

2. En la función `int main(void)`crear un ciclo en el cual se pongan 4 bytes en el registro de datos de 32 bits a la vez, y se obtenga el código CRC al final.

```
int main(void) {
    // datos ficticios para probar
    uint8_t data_array[4] = { 0x41, 0x20, 0xff, 0x10 };
  
    while (1) {
        
        //EX 2: Poner 4 bytes en el registro de datos de 32 bits a la vez, obtenga el código CRC
        crc_init_32();
        uint32_t all_bytes = 0x00;
        
        for (i = 3; i >= 1; i--) {
            all_bytes |= data_array[i] << (8 * i);
        }
        all_bytes |= data_array[0];
        CRC->DR = all_bytes;
        uint32_t crc_code = CRC->DR;
        uart2_send(((crc_code) & 0xFF));
        uart2_send(((crc_code >> 8) & 0xFF));
        uart2_send(((crc_code >> 16) & 0xFF));
        uart2_send(((crc_code >> 24) & 0xFF));
        
    }
    return 0;
}
```
