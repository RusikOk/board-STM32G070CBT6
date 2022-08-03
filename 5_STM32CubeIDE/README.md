# описание проектов STM32CubeIDE

<h2>1_SPI_UART_2byte</h2>

подключение и внешний вид:
<img src="https://github.com/RusikOk/board-STM32G070CBT6/blob/main/7_%D1%84%D0%BE%D1%82%D0%BE/1_SPI_UART_2byte/pinout.jpg" alt="">

запускаем программу на выполнение. оба массива пустые:
<img src="https://github.com/RusikOk/board-STM32G070CBT6/blob/main/7_%D1%84%D0%BE%D1%82%D0%BE/1_SPI_UART_2byte/1.png" alt=""><br>

первый брейкпоинт инициализация массива:
<img src="https://github.com/RusikOk/board-STM32G070CBT6/blob/main/7_%D1%84%D0%BE%D1%82%D0%BE/1_SPI_UART_2byte/2.png" alt=""><br>

второй брейкпоинт показывает отправку массива по DMA через SPI2 и прием в блокирующем режиме через SPI1:
<img src="https://github.com/RusikOk/board-STM32G070CBT6/blob/main/7_%D1%84%D0%BE%D1%82%D0%BE/1_SPI_UART_2byte/3.png" alt=""><br>

отправляем 64 шеснадцатибитных слова с порядком следования little-endian. настройки COM порта 2000000 8N1:
<img src="https://github.com/RusikOk/board-STM32G070CBT6/blob/main/7_%D1%84%D0%BE%D1%82%D0%BE/1_SPI_UART_2byte/4.png" alt=""><br>

останавливаем выполнение и видим, что мы правильно получили данные из UART1:
<img src="https://github.com/RusikOk/board-STM32G070CBT6/blob/main/7_%D1%84%D0%BE%D1%82%D0%BE/1_SPI_UART_2byte/5.png" alt=""><br>

ссылки:<br>
<a href="https://github.com/RusikOk/board-STM32G070CBT6/blob/main/6_Docklight/1_SPI_UART_2byte.ptp">файлик доклайта</a><br>