# STM32 LED 警報解除模擬系統

## 📌 一、專題介紹

本專題使用 **STM32 NUCLEO-G474RE 開發板**，透過 C 語言與 HAL 函式庫，設計一套具備倒數計時、密碼驗證、LED 警示及蜂鳴器的警報解除模擬系統。

系統啟動後立即開始 30 秒倒數，使用者需要透過電腦上的 Python 程式輸入正確密碼解除警報。

輸入錯誤密碼時，系統會提高 LED 閃爍速度與呼吸燈亮度，模擬警報逐漸升級的效果。

如果倒數時間結束或連續輸入 5 次錯誤密碼，系統將進入失敗鎖定狀態，LED 全亮且蜂鳴器響起 2 秒。

另外設置 PC13 緊急停止按鈕，讓使用者可以隨時停止系統。

---

## 🛠️ 二、開發環境

| 項目 | 使用工具 |
|---|---|
| 開發板 | STM32 NUCLEO-G474RE |
| 微控制器 | STM32G474RET6 |
| 開發環境 | STM32CubeIDE |
| 硬體設定 | STM32CubeMX |
| STM32 程式語言 | C |
| 函式庫 | STM32 HAL |
| 電腦端程式 | Python 3 |
| UART 通訊套件 | pyserial |
| 通訊介面 | LPUART1 |
| 傳輸速率 | 115200 baud |
| 版本控制 | GitHub |

---

## 🔌 三、硬體腳位配置

| MCU 腳位 | 功能 | CubeMX 設定 |
|---|---|---|
| PC8 | 主 LED 閃爍 | GPIO_Output |
| PC6 | PWM 呼吸 LED | TIM3_CH1 / AF2 |
| PB1 | 有源蜂鳴器控制 | GPIO_Output |
| PC13 | 緊急停止按鈕 | GPIO_EXTI13 |
| PB0 | 玩家 1 按鈕 | GPIO_Input / Pull-up |
| PC7 | 玩家 2 按鈕 | GPIO_Input / Pull-up |
| PA2 | UART TX | LPUART1_TX / AF12 |
| PA3 | UART RX | LPUART1_RX / AF12 |

### 硬體接線

**主 LED（PC8）**

PC8 → 330Ω 電阻 → LED 正極，LED 負極接 GND。

**呼吸 LED（PC6）**

PC6 → 330Ω 電阻 → LED 正極，LED 負極接 GND。

**蜂鳴器（PB1）**

使用 PB1 GPIO 控制有源蜂鳴器。

本專題採用不額外加裝電晶體的配置，但必須確認蜂鳴器額定電壓與工作電流符合 STM32 GPIO 的輸出規格，否則不可直接連接。

**緊急停止按鈕（PC13）**

使用 NUCLEO 開發板內建 B1 按鈕，以 EXTI 中斷實現緊急停止。

**玩家按鈕**

- PB0：玩家 1 按鈕，按下連接 GND。
- PC7：玩家 2 按鈕，按下連接 GND。

目前兩個玩家按鈕保留為輸入功能，尚未加入額外的遊戲邏輯。

---

## ⚙️ 四、系統主要功能

### 1. 30 秒倒數計時

系統啟動後立即開始倒數 30 秒。

使用 `HAL_GetTick()` 計算經過時間：

```c
#define COUNTDOWN_MS 30000U

start_ms = HAL_GetTick();

if ((uint32_t)(HAL_GetTick() - start_ms) >= COUNTDOWN_MS) {
    enter_failure("FAILED: TIMEOUT\r\n");
}
```

倒數過程中，STM32 每秒透過 UART 傳送剩餘時間到 Python。

### 2. 主 LED 閃爍

PC8 LED 會隨著錯誤密碼次數增加而加快閃爍速度。

| 錯誤次數 | LED 翻轉間隔 |
|---|---|
| 0 次 | 500ms |
| 1 次 | 400ms |
| 2 次 | 300ms |
| 3 次 | 200ms |
| 4 次 | 100ms |
| 5 次 | 恆亮，進入失敗狀態 |

### 3. PWM 呼吸燈

使用 TIM3_CH1 控制 PC6 LED，製作逐漸變亮、逐漸變暗的呼吸效果。

PWM 設定：

- PWM 頻率：1kHz
- Prescaler：15
- Auto Reload Register：999
- 更新間隔：20ms
- 最低亮度：5%
- 初始最高亮度：20%

呼吸燈最大亮度隨錯誤次數提高：

| 錯誤次數 | 最大亮度 |
|---|---|
| 0 次 | 20% |
| 1 次 | 40% |
| 2 次 | 60% |
| 3 次 | 80% |
| 4 次 | 100% |
| 5 次 | 固定 100% |

### 4. UART 密碼驗證

STM32 使用 LPUART1 與電腦上的 Python 程式通訊。

通訊設定：

- Baud Rate：115200
- Data Bits：8
- Stop Bits：1
- Parity：None
- Hardware Flow Control：None

正確密碼：

```text
1234
```

輸入正確密碼後：

1. 停止 30 秒倒數。
2. 關閉 PC8 主 LED。
3. 關閉 PC6 呼吸 LED。
4. 關閉 PB1 蜂鳴器。
5. 顯示解除成功訊息。

輸入錯誤密碼後：

1. 錯誤次數增加 1。
2. 主 LED 閃爍速度加快。
3. 呼吸燈最大亮度提高。
4. 回傳錯誤次數到 Python。

### 5. 失敗鎖定與蜂鳴器

符合以下任一條件時，系統進入失敗狀態：

- 30 秒倒數結束。
- 錯誤密碼累積 5 次。

進入失敗狀態後：

- PC8 LED 保持恆亮。
- PC6 LED 保持 100% 亮度。
- PB1 蜂鳴器連續響 2 秒。
- 2 秒後蜂鳴器自動停止。
- LED 持續亮起。
- 無法再使用密碼解除警報。

蜂鳴器採用非阻塞方式計時：

```c
#define BUZZER_MS 2000U

buzzer_start_ms = HAL_GetTick();
buzzer_active = 1;

HAL_GPIO_WritePin(GPIOB, GPIO_PIN_1, GPIO_PIN_SET);
```

計時結束時關閉：

```c
if (app_state == STATE_FAILED && buzzer_active &&
    (uint32_t)(HAL_GetTick() - buzzer_start_ms) >= BUZZER_MS) {

    HAL_GPIO_WritePin(GPIOB, GPIO_PIN_1, GPIO_PIN_RESET);
    buzzer_active = 0;
}
```

使用 `HAL_GetTick()` 而不是 `HAL_Delay(2000)`，避免阻塞主程式及影響緊急停止功能。

### 6. PC13 緊急停止

PC13 使用 EXTI 外部中斷。

按下板載 B1 按鈕時：

- 立即停止倒數。
- 關閉全部 LED。
- 關閉蜂鳴器。
- 系統進入停止狀態。

停止後需要重新啟動開發板，才會再次開始 30 秒倒數。

---

## 💻 五、Python 控制程式

電腦端使用 Python 與 pyserial 進行 UART 通訊。

功能包含：

1. 自動搜尋可用 COM 埠。
2. 讓使用者選擇連線埠。
3. 預設 COM3。
4. 顯示 STM32 傳回的倒數時間。
5. 顯示錯誤密碼次數。
6. 接收使用者輸入的密碼。
7. 將密碼傳送給 STM32。

### 安裝 pyserial

在 PyCharm Terminal 輸入：

```bash
python -m pip install pyserial
```

如果使用 PyCharm 的 `.venv` 虛擬環境，必須確認套件安裝在執行程式的相同 Python 環境中。

### 執行 Python

```bash
python main.py
```

### 執行畫面範例

```text
===== STM32 LED 警報解除模擬 =====

可用 COM 埠：
COM3：STLink Virtual COM Port

請輸入 COM 埠（直接 Enter 使用 COM3）：

已連接 COM3

TIME=30s, ERRORS=0/5
TIME=29s, ERRORS=0/5
TIME=28s, ERRORS=0/5

請輸入密碼：1111
WRONG: 1/5

請輸入密碼：2222
WRONG: 2/5

請輸入密碼：1234
SUCCESS: DISARMED
```

以上為操作示意，實際顯示內容以程式版本及 STM32 回傳訊息為準。

---

## 🔧 六、STM32CubeMX 設定

### System Clock

使用 HSI 內部時脈。

- HSI：16MHz
- PLL：Disabled
- SYSCLK：16MHz
- AHB：16MHz
- APB1：16MHz
- APB2：16MHz

### GPIO

- PC8：GPIO Output / Push Pull
- PB1：GPIO Output / Push Pull
- PB0：GPIO Input / Pull-up
- PC7：GPIO Input / Pull-up
- PC13：GPIO EXTI / Rising Edge
- PC6：TIM3_CH1 / AF2

### TIM3 PWM

- Clock Source：Internal Clock
- Channel 1：PWM Generation CH1
- Prescaler：15
- Counter Period：999
- Pulse：50
- PWM Mode：PWM Mode 1
- Polarity：High

### LPUART1

- Mode：Asynchronous
- TX：PA2
- RX：PA3
- Baud Rate：115200
- Word Length：8 Bits
- Stop Bits：1
- Parity：None

使用 ST-LINK USB 虛擬 COM 埠進行通訊。需確認開發板的 VCP 焊橋與 UART 腳位配置相符。

### NVIC

啟用：

- EXTI15_10 中斷
- LPUART1 中斷

PC13 緊急停止中斷設定為較高優先權。

請避免 `main.c` 與 `stm32g4xx_it.c` 重複定義相同的中斷處理函式。

---

## 🔄 七、系統執行流程

```text
STM32 系統啟動
       |
       v
初始化 GPIO / PWM / UART
       |
       v
開始 30 秒倒數
       |
       v
PC8 LED 閃爍
PC6 LED 呼吸
       |
       v
等待 UART 密碼輸入
       |
       +---- 正確密碼 1234
       |          |
       |          v
       |       解除成功
       |       LED 全關
       |
       +---- 錯誤密碼
       |          |
       |          v
       |       錯誤次數 +1
       |          |
       |          v
       |       更新 LED 速度與亮度
       |          |
       |          v
       |       累積 5 次？
       |          |
       |          +---- 否：繼續倒數
       |          |
       |          +---- 是：失敗鎖定
       |
       +---- 30 秒倒數結束
                  |
                  v
              失敗鎖定
                  |
                  v
              LED 全亮
              蜂鳴器響 2 秒
                  |
                  v
              等待 PC13

任何狀態按下 PC13
       |
       v
全部輸出關閉
       |
       v
系統停止
```

---

## ✅ 八、測試項目

| 測試 | 預期結果 |
|---|---|
| 開機啟動 | 開始 30 秒倒數 |
| PC8 主 LED | 每 500ms 切換亮滅 |
| PC6 呼吸燈 | 平滑變亮與變暗 |
| 輸入錯誤密碼 | 錯誤次數增加 |
| 錯誤累積 4 次 | LED 閃爍速度達最快 |
| 錯誤累積 5 次 | 進入失敗鎖定 |
| 30 秒倒數結束 | 進入失敗鎖定 |
| 蜂鳴器 | 失敗後連續響 2 秒 |
| 輸入正確密碼 | 解除警報 |
| 失敗後輸入正確密碼 | 保持失敗鎖定 |
| 按下 PC13 | 所有輸出立即關閉 |
| UART 等待輸入 | LED 與倒數持續運作 |

---

## 📚 九、使用技術與學習成果

本專題整合以下嵌入式系統技術：

**GPIO（General Purpose Input/Output）**

控制 LED、蜂鳴器及讀取按鈕狀態。

**EXTI（External Interrupt）**

利用外部中斷實現緊急停止。

**PWM（Pulse Width Modulation）**

透過 TIM3 調整 LED 占空比，製作呼吸燈效果。

**UART（Universal Asynchronous Receiver/Transmitter）**

建立 STM32 與 Python 的雙向序列通訊。

**HAL_GetTick()**

使用非阻塞方式實作倒數、LED 閃爍及蜂鳴器計時。

**State Machine（狀態機）**

區分倒數中、成功、失敗鎖定及緊急停止等狀態。

---

## 📝 十、專題結論

本專題成功整合 STM32 的 GPIO、EXTI、TIM3 PWM、LPUART1 及 HAL 計時功能，完成一套以 LED、蜂鳴器與密碼驗證為核心的警報解除模擬系統。

透過非阻塞式程式設計，使倒數計時、LED 閃爍、PWM 呼吸燈及 UART 通訊能夠同時運作。

本專題不僅練習 STM32 的基本硬體控制，也加深了對中斷處理、計時邏輯、狀態管理與電腦串列通訊的理解。

未來可增加玩家按鈕互動、LCD 倒數顯示、更多警報模式及完整的遊戲流程。

---

**Project: STM32 NUCLEO-G474RE LED Alarm Simulation System**

**Language: C / Python**

**Development Environment: STM32CubeIDE / PyCharm**
