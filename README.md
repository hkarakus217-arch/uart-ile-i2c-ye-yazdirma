# uart-ile-i2c-ye-yazdirma

#include "string.h"
#include "i2c-lcd.h"
#include "stdio.h"

#define RX_BUFFER_SIZE 16

uint8_t rx_data[RX_BUFFER_SIZE];

char rx_string_buffer[RX_BUFFER_SIZE];

int buffer_index = 0;


  // LCD'yi baslat
  
  lcd_init();
  
  lcd_clear();
	
 lcd_put_cur(0, 0);
 
  lcd_send_string("Hosgeldiniz :)");
  
  // Ilk karakteri almak i�in UART alimini baslat
  
  HAL_UART_Receive_IT(&huart1, (uint8_t *)rx_data, RX_BUFFER_SIZE);

  while (1)
  {
    /* USER CODE END WHILE */

    /* USER CODE BEGIN 3 */
		lcd_put_cur(1, 0);
		lcd_send_string(rx_data);
		HAL_GPIO_TogglePin(GPIOB, GPIO_PIN_9);
		HAL_Delay(200);
  }
  /* USER CODE END 3 */
}


void HAL_UART_RxCpltCallback(UART_HandleTypeDef *huart)
{
	if (huart->Instance == huart1.Instance)
	{
	 // Her karakter aliminda LED'i yakip söndür
		HAL_UART_Transmit(&huart1, (uint8_t *)rx_data, sizeof(rx_data),RX_BUFFER_SIZE);
		if (rx_data[0] == '\r' || rx_data[0] == '\n')
		{
				rx_string_buffer[buffer_index] = '\0';
				HAL_UART_Transmit(&huart1, (uint8_t *)rx_data, sizeof(rx_data),RX_BUFFER_SIZE);
//				lcd_clear();
//				lcd_put_cur(0, 0);
//				lcd_send_string(rx_data);
				
				buffer_index = 0;
		}
		else
		{
				if (buffer_index < RX_BUFFER_SIZE - 1)
				{
						rx_string_buffer[buffer_index++] = rx_data[0];
				}
		}
		HAL_UART_Receive_IT(&huart1, (uint8_t *)rx_data, RX_BUFFER_SIZE);
	}
}

}

