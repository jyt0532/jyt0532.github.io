---
layout: post
title: 矽谷找資深工程師工作心得分享
comments: True 
subtitle: 求職心得
tags: job
author: jyt0532
---

這篇文章想跟大家聊聊在矽谷找工作的心得 常常看到不少人分享畢業前找Entry Level工程師的心得
大多都是刷題刷題在刷題 但我這次找工作的半年期間 發現找資深的職缺跟找Entry Level的準備方式是完全不一樣的 於是就有了這篇文 把所見所聞分享給大家

## 前言

在Linkedin待了三年之後 我決定要找下一份工作 最主要的原因是股票快拿完了(美國的公司剛入職先給你四年RSU) Refresh的RSU是少之又少 所以入職四年之後我的薪水會有個很明顯的Cliff 甚至會比剛入職拿的還少 這簡直不能忍 於是才開始準備面試


這篇文章不會告訴你你要做什麼或應該做什麼 而是以各個小tip的方式 主要是說明資深工程師面試相對於Entry或New Grad工程師面試的差別

### Resume不太重要

首先最大的差別 如果你現在是在知名大廠工作 那Resume根本不重要 你稍微寫一下你在哪個公司哪個組待了多久就好了  至於到底做了什麼細節 HR根本就看不懂 而且大部分資深工程師的面試都會有一關叫做Project Deep Dive 你在那關把你曾做過的project準備好就好 大多數面試官面你之前根本沒看過你的履歷 就算有看也是當天早上或是面試之前五分鐘看的 所以根本也不會因為你履歷寫得好一點或不好一點影響了你面試的機會

仔細想想也是挺合理的 畢業前大家都沒經驗 只能從課程的Project中寫一些很Fancy的東西吸引HR注意 但開始工作後 其實每家都在缺人 只要稍微修改一下意思一下就好 不用花超過一小時去改履歷

### 5年經驗的要求

我面試的公司中 大多都是要5年經驗才給你面資深的職缺 比如Uber或Facebook 在台灣的經驗也算 我在台灣1年 Yahoo 1.5年 領英3年 就拿到了資深工程師的入場卷 就我所知如果你小於5年那即使你真的超強也很難給到E5/L5/T5

這通常是在HR跟你的第一通電話中 你就要把話講清楚 我就是跟Uber說我在台灣工作過 所以算五年 所以他才給我面L5A 我跟Facebook說E4我是不可能去的 有機會面E5我們再繼續談 他看了看就說好我們面試就Targeting for E5 

所以跟HR的第一通電話就要把Expectation給設好

### Coding要求下降

第二個明顯的差別 就是電面的要求變得很低 基本上只是要稍微篩檢一下不會寫程式的人 我F家電面第二個Medium題都沒寫完也過了 這在之前面E3的時候是不可能的事

### 電面小心得

補充一下F家電面 被面的題目不方便說 但我當時以為電面一小時 結果45分鐘的時候他跟我說時間差不多了 就這樣吧 我簡直震驚 哇賽還沒寫完啊 當時我在電話掛掉前 把我還沒寫完的東西加上註解 比如說

{% highlight java %}
// TODO: Add visited set
{% endhighlight %}

就這樣 面試官還說沒時間了就別問問題了掰 當時就覺得掛了 但兩天後HR打來 他問我面的如何我還跟他說我面的不好謝謝你為我做的所有安排等等 

結果他說 我們面試看好幾個面向 Problem solving, Coding skill, test, communication

面試官說Problem Solving有點struggle(就是最佳解想的有點慢) 但寫Code的速度 Code的測試的容易程度 以及Communication都還ok **於是他們就翻出了電面的Video** 當然電話本身是不錄音的 所以他們看的就是我寫程式的過程 雖然最後時間到了沒寫完 但因為我加了註解所以他們看得出來我有考慮到某些condition 就這樣拿到onsite

結論就是 即使沒時間寫 都要在電面的panel上打下註解 或是你的想法 

### 過去Project Deep Dive

這個就是修行在個人了 Dropbox面試甚至要面試前就準備好兩個Topic 面試時看面試官對哪個比較有興趣  這種你比世界上所有人都熟的東西準備起來CP值是最高的 花個幾天自己找個白板畫個圖好好練習就是了

### 面試要慎選

因為你還要上班 基本上如果所有的面試都接 你的時間一定不夠 又要顧工作又要準備面試會非常累 我當時就是幾乎所有面試都去試試 但後面才發現根本沒那麼多時間去onsite 

### 系統設計

資深工程師的重點就是系統設計面試了 我的心態就是 如果我Leetcode刷了個400+題 我應該可以拿到各個大小廠的E4 offer 但如果我的目標是E5 offer的話 那系統設計的比重要更多

我這次準備面試 系統設計花的時間超過一半 因為我這次的目標就是E5 

(八家拒 上兩家E5) > (十家E4) 

所以與其準備的廣 不如準備的深 只要十家有兩家面到你很熟練的東西就中了

分享一下最主要的準備素材

[Grokking the system design interview](https://www.educative.io/courses/grokking-the-system-design-interview): 這個教學的東西要滾瓜爛熟才行 基本上換湯不換藥 比如說QuadTree找最近的司機 或是生成Newsfeed等等的概念都要很熟

[system design primer](https://github.com/donnemartin/system-design-primer): 這我主要當作字典來查 有不熟的東西就來這裡讀一讀

[System Design Interview – Step By Step Guide](https://www.youtube.com/watch?v=bUHFg8CZFws&t=3463s): 這個人的頻道非常不錯 一個小時的影片我通常都需要暫停好幾次做筆記 常常一看就是三四個小時

[Designing Data-Intensive Applications](/toc/designing_data_intensive-application/): 不要懷疑 就是DDIA

### 寫Blog對我的影響

不是要偷渡Blog 我寫Blog也沒收錢 基本上就是我對什麼有興趣就寫什麼 想讀Effective Engineer就看Effective Engineer 想寫Design Pattern就寫Design Pattern 沒人可以規定我要寫什麼 2018年年底看到人家推薦DDIA 覺得這是本不可多得的好書 就把整本K完 我在準備面試前把我自己讀的筆記大概看了超過10遍吧 因為是自己的筆記 所以每次複習起來都很快 面試系統設計的時候就刻意跟書上的東西沾上邊 面試官就覺得你懂的很深

### 反問問題

可以[參考這篇文章](/2020/03/06/post-interview-questions/)

### 總結

這次跳槽心得跟上次已經全然不一樣 上次是G, F, L各丟一輪不面白不面 誰要我我就去哪 但工作五年之後 升到了Senior 拿到了綠卡 心態上有著全面的改變 面試已經不是公司選你 而是你選公司 你會開始有很多面向必須選擇

選組 選老闆 選年薪 選產業 選工作地點

這次是我工作五年的轉捩點 所以各個規模的公司都有面 10人的 50人的 快IPO的 剛IPO的 穩定的大公司 幾乎都面了 再三比較之下 覺得自己實力還遠遠不足  所以最後決定還是待在大廠練功 

希望在未來的某一天 也許是成為Staff 也許是Senior Staff 當我覺得我可以在一個小公司直接Report給VP或是CTO 每個禮拜開的會都是關於公司未來的走向 公司成敗與我息息相關 公司會因為我做的決定而起飛或是倒閉 的時候 我會毅然決然的進Startup拼一波 在那之前 還是繼續默默練功吧



