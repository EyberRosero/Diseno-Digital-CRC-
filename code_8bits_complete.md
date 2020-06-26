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

/* Private function prototypes */
void uart2_send(uint8_t data);
void crc_init_8(void);
void crc_init_32(void);
```
