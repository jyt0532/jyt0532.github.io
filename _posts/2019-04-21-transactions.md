---
layout: post
title: Designing Data-Intensive Application - Transactions - ACID
comments: True 
subtitle: 事務 - ACID
tags: systemDesign 
author: jyt0532
excerpt: 本篇文章介紹事務以及ACID的概念
---

這是Designing Data-Intensive Application的第二部分第三章節的Part1: ACID


筆者註: 這個章節書中敘述的方式太過凌亂 對於初學者非常吃力 筆者改變了介紹的方式甚至例子都做了更改 在本部落格中將事務一章分為三篇文章

事務Part1 - ACID

事務Part2 - 弱隔離級別

事務Part3 - 可串行化

本篇是系列文的Part1

本文所有圖片或代碼來自於原書內容
{% include copyright.html %}


# 事務

在數據系統的現實世界中 很多事情都可能出錯

1.數據庫軟硬體都可能在任意時刻發生故障(包括寫操作進行到一半時)

2.應用程序可能在任意時刻崩潰(包括一系列操作的中間)

3.網路中斷可能切斷數據庫和應用的連接

4.多個客戶端可能會同時寫入數據庫 覆蓋彼此的改動

5.客戶端可能讀取到無意義的數據 因為數據還沒更新完

6.客戶之間的Race condition 導致錯誤

為了達成**可靠性** 系統必須處理這些故障 確保它們不會導致整個系統的災難性故障 但是這容錯機制的工作量太大 必須仔細考慮所有可能狀況 並仔細測試

數十年來 事務(Transaction)一直是簡化這些問題的首選機制

事務是應用程序將**多個讀寫操作組合成一個邏輯單元**的一種方式 事務中的所有讀寫操作被視作單個操作來執行 
整個事務要馬成功提交(commit) 要馬失敗(abort) 失敗後需要roll back同一個事務所有的讀寫操作 並且可以安全地重試

問題變得簡單很多 因為我們不用再擔心**部分失敗**

當今市面上的數據庫 都已經把事務視為理所當然 但充分理解他也有好處 因為某些情況 你可以選擇性地弱化事務保證 或是完全放棄事務保證(比如說達到更高性能或更高可用性) 

那要怎麼知道你的應用到底需不需要事務呢 你需要理解事務可以提供的安全保障以及代價

本章將研究許多出錯案例 並探索數據庫用於防範這些問題的算法 還會提到許多並發控制的領域 和各種race condition 以及數據庫如何實現read committed, snapshot isolation和serializability

本章同時適用於單機數據庫與分佈式數據庫 下一章會重點討論僅出現在分佈式系統中的特殊挑戰

## 事務的重要概念

我們先了解一下 事務帶給我們的保證

### ACID

ACID代表原子性(Atomicity) 一致性(Consistency), 隔離性(Isolation)和持久性(Durability) 它由TheoHärder和Andreas Reuter於1983年創建 讓大家可以在容錯機制中建立精確的術語

#### 原子性(Atomicity)

當客戶想進行多次寫入 但在一些寫操作處理完之後出現故障的情況(比如進程崩潰 網路中斷 瓷碟滿了等等) 如果這些寫操作被歸類到同一個原子事務中 而這個事務不能成功提交的話 則該事務必須被中止 並且數據庫必須丟棄或撤消該事務中所做的任何寫入

如果沒有原子性 在多處更改進行到一半時發生錯誤 很難知道哪些更改已經生效 哪些沒有生效 有了原子性之後 只要失敗我們就是重跑一次 不會有副作用

原子性指的是 能夠在錯誤時中止事務 丟棄該事務進行的所有寫入變更的能力

#### 一致性(Consistency)

很不幸的 光在本書中 這個詞就有四個意思

1.在[複製](/2019/02/12/replication/)文中 我們討論了副本一制性以及異步複製系統中的最終一致性問題(參閱[複製延遲問題](/2019/02/12/replication/#複製延遲問題))

2.Consistent Hash 是某些系統用於重新分區的一種分區方法

3.在CAP定理中 一致性一詞用於表示可線性化

4.在ACID中 一致性是指數據庫在應用程序的特定概念中處於 *良好狀態*

良好狀態的意思是 **對數據的一組特定陳述必須始終成立** 比如說在執行事務前 一個銀行所有戶頭的錢加起來是X 那跑過事務後 這個不變量維持不變 不會說A給B 1000塊 B的戶頭已經加1000了可是A的沒扣

要注意的是 一致性的這種概念取決於應用程序對不變量的觀念 今天你想要A +1000但是B -500也沒有人可以阻止你 因為數據庫只管存儲

當初在ACID的論文中 C是被硬丟進去湊單詞的XD

#### 隔離性(Isolation)

大多數數據庫都會同時被多個客戶端訪問 如果它們各自讀寫數據庫的不同部分那當然沒什麼問題 但如果他們訪問的是相同的對象 那就可能會遇到並發問題或race condition

![Alt text]({{ site.url }}/public/DDIA/DDIA-7-1.png)

假設你有兩個客戶端同時在數據庫中增長一個計數器 每個客戶端需要讀取計數器的當前值 加 1  再回寫新值 原本應該要是44的結果卻是43

隔離性意思是 **同時執行的事務是相互隔離的** 傳統的數據庫教科書將隔離性形式化為可串行化(Serializability) 意味著每個事務可以假裝它是唯一在整個數據庫上運行的事務

可是實踐上這是很困難的而且開銷非常大 所以現實生活中 我們會在Isolation這個面向做出取捨 我們會在[Part2](/2019/04/29/weak-isolation-levels/)中討論這個問題

#### 持久性(Durability)

持久性 是一個承諾 即一旦事務成功完成 即使發生硬件故障或數據庫崩潰 寫入的任何數據也不會丟失

單點的數據庫中 持久性通常意味著數據已被寫入硬碟或是SSD 多點的數據表示 持久性可能意味著數據已成功複製到一些節點 為了提供持久性保證 數據庫必須等到這些寫入或複製完成後 才能報告事務成功提交

別忘了 完美的持久性是不存在的 如果所有硬碟跟備份同時被銷毀 那也沒有任何數據庫救得了

> **複製和持久性**
>
> 現實生活比理論 可能還要更加不完美
> 
> 1.如果你寫入硬碟然後機器掛了 即使數據沒有丟失 在修復機器或將磁盤轉移到其他機器之前 也是無法訪問
>
> 2.一個故障(停電等等)可能會讓你所有內存的資料不見 故內存數據庫仍然要和硬碟寫入打交道
>
> 3.在異步複製系統中 當主庫不可用時 最近的[寫入操作可能會丟失](/2019/02/12/replication/#處理節點停機)
>
> 4.就連硬碟上的數據可能會在沒有檢測到的情況下逐漸損壞
> 
> 5.SSD運行的前四年 30％到80％的硬盤會產生至少一個bad block
>
> 6.如果SSD斷電 可能會在幾週內開始丟失數據 具體取決於溫度
>
> 所以現實生活中 沒有一種技術可以提供絕對保證 只有各種降低風險的技術 最好抱著懷疑的態度接受任何理論上的**保證**
	
### 單對象和多對象操作	

原子性和隔離性描述了客戶端在同一事務中執行多次寫入時 數據庫要做的事

原子性: 如果在一系列寫操作的中途發生錯誤 則應中止事務處理 並丟棄當前事務的所有寫入 保證all-or-nothing

![Alt text]({{ site.url }}/public/DDIA/DDIA-7-3.png)

以上圖來說 原子性確保發生錯誤時 此事務先前的任何寫入都會被撤消 以避免狀態不一致


隔離性: 同時運行的事務不應該互相干擾 如果一個事務進行多次寫入 那其他事務要馬看到全部的寫入結果 要馬全都看不到

如果你想同時修改多個對象 通常需要多對象事務(multi-object transaction) 來保持多數個數據是同步的

假設我們現在想找一個用戶未讀的email

{% highlight sql %}
SELECT COUNT (*) FROM emails WHERE recipient_id = 2 AND unread_flag = true
{% endhighlight %}

可是每次都跑這個有點太慢 我們可以另外存一個計數器 每收到一個新信 計數器加一 每讀一封信 計數器減一

但下列情況發生了
![Alt text]({{ site.url }}/public/DDIA/DDIA-7-2.png)

對User2來說 郵件列表裡顯示有未讀消息 但計數器顯示為零未讀消息

這就是為什麼我們需要隔離性 User要馬同時看到新郵件和增長後的計數器 要馬都看不到 不會看到執行到一半的中間結果

所以我們再處理多對象事務的時候 需要有個明確的定義 來知道哪些讀和哪些寫是屬於同一個事務 在任何特定連接上 `BEGIN TRANSACTION` 和 `COMMIT` 語句之間的所有內容 被認為是同一事務的一部分

#### 單對象寫入

但針對單獨對象的寫入 原子性和隔離是適用 比如你正在向數據庫寫入20KB的json檔案:

1.如果發送第一個10KB之後網路中斷  數據庫是否存儲了不可解析的10KB JSON片段

2.如果數據庫正在覆蓋Disk上的前一個值的過程中 電源發生故障

3.如果另一個客戶端在寫入過程中讀取該文檔

這些特殊的狀況都不好處理 所以存儲引擎對於單個對象也需要提供原子性和隔離性

原子性可以透過log來修復任何崩潰情況(參閱[讓B-tree更可靠](/2019/01/19/storage-and-retrieval/#讓b-tree更可靠)) 或是每個對象都有個獨立的鎖 一次只有一個thread可以訪問一個對象

某些數據庫也提供複雜一點的原子操作 比如說[比較和設置(CAS, compare-and-set)](#) 當值沒有被其他人修改過時 才允許執行寫操作

#### 多對象事務的需求

許多分佈式數據存儲已經放棄了多對象事務 最主要的原因是**因為多對象事務很難跨分區實現** 更不用說想達到高可用或高性能

但我們到底需不需要多對象事務呢?

大多數情況 對於單獨對象的插入更新刪除就已經足夠 但是有許多場景需要協調寫入幾個不同的對象

1.在關係數據模型中 常常有foreign key的概念 多對象事務使你確信這些引用始終有效 如果這些關聯不存在 那關係數據模型就失去了保證

2.圖數據模型中 一個頂點有著到其他頂點的邊 多對象事務使你確信這些引用關聯有效

3.在文檔數據模型中 通常需要一起被更動的字段都在同一個文檔中 這被視為單個對象 但是缺乏連接功能的文檔會鼓勵非規範化(denormalization)

4.具有二級索引的數據庫中 每次更改值時都需要更新索引 這些索引就是不同的數據庫對象

當然以上的應用都可以在沒有事務的情況下實現 然而沒有原子性 錯誤處理就複雜得多 而沒有隔離性 就會導致並發問題

#### 處理錯誤和處理中止

事務的一個關鍵特性是 如果發生錯誤 它可以中止並安全地重試

所以ACID數據庫有著以下的哲學:

> 如果數據庫有違反其原子性 隔離性或持久性的危險 則寧願完全放棄事務 而不是留下半成品

註: 有一個系統例外 就是如果是[無主複製](/2019/02/12/replication/#無主複製)的系統 這個系統主要是在盡力而為的基礎上進行工作 所以如果這個系統出錯了 從錯誤中回復是**應用程序的責任**

儘管重試一個中止的事務是一個簡單而有效的錯誤處理機制 但還是有缺點

1.有可能事務的處理實際上成功了 但是你的server要跟用戶回傳成功訊息這一步網路斷了 所以客戶以為提交失敗 那客戶端重試的話會導致一個事務被執行兩次

2.如果錯誤是由於負載過大造成的 則重試會讓問題變得更糟 你通常需要exponential backoff重試機制 去單獨處理與過載相關的錯誤

3.只有在發生臨時性錯誤(死鎖, 異常情況, 臨時性網路中斷, 故障切換等等) 才需要重試 如果是永久性的錯誤(違反約束)的話重試是毫無意義的

4.如果事務在數據庫之外也有副作用 那即使事務中止 也可能發生這些副作用 常見例子是發電子郵件提醒 如果你想確保幾個不同的系統一起提交或放棄 [二階段提交](/)可以提供幫助

## 總結

本文中我們討論了ACID的各個意思 但其中一個保證開銷非常非常大 那就是Isolation隔離性的保證

完全的事務隔離 稱為可串行化(Serializability) 需要花費非常多的心力 我們會在[Part3](/)中討論可串行化

但除了可串行化之外 我們有一些稍微弱化的保證 犧牲一點隔離性換取高一點的性能


以下是四種不同的隔離程度 跟各種隔離程度可能會發生的並發問題

| 隔離級別 | Dirty Write | Dirty Read | Non-repeatable read  | Phantom read
| --- | --- | --- | --- | --- |
| None | O | O | O | O |
| Read Uncommited      | X | O | O | O |
| Read Commited(Non-repeatable read) | X | X | O | O |
| Repeatable read      | X | X | X | O |
| Serializable      | X | X | X | X |

我們在[Part2](/2019/04/29/weak-isolation-levels/)會探討各個隔離級別以及實踐方式

