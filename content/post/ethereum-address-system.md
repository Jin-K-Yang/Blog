---
title: "公鑰、私鑰、地址傻傻分不清！？"
author: Ken
date: 2023-02-19T19:35:41Z
draft: false
tags: [
    "blockchain",
    "Ethereum",
    "address",
]
math: true
coverImage: "https://miro.medium.com/max/875/0*n4rSGgp77ube9Y6Y"
---
{{< math.inline >}}
{{ if or .Page.Params.math .Site.Params.math }}
{{ partial "math.html" . }}
{{ end }}
{{</ math.inline >}}

在區塊鏈的世界中，我們時常會用到私鑰簽署交易，但我們真的理解私鑰的意義嗎？而公鑰又是甚麼？和地址又有甚麼區別呢？然而每條區塊鏈所用的 加密方式都不盡相同，本文將會以 Ethereum 為例深入探討這些看似簡單卻又複雜的問題。

![](https://miro.medium.com/max/875/0*n4rSGgp77ube9Y6Y)

<center>Photo by  <a href="https://unsplash.com/@jasondeblooisphotography?utm_source=medium&utm_medium=referral">Jason D</a> on <a href="https://unsplash.com/?utm_source=medium&utm_medium=referral">Unsplash</a></center>

## 私鑰 Private Key

### 首先，我們先來探討甚麼是私鑰？ ###

> It is assumed that the sender has a valid private key pr, which is a randomly selected positive integer (represented as a byte array of length 32 in big-endian form) in the range [1, secp256k1n − 1].

{{< math.inline >}}
上面是引用  <a href="http://gavwood.com/paper.pdf">Ethereum 黃皮書</a>對私鑰的介紹，其中 \(\ secp256k1n = 2^{256}\) 。私鑰其實就是一串隨機數，具體而言，以太坊的私鑰是一串隨機正整數，而這個值的區間位於 \(1\) ~ \(2^{256}\) 之間，也就是說，它是一串隨機生成的 256 bits 序列。這個 range 是非常巨大的，要生成兩個一樣的私鑰是一件幾乎不可能會發生的事。（如果你無法想像 \(2^{256}\) 實際到底有多大，可以觀看以下影片）
{{</ math.inline >}}

{{< youtube S9JGmA5_unY >}}
<center>2²⁵⁶ 實際到底有多大？</center>

### 而私鑰為甚麼會如此重要呢？ ###

在真實的世界中，「**擁有**」的定義是領有、具有，得到或者保持著某種東西，你可以透過出示地契證明你擁有某一棟房子。但在區塊鏈的世界中，**擁有**某個東西只不過是在帳本上將你的資產做加法，並且用數學的方式**證明**它的存在，當全世界的人都**認可**這個紀錄，那麼你就**擁有**它，儘管你其實並沒有實際擁有任何東西。簡單來講，當你知道你的私鑰的時候，你就**擁有**你所有的貨幣資產，包括 NFT 和 Token，因為多數人都能夠**證明**這條紀錄是千真萬確的。

私鑰可以簽署交易，就像在現實生活中需要簽名才能夠轉帳一樣，簽署過後的交易才能夠對你的帳戶資產進行更改。所以雖然在區塊鏈中大家都可以產生一筆交易更動你的資產，但是只有擁有私鑰的人才能夠簽署這筆交易，交易才能被認可和被驗證。

> 在現實世界中可以捏造一個假的簽名，完成交易，但是在區塊鏈的世界中，因為它底層的密碼學特性，這件事情是不可能發生的，目前沒有一個人可以在沒有私鑰的情況下簽署交易。

> Not Your Keys, Not Your Coins

這也是上面這句格言的由來，如果有人**知道**你的私鑰，那麼他就**擁有**你所有的資產了，許多駭客就是透過釣魚信件和瀏覽器快取在你沒發現的情況下竊取你的私鑰，進而盜走你的資產。

## 公鑰 Public Key

{{< math.inline >}}
在介紹公鑰之前須要先說明一下甚麼是<b>單向函數（One-way function）</b>，單向函數指的是輸入一個 input 到函數 \(\ f(x)\) ，會很容易地得到 output，但是只給定 output 卻很難反向計算出 input。舉個例子，我們假設 \(p\), \(q\) 為質數，並且是非常大的數，對於 \(\ n = p*q\) ，給定一個 \(p\) 和 \(q\) 可以很容易得到 \(n\) ，但給定一個 \(n\) 卻很難得出 \(p\) 和 \(q\) 。
{{</ math.inline >}}

> 注意：單向函數很難反向計算，但我們沒辦法證明它真的很難計算，我們只能說目前沒有人可以反向計算出來而已，不能保證明天還是沒有人計算出來，因此「單向函數真的存在嗎？」是一個目前為止都沒有辦法回答的問題。


<center>{{< figure src="https://miro.medium.com/max/750/1*upDx16KnvrMrGQBmzJ1SjA.png" link="https://computersciencewiki.org/index.php/One-way_function" title="One way function" >}}</center>

在 Ethereum 中，私鑰是用來將訊息**簽名**的，而公鑰是用來**驗證簽名**的，所有人都可以獲取你的公鑰以驗證簽名。Ethereum 的**公鑰**是由**私鑰**經過**橢圓曲線演算法** [**ECDSA**](https://en.wikipedia.org/wiki/Elliptic_Curve_Digital_Signature_Algorithm)**(Elliptic Curve Digital Signature Algorithm)**  算出來的，得出的結果長度為 512 bits，其演算法使用橢圓曲線（Elliptic curve）這個**單向函數**。因此透過私鑰轉換成公鑰非常容易，但是要透過公鑰轉換成私鑰是一件非常困難的事，這也恰恰符合前面所說的私鑰不能隨意洩露。

> ECDSA 背後牽涉到很多艱深的數學，因此本篇文章不會討論他的原理，未來會另外寫一篇有關 ECDSA 的文章加以探討。

## 地址 Address

地址就像銀行帳號，是 Ethereum 中用來代表一個帳戶的方式，任何交易都是由一個地址發起的，而根據 Ethereum 黃皮書中對於地址的定義如下：

> For a given private key, the Ethereum address A is defined as the rightmost 160-bits of the Keccak hash of the corresponding ECDSA public key.

Ethereum 的地址是由公鑰生成的，先將公鑰代入 Keccak hash function 中，這個函數是一個單向函數，其 output 長度為 256 bits，Ethereum 取最右邊的 160 bits 作為帳戶地址。

然而你可能會問為何需要地址呢？其實在 Bitcoin 的白皮書裡是沒有地址這個概念的，是後來 Ethereum 為了讓帳戶位址更容易記住才有地址這個概念的。（壓縮過後的公鑰為 256 bits vs. 地址 160 bits）

### 地址有可能重複嗎？ ###

{{< math.inline >}}
答案是有的，但是機率非常小，小到幾乎不可能，Ethereum 地址長度為 160 bits，代表有 \(2^{160}\) 種可能，根據<a href="https://zh.wikipedia.org/zh-tw/%E7%94%9F%E6%97%A5%E5%95%8F%E9%A1%8C">生日問題</a>，當有 \(2^{80}\) 個不同的地址被創造出來時，地址重複的機率會上升到 50%。然而要產生 \(2^{80}\) 個不同的地址，需要地球上的每一個人每秒都能夠產生一個新的地址，持續產生八百萬年才有可能。
{{</ math.inline >}}

## Conclusion

上面我們講述了私鑰、公鑰以及地址是如何產生的，以及個別的功用是甚麼，下圖簡單的總結了從私鑰變成地址的過程。雖然省略了中間的數學過程，但是希望大家看完本文能夠對於區塊鏈的奧妙更加了解。

![](https://miro.medium.com/max/875/1*1RkWbm58snAEi325ltzbDA.png)

[https://medium.com/@codetractio/inside-an-ethereum-transaction-fa94ffca912f](https://medium.com/@codetractio/inside-an-ethereum-transaction-fa94ffca912f)

## Reference

-   [Part One: Understanding Private Keys](https://medium.com/portis/part-one-understanding-private-keys-311389737fbe)
-   [Part Two: Turning Random Numbers into an Ethereum Address](https://medium.com/portis/part-two-turning-random-numbers-into-an-ethereum-address-3928f56b225c)
-   [Ethereum Yellow Paper](http://gavwood.com/paper.pdf)
-   [Secp256k1-domain-parameters](https://www.cs.utexas.edu/users/moore/acl2/manuals/current/manual/index-seo.php/ECURVE____SECP256K1-DOMAIN-PARAMETERS#:~:text=Domain%20parameters%20of%20the%20secp256k1,the%20curve%2C%20as%20nullary%20functions.)
-   [Ethereum Address vs Public Key](https://ethereum.stackexchange.com/questions/33171/ethereum-address-vs-public-key)
-   [Account uniqueness guaranteed?](https://ethereum.stackexchange.com/questions/4299/account-uniqueness-guaranteed)