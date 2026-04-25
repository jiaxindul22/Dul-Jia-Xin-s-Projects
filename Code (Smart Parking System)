#include "main.h"

/* Private variables ---------------------------------------------------------*/
uint8_t ir_sensor_states[4] = {0}; // IR sensor states
uint8_t obstacle_detected;          // KY-032 obstacle detection state

/* Private function prototypes -----------------------------------------------*/
void SystemClock_Config(void);
static void MX_GPIO_Init(void);
uint8_t IR_Sensor_Read(uint16_t sensorPin);
void IR_Sensors_Init(void);
void LED_Set(uint8_t yellow, uint8_t red);
void Buzzer_On(void);
void Buzzer_Off(void);
uint8_t KY032_Read(void);

int main(void)
{
  HAL_Init();
  SystemClock_Config();
  MX_GPIO_Init();

  IR_Sensors_Init();

  // Introduce a delay to allow sensors to stabilize after power-up
  HAL_Delay(500);

  while (1)
  {
    // Read all IR sensors (PA1 - PA4)
    ir_sensor_states[0] = IR_Sensor_Read(GPIO_PIN_1);
    ir_sensor_states[1] = IR_Sensor_Read(GPIO_PIN_2);
    ir_sensor_states[2] = IR_Sensor_Read(GPIO_PIN_3);
    ir_sensor_states[3] = IR_Sensor_Read(GPIO_PIN_4);

    // Check if all IR sensors are active (detecting the correct color or region)
    if (ir_sensor_states[0] && ir_sensor_states[1] && ir_sensor_states[2] && ir_sensor_states[3]) {
      HAL_GPIO_WritePin(GPIOA, GPIO_PIN_0, GPIO_PIN_SET);  // Yellow LED turn on
      HAL_GPIO_WritePin(GPIOA, GPIO_PIN_7, GPIO_PIN_RESET);  // Red LED turn off
    } else {
      HAL_GPIO_WritePin(GPIOA, GPIO_PIN_0, GPIO_PIN_RESET);  // Yellow LED turn off
      HAL_GPIO_WritePin(GPIOA, GPIO_PIN_7, GPIO_PIN_SET);  // Red LED turn on
    }

    // Read obstacle detection state from KY-032
    obstacle_detected = KY032_Read();

    // If obstacle is detected, sound the buzzer
    if (obstacle_detected == GPIO_PIN_RESET) {
      HAL_GPIO_WritePin(GPIOA, GPIO_PIN_6, GPIO_PIN_SET);  // Buzzer on
    } else {
      HAL_GPIO_WritePin(GPIOA, GPIO_PIN_6, GPIO_PIN_RESET);  // Buzzer off
    }

    // Short delay to allow sensor reading stabilization
    HAL_Delay(100);
  }
}

// Initialize IR sensors (PA1 - PA4)
void IR_Sensors_Init(void) {
  GPIO_InitTypeDef GPIO_InitStruct = {0};

  __HAL_RCC_GPIOA_CLK_ENABLE();

  GPIO_InitStruct.Pin = GPIO_PIN_1 | GPIO_PIN_2 | GPIO_PIN_3 | GPIO_PIN_4;
  GPIO_InitStruct.Mode = GPIO_MODE_INPUT;
  GPIO_InitStruct.Pull = GPIO_PULLUP;  // Use pull-up for better signal stabilization
  HAL_GPIO_Init(GPIOA, &GPIO_InitStruct);
}

// Read the IR sensor state
uint8_t IR_Sensor_Read(uint16_t sensorPin) {
  return HAL_GPIO_ReadPin(GPIOA, sensorPin);
}

// Read KY-032 obstacle detection sensor (connected to PA5)
uint8_t KY032_Read(void) {
  return HAL_GPIO_ReadPin(GPIOA, GPIO_PIN_5);  // Read from PA5 (KY-032)
}

void SystemClock_Config(void)
{
  RCC_OscInitTypeDef RCC_OscInitStruct = {0};
  RCC_ClkInitTypeDef RCC_ClkInitStruct = {0};

  __HAL_RCC_PWR_CLK_ENABLE();
  __HAL_PWR_VOLTAGESCALING_CONFIG(PWR_REGULATOR_VOLTAGE_SCALE1);

  RCC_OscInitStruct.OscillatorType = RCC_OSCILLATORTYPE_HSI;
  RCC_OscInitStruct.HSIState = RCC_HSI_ON;
  RCC_OscInitStruct.HSICalibrationValue = 16;
  RCC_OscInitStruct.PLL.PLLState = RCC_PLL_NONE;
  if (HAL_RCC_OscConfig(&RCC_OscInitStruct) != HAL_OK)
  {
    Error_Handler();
  }

  RCC_ClkInitStruct.ClockType = RCC_CLOCKTYPE_SYSCLK | RCC_CLOCKTYPE_HCLK | RCC_CLOCKTYPE_PCLK1 | RCC_CLOCKTYPE_PCLK2;
  RCC_ClkInitStruct.SYSCLKSource = RCC_SYSCLKSOURCE_HSI;
  RCC_ClkInitStruct.AHBCLKDivider = RCC_SYSCLK_DIV1;
  RCC_ClkInitStruct.APB1CLKDivider = RCC_HCLK_DIV1;
  RCC_ClkInitStruct.APB2CLKDivider = RCC_HCLK_DIV1;
  if (HAL_RCC_ClockConfig(&RCC_ClkInitStruct, FLASH_LATENCY_0) != HAL_OK)
  {
    Error_Handler();
  }
}

static void MX_GPIO_Init(void)
{
  GPIO_InitTypeDef GPIO_InitStruct = {0};

  // Enable GPIO Clocks
  __HAL_RCC_GPIOA_CLK_ENABLE();

  // Configure GPIO pins PA1 - PA4 for IR sensors
  GPIO_InitStruct.Pin = GPIO_PIN_1 | GPIO_PIN_2 | GPIO_PIN_3 | GPIO_PIN_4;
  GPIO_InitStruct.Mode = GPIO_MODE_INPUT;
  GPIO_InitStruct.Pull = GPIO_PULLUP;  // Pull-up for better stability
  HAL_GPIO_Init(GPIOA, &GPIO_InitStruct);

  // Configure GPIO pins for LEDs and Buzzer
  GPIO_InitStruct.Pin = GPIO_PIN_0 | GPIO_PIN_6;  // Green and Red LEDs
  GPIO_InitStruct.Mode = GPIO_MODE_OUTPUT_PP;
  GPIO_InitStruct.Pull = GPIO_NOPULL;
  GPIO_InitStruct.Speed = GPIO_SPEED_FREQ_LOW;
  HAL_GPIO_Init(GPIOA, &GPIO_InitStruct);

  // Initialize Buzzer pin
  GPIO_InitStruct.Pin = GPIO_PIN_7; // Buzzer at PA7
  HAL_GPIO_Init(GPIOA, &GPIO_InitStruct);
}

void Error_Handler(void)
{
  // User can add their own implementation to report the error, e.g., blinking an LED
  __disable_irq();
  while (1)
  {
    // Infinite loop to stop execution in case of an error
    HAL_GPIO_TogglePin(GPIOA, GPIO_PIN_0);  // Toggle an LED to indicate error
    HAL_Delay(500); // 500 ms delay
  }
}
