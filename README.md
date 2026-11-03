# 30 秒警報解除模擬系統（NUCLEO-G474RE 期中專題）

利用 STM32、LED、蜂鳴器及 Python 密碼輸入，模擬一套具有倒數計時、警報升級與緊急停止功能的電子控制系統。

## 1. 情境故事：誰、在哪、遇到什麼麻煩

在電子實習課的模擬情境中，玩家需要在 30 秒內解除警報。

系統啟動後，兩顆 LED 開始閃爍與呼吸，代表警報正在倒數。玩家必須透過電腦輸入正確密碼。

如果輸入錯誤，LED 閃爍速度會加快、呼吸燈亮度提高，營造越來越緊迫的效果。

如果倒數結束或輸入錯誤達 5 次，系統進入失敗鎖定狀態，兩顆 LED 全亮，蜂鳴器發出 2 秒警報聲。

此外，設置 PC13 緊急停止按鈕，可以隨時停止系統，確保操作安全。

## 2. 替身對照：故事裡的模組 → 板上替身

| 故事中的設備 | 實際使用硬體 | 功能 |
|---|---|---|
| 警示燈 | PC8 LED | 閃爍表示警報啟動 |
| 警報強度顯示 | PC6 PWM LED | 隨錯誤次數增加亮度 |
| 警報聲 | PB1 有源蜂鳴器 | 失敗時鳴叫 2 秒 |
| 密碼輸入器 | Python 命令列 | 輸入密碼並透過 UART 傳送 |
| 緊急停止開關 | PC13 板載 B1 按鈕 | 立即停止所有輸出 |
| 玩家控制按鈕 | PB0、PC7 | 預留玩家互動功能 |
| 倒數顯示器 | Python 終端機 | 顯示剩餘時間與錯誤次數 |

本專題使用 LED、蜂鳴器與程式狀態機模擬警報系統，不涉及真實危險裝置。

## 3. 功能與驗收條件：動作、預期、實測結果

| 測試動作 | 預期結果 | 目前實測 |
|---|---|---|
| STM32 開機 | 開始 30 秒倒數 | 待補紀錄 |
| PC8 主 LED | 每 500ms 翻轉一次 | 待補紀錄 |
| PC6 呼吸 LED | 逐漸變亮、變暗 | 待補紀錄 |
| 錯誤密碼 1 次 | 閃爍間隔改為 400ms | 待補紀錄 |
| 錯誤密碼 2 次 | 閃爍間隔改為 300ms | 待補紀錄 |
| 錯誤密碼 3 次 | 閃爍間隔改為 200ms | 待補紀錄 |
| 錯誤密碼 4 次 | 閃爍間隔改為 100ms | 待補紀錄 |
| 錯誤密碼 5 次 | 系統鎖定、蜂鳴器響 2 秒 | 修正後待重新測試 |
| 30 秒結束 | LED 全亮、蜂鳴器響 2 秒 | 已確認蜂鳴器正常響 2 秒 |
| 輸入正確密碼 | 停止倒數，關閉全部輸出 | 待補紀錄 |
| 失敗後輸入正確密碼 | 無法解除鎖定 | 待補紀錄 |
| 按下 PC13 | 立即停止倒數、LED 與蜂鳴器 | 待補紀錄 |

### 第三者測試

測試者不需要事先了解程式碼，只需要：

1. 依照 README 完成硬體接線與程式燒錄。
2. 開啟 Python 命令列控制程式。
3. 連接 ST-LINK 虛擬 COM 埠。
4. 重新啟動 STM32。
5. 分別測試正確密碼、五次錯誤密碼、30 秒超時及 PC13 停止功能。
6. 將實際結果填入上方驗收表。

## 4. 用到的技術：第 3～7 週勾選表

以下列出專題使用的技術，實際週次依課程投影片對照。

- [x] GPIO 輸入與輸出
- [x] EXTI 外部中斷
- [x] TIM3 計時器
- [x] PWM 脈波寬度調變
- [x] LPUART1 序列通訊
- [x] UART 接收中斷
- [x] HAL 函式庫
- [x] HAL_GetTick() 非阻塞計時
- [x] Python 與 pyserial 通訊
- [x] 狀態機控制
- [ ] ADC 類比數位轉換（未使用）
- [ ] I2C / SPI 通訊（未使用）

### 技術說明

**GPIO：** 控制 LED、蜂鳴器及讀取玩家按鈕。

**EXTI：** PC13 偵測緊急停止，並立即執行關閉輸出動作。

**TIM3 PWM：** 控制呼吸 LED 亮度，實現平滑的呼吸效果。

**USART2：** 建立 STM32 與 Python 之間的雙向通訊。

**HAL_GetTick()：** 實作 30 秒倒數、LED 閃爍及蜂鳴器 2 秒計時，避免阻塞主迴圈。

## 5. 腳位表：用到的每支腳與用途

| MCU 腳位 | 使用功能 | CubeMX 設定 |
|---|---|---|
| PC8 | 主 LED | GPIO_Output |
| PC6 | 呼吸 LED | TIM3_CH1 / AF2 |
| PB1 | 有源蜂鳴器 | GPIO_Output |
| PC13 | 緊急停止 | GPIO_EXTI13 |
| PA2 | UART TX | LPUART1_TX / AF12 |
| PA3 | UART RX | LPUART1_RX / AF12 |
| GND | 共地 | GND |

### 接線說明

**主 LED：**

PC8 → 330Ω 電阻 → LED 正極，LED 負極接 GND。

**呼吸 LED：**

PC6 → 330Ω 電阻 → LED 正極，LED 負極接 GND。

**有源蜂鳴器：**

PB1 使用 GPIO 輸出控制蜂鳴器。

目前採用不加電晶體的配置，但蜂鳴器必須符合 STM32 3.3V GPIO 的額定輸出電流及電壓規格，否則不可直接連接。

**緊急停止：**

PC13 使用開發板內建 B1 按鈕。

## 6. 建置與執行：開發環境、燒錄步驟、終端機與命令

### 開發環境

| 項目 | 使用工具 |
|---|---|
| 開發板 | STM32 NUCLEO-G474RE |
| 微控制器 | STM32G474RET6 |
| STM32 IDE | STM32CubeIDE |
| STM32 程式語言 | C |
| 函式庫 | STM32 HAL |
| 電腦端 | Python 3 |
| Python 編輯器 | PyCharm |
| 通訊套件 | pyserial |
| UART | LPUART1 |
| 通訊速率 | 115200 8N1 |

### STM32CubeMX 設定

**系統時脈**

- HSI：16MHz
- PLL：Disabled
- SYSCLK：16MHz

**TIM3 PWM**

- Channel：TIM3_CH1
- Prescaler：15
- Counter Period：999
- PWM Frequency：1kHz
- Duty 更新時間：20ms

**USART2**

- TX：PA2
- RX：PA3
- Baud Rate：115200
- Data Bits：8
- Parity：None
- Stop Bits：1

**中斷設定**

- EXTI15_10：PC13 緊急停止
- LPUART1：UART 接收

注意不要在 `main.c` 與 `stm32g4xx_it.c` 同時定義相同的中斷函式。

### 燒錄步驟

1. 開啟 STM32CubeIDE。
2. 載入專題。
3. 確認 CubeMX 腳位設定。
4. 執行 Build 編譯。
5. 使用 ST-LINK USB 連接開發板。
6. 執行 Run 或 Debug 燒錄程式。

### Python 設定

在 PyCharm Terminal 安裝：

`python -m pip install pyserial`

執行程式：

`python main.py`

Python 啟動後會列出可使用的 COM 埠，使用者選擇正確埠號（預設 COM3）。

接著按下 STM32 RESET，重新開始完整倒數。

終端機可顯示：

- `TIME`：剩餘秒數
- `ERRORS`：密碼錯誤次數
- `WRONG`：輸入錯誤
- `SUCCESS`：成功解除
- `FAILED`：警報失敗鎖定

## 7. 程式結構：哪個檔案做什麼、ISR 與主迴圈分工

### STM32 主程式

| 程式名稱 | 功能 |
|---|---|
| `main.c` | 系統主程式與初始化 |
| `MX_GPIO_Init()` | 初始化 GPIO |
| `MX_TIM3_Init()` | 初始化 PWM |
| `MX_LPUART1_UART_Init()` | 初始化 UART |
| `SystemClock_Config()` | 系統時脈設定 |
| `handle_password()` | 密碼判斷 |
| `enter_failure()` | 進入失敗鎖定 |
| `outputs_off()` | 關閉全部輸出 |
| `HAL_GPIO_EXTI_Callback()` | PC13 緊急停止 |
| `HAL_UART_RxCpltCallback()` | UART 接收中斷 |

### ISR 與主迴圈分工

**ISR（Interrupt Service Routine）**

負責即時事件處理：

- PC13 緊急停止。
- UART 接收資料。
- 設定接收完成標誌。

**Main Loop（主迴圈）**

負責持續執行：

- 30 秒倒數。
- 密碼判斷。
- LED 閃爍。
- PWM 呼吸燈。
- 蜂鳴器 2 秒計時。
- UART 狀態回報。

使用非阻塞式控制，讓多個功能能夠同時運作。

### Python 程式

`main.py` 負責：

1. 搜尋 COM 埠。
2. 與 STM32 建立 UART 連線。
3. 接收 STM32 訊息。
4. 讀取使用者輸入的密碼。
5. 傳送密碼給 STM32。

## 8. 實作照片：每張附一行圖說

### 圖 1：STM32 開發板與硬體設備

![STM32 開發板](docs/board.jpg)

圖說：NUCLEO-G474RE 開發板連接 LED、蜂鳴器與玩家按鈕。

### 圖 2：硬體接線

![硬體接線](docs/wiring.jpg)

圖說：PC8、PC6、PB1 與按鈕的實際接線方式。

### 圖 3：Python 執行畫面

![Python 終端機](docs/terminal.jpg)

圖說：Python 命令列顯示倒數時間、錯誤次數及密碼輸入結果。

**注意：** 需要先將照片上傳到 GitHub Repository 的 `docs` 資料夾，並使用上述檔名，圖片才會正常顯示。

## 9. AI 使用說明：哪裡請 AI、怎麼查核

### 使用 AI 協助的部分

本專題使用 ChatGPT 協助：

1. 整理 STM32 專題需求。
2. 產生 C 語言程式草稿。
3. 撰寫 Python UART 通訊程式。
4. 說明 STM32CubeMX 設定。
5. 解釋 PWM、UART、EXTI 及 HAL 計時原理。
6. 分析蜂鳴器計時異常。
7. 撰寫程式中文註解。
8. 整理 README 文件。

### 查核方式

程式及 AI 建議需要經過下列方式確認：

- 透過 STM32CubeMX 檢查腳位。
- 使用 STM32CubeIDE 編譯。
- 比較 UART 傳送與接收結果。
- 實際觀察 LED 與蜂鳴器。
- 測試 30 秒與 2 秒計時功能。
- 檢查 PC13 中斷是否正常。

### 發現並修正的問題

測試過程曾發現：

**30 秒倒數結束後，蜂鳴器可以正常響 2 秒，但錯誤密碼累積 5 次時，蜂鳴器只短暫響一聲。**

後續檢查程式，發現可能是主迴圈使用較早讀取的時間值，導致蜂鳴器計時判斷提前成立。

因此將蜂鳴器結束判斷改為重新取得 `HAL_GetTick()`，避免使用過期的時間值。

修正後仍需要實際燒錄與重新測試，才能確認問題完全排除。

## 10. 已知限制：哪些用替身、換真實要改哪裡

### 1. 使用模擬警報設備

本系統使用 LED 和蜂鳴器模擬警報，沒有真實危險裝置。

### 2. 倒數顯示依賴電腦

目前倒數時間主要透過 Python 顯示，沒有額外 LCD 或 OLED。

未來可增加顯示器，使系統不需要電腦也能顯示倒數。

### 3. 玩家按鈕尚未加入互動

PB0、PC7 已經預留為玩家按鈕。

未來可以設計雙人合作解除警報的功能。

### 4. 蜂鳴器驅動限制

目前使用 PB1 GPIO 控制有源蜂鳴器，適用於符合 MCU GPIO 電氣規格的低電流元件。

如果更換成高電流蜂鳴器，需要修改驅動電路。

### 5. 尚待完成的測試

五次錯誤密碼的蜂鳴器 2 秒計時、PC13 緊急停止反應，以及所有狀態切換仍需完整驗收。

### 6. 密碼與通訊安全

本專題使用示範密碼與 UART 通訊，沒有加密或身分認證機制，因此不適合用於真實保全設備。

---

**專題名稱：** STM32 30 秒警報解除模擬系統

**開發平台：** NUCLEO-G474RE

**程式語言：** C、Python

**開發工具：** STM32CubeIDE、PyCharm

**主要技術：** GPIO、EXTI、TIM3 PWM、LPUART1、HAL_GetTick()
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
