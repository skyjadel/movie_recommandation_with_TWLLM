# 搭載電影推薦系統的LLM

## 專案概述
這個專案分成兩個部分

一個是基於 <a href= 'https://grouplens.org/datasets/movielens/'>https://grouplens.org/datasets/movielens/</a> 的 25M-MovieLens 資料所建構的電影推薦系統 <br>
這個資料集包含了兩千五百萬筆由十幾萬個使用者給予六萬多部電影的評分，0.5分最低，5.0分最高 <br>
我使用了其中的約2260萬筆資料來建構一個推薦系統，這個推薦系統透過詢問使用者喜歡的電影 <br>
進而推估使用者可能會喜歡哪些他沒有提到的電影 <br>

另一個部分則是利用 RAG 技術將推薦系統與大型語言模型串接起來 <br>
在這個部分，LLM必須先判斷使用者是否有推薦電影的需求，如果有，則進入推薦電影模式 <br>
推薦系統會從 MovieLens 資料集裡面最受好評的 300 部電影當中隨機選取 25 部列給使用者，請使用者選出 1~3 部喜歡的電影 <br>
推薦系統再以這些電影為依據，從預先算好的推薦資料庫當中查詢喜歡這些電影的人可能也會喜歡的電影的綜合評分 <br>
然後選出推薦片單，做為 prompt 交給 LLM，再讓 LLM 產生推薦文輸出給使用者

我由於要讓整個系統可以以台灣中文運作，所以我選擇了由台大資工林彥廷團隊以繁體中文語料 finetune 過的 Llama-3-Taiwan 模型 <br>
我選擇的是 8B 個參數的版本，可以在我的本機以 float16 精度執行

## 檔案說明

### ChatBot_With_RAG.ipynb
專案的主程式，執行能夠推薦電影的對話機器人

### Recommandation_cross_score_calculation.ipynb
預先計算推薦資料庫的程式，負責讀取 grouplens.org 所提供的電影評分資料集，並計算出裡面電影之間的高分交集程度。

### rating.csv, movies.csv & user.csv
25M-MovieLens 資料集的原始檔案，rating.csv 存有 25M 筆電影評分，欄位包含使用者ID，電影ID，評分與時間戳記 <br>
movies.csv 則儲存了電影ID與電影之間的對應關係，欄位包含電影ID，電影標題，類型，年份與上線時間戳記 <br>
rating.csv 檔案太大 (587MB)，放不上GitHub，請到 <a href='https://drive.google.com/file/d/1Y3IKPQv0vKC3xhlMyZBhUx6Jd3eSWBYE/view?usp=sharing'>https://drive.google.com/file/d/1Y3IKPQv0vKC3xhlMyZBhUx6Jd3eSWBYE/view?usp=sharing<a> 下載

### hundred_likers.csv
列出所有至少拿到 100 個 4 分以上高分的電影，欄位包含電影ID，電影標題，年份，高分次數，高分率 <br>
其中高分率為有機會為這部電影評分的使用者當中，有多少比例的人給了它高分。 <br>
這邊「有機會為這部電影評分的使用者」的認定標準為：在這部電影上線之後，有為任何電影評過分的使用者，被認定為「有機會」為這部電影評分。 <br>
舉例來說，2010年上映的「全面啟動」在 162541 個使用者當中，拿到 29620 個高分評價 <br>
然而在這個資料集當中，全面啟動上映之後仍有評分紀錄(不限於給全面啟動評分)的使用者只有 64625 人 <br>
因此全面啟動的高分率為 29620 / 64625 = 45.8% <br>
這個設計是為了讓較新上映而沒有時間累積大量評分的電影，能與上映 20~30 年以上的電影有公平競爭的機會

### best_300.csv
前面 hundred_likers.csv 當中，高分率佔前 300 名的電影 <br>
在整個系統當中做為提示使用者喜歡的電影的題庫存在

### cross_scores.db
由 Recommandation_cross_score_calculation.ipynb 產生的，包含電影之間交叉分數的 sqlite database 檔案 <br>
rating.csv 檔案太大 (120MB)，放不上GitHub，請到 <a href='https://drive.google.com/file/d/1ANEeGekSrWQyzWnDHKSxF5CUE8wFPvG6/view?usp=sharing'>https://drive.google.com/file/d/1ANEeGekSrWQyzWnDHKSxF5CUE8wFPvG6/view?usp=sharing<a> 下載

## 需求套件與硬體
比較特殊的有： <br>
pytorch, transformers <br>
另外還需要： <br>
pandas, matplotlib, numpy, json, sqlite3，另外 code 裡面有用到執行進度條套件 tqdm，如果不想裝就要去 code 裡面改掉。 <br>
硬體需求： <br>
以 float16 精度執行 Llama-3-Taiwan-8B 模型，需要 20GB 以上的 VRAM 比較保險。







