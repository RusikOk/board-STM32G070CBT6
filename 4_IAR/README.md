# описание проектов IAR

## 1_SPI_UART_2byte

вызываем прерывание от UART каждые 2 байта и пишем в 16 битную переменную.

отправляем 64 шеснадцатибитных слова с порядком следования little-endian. настройки COM порта 2000000 8N1:
<img src="https://github.com/RusikOk/board-STM32G070CBT6/blob/main/7_%D1%84%D0%BE%D1%82%D0%BE/1_SPI_UART_2byte/4.png" alt=""><br>

останавливаем выполнение и видим, что мы правильно получили данные из UART1:
<img src="https://github.com/RusikOk/board-STM32G070CBT6/blob/main/7_%D1%84%D0%BE%D1%82%D0%BE/1_SPI_UART_2byte/5.png" alt=""><br>

ссылки: <a href="https://github.com/RusikOk/board-STM32G070CBT6/blob/main/6_Docklight/1_SPI_UART_2byte.ptp">файлик доклайта</a><br>

## 2_SPI_SLAVE

проверка времени реакции на программный NSS сигнал. 
как оказалось у M0 серии нету счетчика тактов ни CYCLECOUNTER, ни даже DWT_CYCCNT. потому началось веселье с ногодрыгом и осциллографом)

### дергаем всема ножками МК на порту PC используя CMSIS 

<img src="https://github.com/RusikOk/board-STM32G070CBT6/blob/main/7_%D1%84%D0%BE%D1%82%D0%BE/2_SPI_SLAVE/SDS00009.BMP" alt="">

```c++
int main(void)
{
  while(1)
  {
          GPIOC->BSRR = 0xffffffff; // устанавливаем 1 на всем порте
          GPIOC->BRR =  0xffffffff; // устанавливаем 0 на всем порте
          HAL_Delay(100);
  }
}
```

### проверяем какое время нужно для перехода пина 1 -> 0 используя HAL

<img src="https://github.com/RusikOk/board-STM32G070CBT6/blob/main/7_%D1%84%D0%BE%D1%82%D0%BE/2_SPI_SLAVE/SDS00010.BMP" alt="">

```c++
#define SPI2_NSS_H2L()          HAL_GPIO_WritePin(SPI2_NSS_GPIO_Port, SPI2_NSS_Pin, GPIO_PIN_RESET)
#define SPI2_NSS_L2H()          HAL_GPIO_WritePin(SPI2_NSS_GPIO_Port, SPI2_NSS_Pin, GPIO_PIN_SET)

int main(void)
{
  while(1)
  {
          GPIOC->BSRR = 0xffffffff; // устанавливаем 1 на всем порте
          SPI2_NSS_H2L();
          GPIOC->BRR =  0xffffffff; // устанавливаем 0 на всем порте
          HAL_Delay(100);
  }
}
```

### проверяем какое время нужно для перехода пина 1 -> 0 используя HAL и попадания в обработчик прерывания

<img src="https://github.com/RusikOk/board-STM32G070CBT6/blob/main/7_%D1%84%D0%BE%D1%82%D0%BE/2_SPI_SLAVE/SDS00011.BMP" alt="">

```c++
#define SPI2_NSS_H2L()          HAL_GPIO_WritePin(SPI2_NSS_GPIO_Port, SPI2_NSS_Pin, GPIO_PIN_RESET)
#define SPI2_NSS_L2H()          HAL_GPIO_WritePin(SPI2_NSS_GPIO_Port, SPI2_NSS_Pin, GPIO_PIN_SET)

void HAL_GPIO_EXTI_Falling_Callback(uint16_t GPIO_Pin)
{        
        GPIOC->BRR =  0xffffffff; // устанавливаем 0 на всем порте
        HAL_SPI_TransmitReceive_DMA(&hspi1, (uint8_t *)&slaveTxBuf, (uint8_t *)&slaveRxBuf, SLAVE_TX_RX_BUF_LEN);
}

int main(void)
{
  while(1)
  {
          GPIOC->BSRR = 0xffffffff; // устанавливаем 1 на всем порте
          SPI2_NSS_H2L();
          HAL_Delay(100);
  }
}
```

### проверяем какое время нужно для перехода пина 1 -> 0 используя HAL, попадания в обработчик прерывания и запуск чтения SPI по DMA 

<img src="https://github.com/RusikOk/board-STM32G070CBT6/blob/main/7_%D1%84%D0%BE%D1%82%D0%BE/2_SPI_SLAVE/SDS00012.BMP" alt="">

```c++
#define SPI2_NSS_H2L()          HAL_GPIO_WritePin(SPI2_NSS_GPIO_Port, SPI2_NSS_Pin, GPIO_PIN_RESET)
#define SPI2_NSS_L2H()          HAL_GPIO_WritePin(SPI2_NSS_GPIO_Port, SPI2_NSS_Pin, GPIO_PIN_SET)

void HAL_GPIO_EXTI_Falling_Callback(uint16_t GPIO_Pin)
{        
        HAL_SPI_TransmitReceive_DMA(&hspi1, (uint8_t *)&slaveTxBuf, (uint8_t *)&slaveRxBuf, SLAVE_TX_RX_BUF_LEN);
        GPIOC->BRR =  0xffffffff; // устанавливаем 0 на всем порте		
}

int main(void)
{
  while(1)
  {
          GPIOC->BSRR = 0xffffffff; // устанавливаем 1 на всем порте
          SPI2_NSS_H2L();
          HAL_Delay(100);
  }
}
```

### проверяем какое время нужно для перехода пина 1 -> 0 используя CMSIS и попадания в обработчик прерывания

<img src="https://github.com/RusikOk/board-STM32G070CBT6/blob/main/7_%D1%84%D0%BE%D1%82%D0%BE/2_SPI_SLAVE/SDS00013.BMP" alt="">

```c++
#define SPI2_NSS_H2L()          SPI2_NSS_GPIO_Port->BRR = (uint32_t)SPI2_NSS_Pin
#define SPI2_NSS_L2H()          SPI2_NSS_GPIO_Port->BSRR = (uint32_t)SPI2_NSS_Pin

void HAL_GPIO_EXTI_Falling_Callback(uint16_t GPIO_Pin)
{        
        GPIOC->BRR =  0xffffffff; // устанавливаем 0 на всем порте
        HAL_SPI_TransmitReceive_DMA(&hspi1, (uint8_t *)&slaveTxBuf, (uint8_t *)&slaveRxBuf, SLAVE_TX_RX_BUF_LEN);
}

int main(void)
{
  while(1)
  {
          GPIOC->BSRR = 0xffffffff; // устанавливаем 1 на всем порте
          SPI2_NSS_H2L();
          HAL_Delay(100);
  }
}
```

### проверяем какое время нужно для перехода пина 1 -> 0 используя CMSIS, попадания в обработчик прерывания и запуск чтения SPI по DMA 

<img src="https://github.com/RusikOk/board-STM32G070CBT6/blob/main/7_%D1%84%D0%BE%D1%82%D0%BE/2_SPI_SLAVE/SDS00012.BMP" alt="">

```c++
#define SPI2_NSS_H2L()          SPI2_NSS_GPIO_Port->BRR = (uint32_t)SPI2_NSS_Pin
#define SPI2_NSS_L2H()          SPI2_NSS_GPIO_Port->BSRR = (uint32_t)SPI2_NSS_Pin

void HAL_GPIO_EXTI_Falling_Callback(uint16_t GPIO_Pin)
{        
        HAL_SPI_TransmitReceive_DMA(&hspi1, (uint8_t *)&slaveTxBuf, (uint8_t *)&slaveRxBuf, SLAVE_TX_RX_BUF_LEN);
        GPIOC->BRR =  0xffffffff; // устанавливаем 0 на всем порте		
}

int main(void)
{
  while(1)
  {
          GPIOC->BSRR = 0xffffffff; // устанавливаем 1 на всем порте
          SPI2_NSS_H2L();
          HAL_Delay(100);
  }
}
```

## 3_UART_2byte_LLdriver

расбераюсь с устройством LL драйвера на примере UART

## 4_SPI_DMA_LL_driver

расбераюсь с устройством LL драйвера на примере SPI/DMA


ссылки:<br>
<a href="http://microsin.net/programming/arm/iar-techniques-for-measuring-the-elapsed-time-stm32.html">IAR: техники измерения времени выполнения кода STM32</a><br>
<a href="https://hubstub.ru/stm32/82-vremya-vipolneniya-koda-stm32.html">Время выполнения кода STM32</a><br>
