架構

![alt text](image.png)


開始

1. 建立新專案
```
File → New → Workspace
→ 輸入名稱（如 SixPort_Design）
→ 選擇 "Create"
```
![alt text](image-1.png)

Step 2：建立 Schematic 並設定基板 (MSub)
Workspace 建好後，要新增一個 Schematic 視窗，再放入 MSub 基板元件
```
主視窗 → File → New → Schematic
→ Cell name: 輸入 sixport_schematic
→ OK
```

在左側搜尋欄搜尋 `MSub`，將其拖曳到 Schematic 視窗中
(如果左側搜尋消失可以手動開啟 : View → Docking windows → Component palette )

- 設定參數

| 參數   | 值         | 說明                       |
|:------ | ---------- | -------------------------- |
| `H`    | `0.9 mm`   | 基板厚度                   |
| `Er`   | `4.4`      | FR4 介電常數               |
| `Mur`  | `1`        | 磁導率（保持預設）         |
| `Cond` | `4.1e7`    | 銅的電導率                 |
| `T`    | `35 um`    | 銅箔厚度                   |
| `TanD` | `0`        | 損耗正切                   |
| `Hu`   | `1e+33 mm` | 微帶線上方到金屬蓋板的距離 |


Step 3. 設定 S 參數模擬
```
Simulate → S-Parameter Simulation
→ Start:  0 GHz
→ Stop:  10 GHz
→ Step: 0.01 MHz
```



Step 4. 放置元件


在左側搜尋蘭搜索這些元件 : `MLIN` / `MBEND` / `MTEE_ADS` / `MSUB` / `TermG` / `R`

- 元件功能

![alt text](image-2.png)


> **關鍵概念** :
> 
> **W（線寬）** → 只由阻抗 Zo 和基板參數決定，**跟頻率無關**
>
> **L（線長）** → 才跟頻率有關（頻率越高，λ/4 越短）


# Stage 1 . 一階威爾金生電路 ( freq = 5.8 GHz , Zo = 50 Ω )

- Reference : 
    - 微波工程 ( 作者 : 郭仁財 )  Pg. 314-317


![alt text](149717.jpg)



ADS 內建一個專門算微帶線的工具 ( `Tools` → `LineCalc` → `Start LineCalc`)

設定基板參數 > 輸入阻抗 > 按下 按下 Synthesize 就會得到對應的線寬 W 和線長 L

( Zo = 50 Ω  , 根號2* Zo = 70.71 Ω )

![alt text](image-3.png)


Step 5. 連接元件
將元件按照電路圖連接起來，並且在適當的位置放置 TermG 和 R 元件來模擬端口和負載

- 電路圖

![alt text](image-5.png)


> 為什麼饋線 L 不需要是 λ/4？
>
> 威爾金森的工作原理只依賴中間那段 70.71Ω 的 λ/4 線來做阻抗變換。
>
> 輸入輸出的 50Ω 饋線，功能只是「把訊號從 Port 帶進來/送出去」，長度多一點少一點，只會讓訊號多走一小段路，不影響分配功能。
--------------------
> λ/4 那段不能一線畫到底，要分開來畫
>
> 核心原因：
> λ/4 線很長放不下

Step 6. 執行模擬

- 按下 "齒輪" 圖示
![alt text](image-4.png)

- 選擇剛剛建立的 S 參數模擬設定，按下 OK
![alt text](<螢幕擷取畫面 2026-05-13 161209.png>)


模擬結果

![alt text](image-7.png)
> **如何產生 `m1` 標示 :**   選取功能列 `Marker` > `New Line`

**重點** : 就算全是理論值，模擬結果也不會完美對齊中心頻率 5.8 GHz 的點。我們可以從下面幾個方法調整 : 

**Priority 1 :** 
```
# 微調 λ/4 分配臂的長度 

新 L = 舊 L × ( 目前中心頻率 / 5.8) 
```

**Priority 2 :** 
```
# 調整饋線 L 的長度

饋線 L  還沒短到完全「中性」。饋線還是貢獻了一點點額外電氣長度，把頻率稍微往低推
```



### 最終結果分析

![alt text](image-9.png)

| 參數     | 5.8 GHz 的值 | 判讀                     |
|:-------- | ------------:| ------------------------ |
| `S(1,1)` |     `-45 dB` | 輸入匹配極佳 ✅          |
| `S(2,1)` |  `-3.024 dB` | 插入損耗正常 ✅          |
| `S(3,1)` |  `-3.024 dB` | 兩路完全對稱 ✅          |
| `S(2,3)` | `-25.634 dB` | 隔離度 > 20 dB ✅ 夠好了 |
|          |              |                          |




---------------------------------------------------------

# Stage 2 : 90-degree Hybrid Coupler


