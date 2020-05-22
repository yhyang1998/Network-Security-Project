# 開發流程

1. 各自clone下來，開新的branch寫。

    #### 主要分支 (已經存在的)
    * **master** -> 穩定版 (只有確定要把這個功能加上去才把 **develop** merge到 **origin/master** 上)
    * **develop** -> 測試版 (用來另外再分支出 **feature** )

    #### 次要分支 (自己開發時新建的)
    * **myfeature** -> 新功能 （由 **develop** 直接分支，開發新功能。最後merge回 **develop branch**）

2. 兩個人開發時，記得先`git checkout -b myfeature`，切換到自己的分支(**feature**)，每次寫code...

    1. **PULL** -> `git pull origin develop`，更新本機code 
    2. **Coding** -> 修改完一個新功能，就記得`git add` `git commit`
    3. **Push** -> 切換到**develop branch** (`git checkout develop`) 利用 –no-ff 合併分支 (`git merge --no-ff myfeature`) 再來刪除 **myfeature** 分支(`git branch -d myfeature`) 最後資料上傳 (`git push origin develop`)


---
# 進度
- **4/10-4/17**
    - **預計進度**
        - [x] 用feature inverse回去看pca的重要feature有哪些?\
        *(實驗後發現 pca和我們理解的不一樣 不能選重要feature **所以先不管PCA了**)*

        - [x] 嘗試用不同的cluster演算法\
        *(發現 **DBSCAN** 能分出很多群 好像是我們想要的？)*

- **4/17-4/24**
    - **預計進度**
        - [x] 做好preprocessing 讓它能處理任意資料 (主要是**state**的部分 因為之前只針對'FIN', 'CON'兩種state處理)

        - [x] 把time feature拿掉看看 (驗證time 是否真的是重要的feature?)

        - [x] 看看DBSCAN的outlier有哪些 (驗證DBSCAN)

        - [x] 異常分析(把normal 跟 melicious分開 分別下去train 看結果如何)

    - **問題**
        1. DBSCAN離群值怎麼半
        2. 怎麼predict testing data(沒有函式可以叫 dbscan也不能直接顯示群心 寫了一個func -> DBscan_predict 但是不確定那樣對不對)
        3. PCA降為前後的group number怎麼對應
        
        
- **4/24-5/1**
    - **預計進度**
        - [x] 延續上禮拜的clustering, 用DBSCAN做anolamy analysis
        
        - [x] 對http封包跟http特徵做anolamy analysis
        
        - [x] prepossing, 可以處理有些port value為16進位的問題, 及建造0, 1數量相等的大dataset, 以利training
        
        - [x] 簡單了解DL(透過李宏毅線上課程)
        
        - [x] 訓練DNN與testing
        
    - **問題與發現**
        1. **[異常測試]** DBSCAN之anolamy analysis正確率極差(放棄此法)
        2. **[DL]** 剛開始怎麼train，在testing時表現都不佳(不管label_0或label_1 predict結果都是0)，後來發現在training data中，label 0&1 的比例不該差太多，且我們的testing dataset太小，改正這兩個問題後，**結果就好超多!!**
        3. **[DL]** epoch問題，由於initial value是隨機的，所以若epoch太小，可能會導致parameter無法optimize，故epoch值不能太大(會跑很慢)，也不能太小(可能無法converge)
        4. **[DL]** 每個epoch跑完的accuracy跟最後的[loss, accuracy]有甚麼差別?
        5. **[DL]** 參數該如何調? 應該建構哪樣的神經網路架構?
        6. **[DL]** 有normalize類別變數的正確率高於沒有normalize類別變數的，但是若沒有normalize類別變數，他中間會有一次epoch突然飆升(會部會跟initial有關)
        
     - **Anomaly anlysis整理**
        - 以label_0去train，testing data中，如果為離群值，歸類為攻擊封包
        - 分成四大項，選取全部封包、指選取http封包；參數選取全部特徵，參數指選取和http有關之特徵
        - 結果

            _                |選取全部封包|只選取http封包
            :---------------:|:--------:|:----------:
            選取全部特徵       |差        |差
            只選取http有關的特徵|好        |差
        

     - **DL整理**
        1. 使用的dataset  
            - **NUSW10000**: 從大dataset取0-10000筆的資料(0多1少，無法為training使用)
            - **NUSW20000**: 從大dataset取10001-20000筆的資料(0多1少，無法為training使用)
            - **NUSW10000-label0(1)**: NUSW10000中只有0(1)的檔案
            - **NUSW20000-label0(1)**: NUSW20000中只有0(1)的檔案
            - **NUSW10000-0**: 從大dataset取前10000筆label為0的資料
            - **NUSW10000-1**: 從大dataset取前10000筆label為1的資料
            - **NUSW_mix**: NUSW10000-0 + NUSW10000-1，共兩萬筆資料
            - **NUSW_train/NUSW_test**: 由NUSW_mix切開的，testing 有5000筆，training 有15000筆

        2. 參數(value) `此次只調整過epochs`
            - initial
            - learning rate
            - batch size
            - epochs
            
        3. 參數(function) `此次未調整`
            - activation
            - loss
            - optimizer
            
        4.  此次model都用NUSW_mix下去train，再用NUSW10000和NUSW20000去test，若要進一步分析則可以使用NUSW10000-label0(1)/NUSW20000-label0(1)
        
        5. 結果
            - **目的**: scaled, normalized, 和完全沒做數據整理的結果差別  

                **Training Data**: NUSW_mix  

                **Testing Data**: NUSW10000  

                *PS: Scaling 和 Normalizing 的差別只在於**對哪些特徵做normalize***  


            **Scaling/Normalizing比較**
            _        |選取特徵數|優缺點
            :---------:|:------:|:---:
            Scaling    |所有    |能將所有資料都固定在一定區間，但沒考慮到類別型態資料
            Normalizing|自己決定 |不normalize類別資料，不會造成資料失真 

        
            **使用在DL裡的結果**
            方法       |結果(正確率)  |其他發現
            :--------:|:-----------:|:----:
            Scaled    |0.985        |在5 - 7個epoch間正確率突然顯著上升(0.5-0.9)
            Normalized|0.908        |1 - 10個epoch之正確率接平滑上升
            Do Nothing|0.671        |最終結果不佳

            - **目的**: 對不同Testing data的成果  

                **數據處理方式**: Scaling  


            方法          |結果(正確率)  
            :-----------:|:-----------:
            mix_and_10000|0.985
            mix_and_20000|0.957
     - **Meeting 結論**   
        1. 後來發現正確性會那麼高是因為training 和 testing data重複性很高。若用不相干的training和testing data測會發現accuracy很低。  
        為了改進performance，**減少每一層的unit數**(`Dense(units=100, activation='sigmoid')` unit **由原本的500減為100**，如此一來可增加每個unit在back propagation的影響性)，還**增加深度**，並將**optimizer改成adam**，但最終成果準確率會一直浮動，模型有待加強。

- **5/1-5/8**
    - **預計進度**
        - [x] 寫好自動分training和testing的code
        - [x] 使用其他training和testing data去測試model，如果效果不彰，可以試著更改model的參數(unit, activation function, loss, optimizer)，觀察結果 (找epoch的threashold)
        - [x] Survey isolation function
        - [x] Add EarlyStopping Callbacks

    - **問題與發現**
        1. 使用`train_test_split`切分train和test後會改動到原本的data type(把dataframe 改成 numpy)
        2. 由於是random切training和testing，所以如果在preprocessing時做one-hard-encoding 會有兩個不同的問題。
            - 若是事先定好要轉換的項目，會有些項目沒有被one-hard encode到
            - 若是沒事先定好要轉換的項目，可能會造成testing和training最終feature數不同，而model不能fit
        3. 用新的dataset測，發現連training都train不好
        
    - **更改參數實驗結果**
        1.
        - **experiment**
            - 參數：  

            dataset|activation     | layer type  | layer num|input units per layer| loss | optimizer| batch size | epochs 
            :----:|:-------------:|:-----------:|:------------:|:---------------:|:----:|:--------:|:------------|:---:
            NUSW-mix|sigmoid/softmax|  dense      | 10           |100              |mse   | adam     | 100         | 10  
                
            - result:
            
             Scaled  | 結果(正確率)
             :------:|:----:
             y       | 99.8%!!!?

        - **control - dataset**
             dataset    | 結果(正確率)
             :---------:|:----:
             NUSW-mix-4 | 49.1%

             ***PS: 所以後來就都用NUSW-mix-4當測試組了看看能不能把正確率提升***
        - **control - loss function**
            
             Loss   | 結果(正確率) - Train | 結果(正確率) - Test
            :------:|:-----:|:-----:
            mse     | 50.2% | 49.1%
            categorical_hinge | 50.2% | 49.1%       
            squared_hinge |  50.2% | 49.1%       
            hinge  |  50.2% | 49.1%
            logcosh  |  50.2% | 49.1%
            categorical_crossentropy|  50.2% | 49.1%     
            binary_crossentropy     |        
            sparse_categorical_crossentropy|   
            kullback_leibler_divergence|       
            poisson                 |        
            cosine_proximity        |        
                
        - **control - activation function**


                Activation       |結果(正確率) - Train | 結果(正確率) - Test
                :----------------:|:------:|:----------:
                sigmoid           |50.2% | 49.1%
                hard_sigmoid      |50.2% | 49.1%     
                tanh              |50.2% | 49.1%     
                relu              |50.2% | 49.1% 
                selu              |50.2% | 49.1% 
                elu               |50.2% | 49.1% 
                softplus          |50.2% | 49.1% 
                softsign          |50.2% | 49.1% 
                
                
        - **control - optimizer**
                網路上都說adam好，還要試其他的嗎?
                
            
     - **Isolation forest**
        - Dataset:
            - train : NUSW_mix_4其中的40000筆label為0的data
            - test : NUSW20000(0,1 mix)
        - Result: 0.75-0.9 (結果會浮動)
- **5/07-5/14**
    - **進度**
        - [x] 用上上次train的不錯的model去test
        - [x] 將flow feature 拿掉看看
        - [x] LSTM model
        - [x] coding style
        
    - **Flow資訊**
        - 用以前寫過的get_imp
        - 經過篩選後只剩下```'srcip', 'sport', 'dstip', 'dsport', 'proto','state', 'dur', 'sbytes', 'sttl', 'Sload', 'Dload', 'swin', 'dwin', 'stcpb', 'res_body_len', 'Stime', 'service', 'Dintpkt', 'is_sm_ips_ports', 'is_ftp_login','attack_cat', 'Label'```
        
    - **變數命名**
        - 類別(train/test/trainlabel/testlabel)_型態(df/np/list)
    
    - **資料集命名**
        - 原始資料為第幾個資料集_起始-終點_label特性(mix/label0/label1)_時間特性(time)
        ***筆數之後會以10000為單位***
        
    - **結論**
        - 有沒有拿掉flow feature不影響結果，所以代表我們只能針對一個一個flow分析，無從針對單一封包。
        - DNN -> 0.5
        - RNN -> 0.87，但會把所有封包判給label0，很像以前資料集中0跟1不均等的情況，但在rnn time-based的概念下，我們的資料即無法取0跟1均等，應該是要依時間順序取
        
- **5/14-5/22**
    - **進度**
        - [x] 修好dnn和rnn的bugs(把nan成0), 並進一步發現他們各自的用途
        - [ ] 讀取pcap檔
        - [x] 專題展簡易流程圖

    - **資料集命名(更改5/7-5/14內容)**
        (依照**是否依照時間順序抓取資料**來決定命名方式)
        - **是**: 原始資料為第幾個資料集_起始-終點_label特性(mix/label0/label1)_時間特性(time)
        ***ps:筆數以10000為單位***
        *(**ex:** 假定資料來自NB15-1, 從第0筆到第1萬筆, label0, 1混合(mix), 則資料集名稱為: **1_s0-e1_mix_time** )*
        - **否**: 原始資料為第幾個資料集_0w(+有幾筆)_1w(+有幾筆)_是否shuffle(yshf/nshf)_時間特性(notime)
        *(**ex:** 假定資料來自NB15-1, 0,1 各取1萬筆資料(共兩萬筆), 沒把0,1 shuffle, 則資料集名稱為: **1_0w1_1w1_nshf_notime**)*
        
    - **問題與發現**
        - 

---
### *補充：Clone fork 差別*

- **Clone** : 把專案在遠端儲存庫上的所有內容複製到**本地**，建立起**本機儲存庫**及工作目錄

- **Fork** : 把別人專案的遠端儲存庫內容複製一份到自己的**遠端儲存庫**

- **使用方法** : 如果在開發者在GitHub上看到有興趣的專案，可以執行Fork指令，把別人專案的遠端儲存庫複製到自己的遠端儲存庫，再執行Clone指令，把自己遠端儲存庫的整個專案的所有內容（包括各版本）複製到本機端儲存庫。

---
### *參考資料*

- [git branch](https://blog.wu-boy.com/2011/03/git-%E7%89%88%E6%9C%AC%E6%8E%A7%E5%88%B6-branch-model-%E5%88%86%E6%94%AF%E6%A8%A1%E7%B5%84%E5%9F%BA%E6%9C%AC%E4%BB%8B%E7%B4%B9/)

