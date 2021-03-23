# teb-project


目標族群：想買房的人(投資者&自住者)



## 產品目標

1. 建立房價預測模型 - 內政部政府開放資料平台、台北市統計資料庫查詢系統
2. 提供所選取行政區之評分最高的正負向文 - mobile01、住展
3. 推薦房屋 - 信義房屋、永慶房仲網
4. 視覺化統計資料


## 推薦系統：輿情分析

當我們在買房子前會先上網爬文看看想買的區域中最好的建案為何？但現在的我們只能搜尋區域並一篇一篇觀看大家都是怎麼評論每一個不同建案的。==因此我們想要提供使用者在選擇區域過後即可出現大家評論前三名的建案，以及最後三名的建案==

```graphviz
graph graphname {
		rankdir=LR;
        爬取資料 -- 文本預處理;
        
        文本預處理 -- 人工Label;
        文本預處理 -- 文字向量化;
        
        人工Label -- 進入模型;
        文字向量化 -- 進入模型;
	}

```



### 文本預處理

從網路上爬下來的資料非常的雜亂，觀察這篇文章我必須刪除標點符號以及不必要的文字像是紅色框框中的文字並不會與建案評論的好壞相關，因此會將他刪除，進而得到乾淨的資料在進行評分。

![](https://i.imgur.com/Lc0ZYtO.png)

### 人工Label

在人工label的部分當中，我們幫正面評論給予label為1，負面評論給予0。

- 一個評論為單位

![](https://i.imgur.com/8u3yBGG.png =500x)

<br>


- 一句話為單位

![](https://i.imgur.com/PzASNan.png =500x)


### 文字向量化

在文字向量化之前必須先斷詞，這裡我們選用CKIP來進行斷詞。

|文本分類模型|特性|
|----|----|
|Tfidf|計算詞頻|
|word2vec|比較每個詞的相似程度|

### 代入模型

- TF-idf + 樸素貝葉斯(評論為單位)

![](https://i.imgur.com/mTVgHDW.png =300x)

> 分類分得很不清楚
> 準確度很低


- Word2vec + LSTM(評論為單位)

![](https://i.imgur.com/yAROT98.png =300x)

> 模型已經完全訓練，但預測準確度仍未提升
> 準確度提升了一些


- Word2vec + LSTM(話為單位)

![](https://i.imgur.com/f3JibTb.png =300x)

> 模型在訓練的過程中預測準確度也相度提升了
> 實際準確度上升

<br><br>

### 結論

![](https://i.imgur.com/tfjKABS.png =400x)

最後可以得知在label時給予每句話一個label會提升準確度且使用LSTM建立模型會是比較恰當的，可以想像是因為LSTM有考慮到文字前後的關聯性，因此準確度會比較高。==爾後我們選用LSTM來幫我們作文本的分類。==


<br><br>

## 推薦系統：計算歐式距離

房仲網中的資料非常的眾多，我們在看到一間符合期望的建案後，以往若想要找尋相似建案必須再從大量的房仲網中尋找，因此我們想==讓使用者在看到一間符合期望的建案時，即可以及時的找到相似的建案進而進行比較。==

```graphviz
graph graphname {
		rankdir=LR;
        爬取資料 -- 正規化;
        正規化 -- 計算歐式距離;
	}

```


<br>


歐式距離公式如下：
![](https://i.imgur.com/5WhMvsa.png  =300x)

<br>

以下以簡單的範例做示範

- 假設我們有圖中的五筆資料，此時使用者選擇第一筆資料當作推薦的目標，希望能找尋與第一筆資料較為相似的建案

![](https://i.imgur.com/v9ruH0M.png)

<br>

- 我們會以第一筆資料與其他四筆資料計算歐式距離，然而可以從中取得數值最小的建案，表示此物件與目標越相似，並將其物建(如下圖為其中的第一筆)回傳給使用者。

![](https://i.imgur.com/OphOkTG.png  =400x)


<br><br>


## 房價預測

在購買房子時，我們時常不知道這筆價錢是否合理，藉此我們提供使用者符合相關條件即可出現根據歷史資料建立模型，預測的價格區間。因為在房價的部分很難可以準確地預測到一個準確的價格，==因此我們把價錢分為五個區間，進而提供使用者在購屋時可以對符合該條件的房價區間。==

```graphviz
graph graphname {
		rankdir=LR;
        爬取資料 -- 資料清洗;
        資料清洗 -- 正規化與標準化;
        正規化與標準化 -- 帶入模型;
	}

```

### 資料清洗

- 加入新變數

除了內政部提供的資料以外另外我們也尋找了其他可能會影響房價的變數，例如：出生死亡、移出移入、房貸案件數、房價所得比、房貸負擔比、經濟成長率、消費者物價指數、M1b 貨幣供給額

以下以加入M1b 貨幣供給額為例，左邊為從網路上抓取的資料，右邊為加入變數後的資料

![](https://i.imgur.com/1oynJwo.png  =500x)



- 遺失值填補

數值型資料：平均值或眾數填補。
文字型資料：依產業知識填補，或補成其它


- label encoding

幫文字資料編碼

![](https://i.imgur.com/Kx2oemk.png  =300x)


- 離群值處理

將離群值壓至+-1.5倍IQR


- 特徵選擇
與建物單坪價格低度相關的特徵移除


### 正規化與標準化

幫資料進行正規化以及標準化

### 帶入模型

其中我們使用到分類模型。

- SVM

使用SVM結果如下

![](https://i.imgur.com/12sidp8.png  =400x)


- DNN

使用深度學習中的DNN進行分類模型，模型在訓練的過程中預測準確度也相度提升了。

![](https://i.imgur.com/C1qGyUj.png =300x)


<br><br>


## powerBI + 網頁呈現

呈現影片如右：https://www.youtube.com/watch?v=8vTNv0tklvI

遇到問題：

### 1. 必須要登入才能觀看

下面是一定要登入才能看的畫面
![](https://i.imgur.com/hRtO3mL.png)


解決辦法：

在azure建立帳號
參考網站：
http://blog.e-happy.com.tw/power-bi-%E9%9B%B2%E7%AB%AF%E5%B9%B3%E5%8F%B0%E4%B8%8A%E7%9A%84%E8%B3%87%E6%96%99%E7%84%A1%E6%B3%95%E7%99%BC%E4%BD%88%E5%88%B0-web%EF%BC%9F/

![](https://i.imgur.com/FCDjujc.png)

再次用上面的帳號登入就可以不用登入顯示在網頁上了

如圖

![](https://i.imgur.com/8jy133R.png)


<br>

### 2. 連動mySQL 排成更新

我們的使用者資料(熱門排行的部分)需要自動排成更新，因此我們需要讓他可以使用排成更新的功能。
但一開始我們沒辦法直接讓他連線本地端的mySQL，如下

![](https://i.imgur.com/hsgIpZb.png)

這裡沒有辦法選擇排成更新的選項
我們需要一個閘道來連接他
參考網站：
https://docs.microsoft.com/zh-tw/power-bi/connect-data/refresh-scheduled-refresh

下載gateway讓他連線

![](https://i.imgur.com/W1xGu28.png =300x)

這樣就建立閘道讓他可以連線了

![](https://i.imgur.com/N5wscxj.png)

在這裡設定要連接的閘道

![](https://i.imgur.com/xL3gDDF.png =400x)

就可以按下排程重新整理的按鍵了!!!!



<br>



### 3. 響應式網頁設定

把powerBI放在網頁上因為本來網頁沒有設定好，所以iframe直接放上去會跑版如下

![](https://i.imgur.com/Cgn7ugu.png =200x)

讓他在網頁css設定響應式iframe

![](https://i.imgur.com/05DykKE.png)


就可以讓它形成響應式的了!

![](https://i.imgur.com/S7LtzU2.png)



<br>

###### tags: `For gitHub`
