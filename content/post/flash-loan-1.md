---
title: "無須抵押品就可以貸款？利用 Aave 實作閃電貸智能合約（上）"
author: Ken
date: 2023-03-12T19:51:58Z
draft: false
tags: [
    "Blockchain",
    "Ethereum",
    "Defi",
    "Security"
]
coverImage: "https://images.unsplash.com/photo-1521897258701-21e2a01f5e8b?ixlib=rb-4.0.3&q=80&fm=jpg&crop=entropy&cs=tinysrgb&w=3600"
---
在 Defi 的領域中，閃電貸扮演著一個非常重要的角色，他是一把雙面刃，本系列將分成上下兩篇文章分別介紹閃電貸以及如何利用去中心化借貸平台 Aave 來實作閃電貸智能合約。

![https://images.unsplash.com/photo-1521897258701-21e2a01f5e8b?ixlib=rb-4.0.3&q=80&fm=jpg&crop=entropy&cs=tinysrgb&w=3600](https://images.unsplash.com/photo-1521897258701-21e2a01f5e8b?ixlib=rb-4.0.3&q=80&fm=jpg&crop=entropy&cs=tinysrgb&w=3600)
<center>Photo by  <a href="https://unsplash.com/@jasondeblooisphotography?utm_source=medium&utm_medium=referral">Jason D</a> on <a href="https://unsplash.com/?utm_source=medium&utm_medium=referral">Unsplash</a></center>

## 甚麼是閃電貸？

閃電貸是一種無需抵押品便可以獲得貸款的方式，而且貸款沒有限制，這是利用以太坊上的交易機制來達到的。**在同一筆交易中借款還款**，交易完成後，以區塊的角度來看，**你的餘額完全沒有改變** ，但其實你已經借了幾百萬美金又還款了，這就是閃電貸的基本原理。

但你可能會問，如果我沒有還款不就可以拿走這些錢了嗎？以太坊的交易機制保證一筆交易要馬全部執行，要馬完全不執行。如果借款人在交易結束前沒有還款，該交易就會被 revert，回到借款發生前的 state，也就是說**借款從來沒有發生過**。因此，不會發生未還款就完成交易的情況。使用者只會損失這筆交易執行到一半所耗費的 gas fee 而已。

## 常見的閃電貸使用方式

因為閃電貸無須抵押品，並且可以立即得到一大筆資金，因此閃電貸的用途涵蓋了非常廣的範圍，以下將介紹幾個閃電貸常用的使用方式。需要注意的是這些使用方式通通都是在**一個 transaction** 中完成的。

> *P.S. 以下範例並沒有將 Defi protocol 的手續費以及 gas fee 考慮進去，在實務上這些都是很重要的一點，例如 Aave 對於使用閃電貸的人會收取一定比例的費用，根據[官方文檔](https://docs.aave.com/developers/v/2.0/guides/flash-loans#flash-loan-fee)，目前是 0.09% ，但是會隨著社區治理而變動。*

<center>
{{< figure src="https://images.ctfassets.net/ohcexbfyr6py/7bDWvjxaJfqacM5ZzWWgE3/5e038d260b9e6e1b44b481a332a817c6/BLOG_-_Flash_loan_transaction.jpg" attr="https://www.moonpay.com/learn/defi/defi-flash-loans" attrlink="https://www.moonpay.com/learn/defi/defi-flash-loans" >}}
</center>

### 套利 Arbitrage

利用閃電貸套利的方式非常多種，以下將解釋最常見的套利方式，利用不同交易所的**價格差**來進行套利。

假設：

- ETH 在 Uniswap 的價格：2000
- ETH 在 Sushiswap 的價格：2001

套利步驟為：

1. 在去中心化借貸平台如 Aave 使用閃電貸借入一筆 USDC。
2. 在 Uniswap 以 2000 的價格買入 ETH。
3. 在 Sushiswap 以 2001 的價格賣掉 ETH。
4. 向 Aave 償還 USDC 貸款。
5. 賺取差價利潤。

<center>
{{< figure src="https://images.ctfassets.net/ohcexbfyr6py/6oiB09a0dTEZihBWTAN7yi/2b97ed6696ec89e8becfb79ddcea4229/BLOG_-_Arbitrage.jpg?w=1920&q=100" attr="https://www.moonpay.com/learn/defi/defi-flash-loans" attrlink="https://www.moonpay.com/learn/defi/defi-flash-loans" >}}
</center>

### 替換抵押品 Collateral Swapping

假設你在 Aave 抵押了 1 ETH 借到了 1000 USDC，而你覺得 ETH 當抵押品可能相對沒有那麼穩定，你想要換成等值的 BTC 當作抵押品，正常的情況下你必須要先償還 1000 USDC 才能夠做到，但是有了閃電貸的幫助，能夠讓你在不還債的情況下做到這件事，步驟如下：

1. 在去中心化借貸平台如 Aave 使用閃電貸借入 1000 USDC。
2. 利用這 1000 USDC 去還債並且將你的抵押品（1 ETH）拿回來。
3. 將 1 ETH 換成等值的 BTC。
4. 將等值的 BTC 拿去抵押並借入 1000 USDC。
5. 利用借入的 1000 USDC 償還閃電貸的貸款。

### 清算自己的債務 Self-liquidation

假設現在 ETH 價格為 2000 USDC，而你在 Aave 抵押了 1 ETH 借到了 1000 USDC，而 ETH 的價格一直下跌到 1500 USDC，即將跌到清算價格，這時候你想要將 ETH 拿回來賣掉止損，正常情況下你需要 1000 USDC 才能夠拿回你抵押的 ETH。但如果使用閃電貸，你可以在沒有錢償還債務的情況下拿回你抵押的 1 ETH，步驟如下：

1. 在去中心化借貸平台如 Aave 使用閃電貸借入 1000 USDC。
2. 償還 1000 USDC 的債務並拿回 1 ETH 的抵押品。
3. 將 1 ETH 換成 1500 USDC。
4. 使用 1000 USDC 償還閃電貸貸款。
5. 持有剩下的 500 USDC。

## 閃電貸與預言機攻擊

另外一個閃電貸的用途就是駭客攻擊，攻擊手法有非常多種，最常見的一種是**價格預言機攻擊**，駭客可以利用閃電貸瞬間拿到一大筆資金，並且針對使用單一 DEX 餵價的 Defi protocol 進行攻擊。詳細步驟如下：

1. 攻擊者利用閃電貸借到一筆 token A。
2. 攻擊者利用 token A 去 Uniswap 交換 token B，導致 tokne A 的價格在 Uniswap 急遽下跌，而 token B 的價格急遽上漲。
3. 假設有一個借貸平台，他的 token 價格來源只有 Uniswap，這個時候攻擊者向此借貸平台抵押價格上漲的 token B ，然後借出價格下跌的 token A，攻擊者可以借出比正常數量還要多的 token A。
4. 攻擊者將一部份借出的 token A 償還給閃電貸並且持有剩下的 token A，這些就是攻擊者的營利。
5. 此筆交易結束後，Uniswap 上的 token A 和 token B 的價格會因為套利行為而回到正常的市場價格，然而這個借貸平台卻借出了不符合抵押品價格的 token A。

上述攻擊步驟都發生在一筆交易中，然而攻擊者可以重複發出這筆交易，直到合約裡所有的錢都被駭走為止。

<center>
{{< figure src="https://i.imgur.com/fAPieIY.png" attr="https://chain.link/education-hub/flash-loans" attrlink="https://chain.link/education-hub/flash-loans" >}}
</center>

### 那麼，該要如何避免這類攻擊手法呢？

既然單一來源的餵價會造成問題，那就讓它變成多個來源，也就是說不止參考 Uniswap 的價格，也參考 Sushiswap 的價格，甚至參考一些中心化交易所的價格。目前最多人用的是使用 Chainlink 提供的價格預言機，它的原理就是利用多個價格的中位數，這樣一來如果有其中一個交易所的價格偏離太多，也不會發生任何問題，如果對於 Chainlink 的原理有興趣可以閱讀他們的 [Doc](https://blog.chain.link/chainlink-price-feeds-secure-defi/?_ga=2.112211076.1690183760.1677998517-1988017328.1674545803)。

## Conclusion

閃電貸是把雙面刃，既可以用於套利，也可以用於駭客攻擊，而且完全沒有風險，因為如果交易失敗頂多就是損失 gas fee（如果進階一點甚至連交易失敗的 gas fee 都可以用 [Flashbots](https://docs.flashbots.net/flashbots-protect/rpc/quick-start#how-to-use-flashbots-protect-rpc-in-metamask) 節省掉）。

在 Defi 領域中他是聲名狼藉的存在，大多數人聽到閃電貸都會覺得又有攻擊事件發生了，但我認為閃電貸只是提供了一種新的資本利用方法，真正的問題應該是有漏洞的合約。長期來看，閃電貸甚至是有益於整個生態產業的，因為未來的 Defi protocol 工程師都必須要考慮以閃電貸獲得的大量資金會不會是潛在的安全漏洞，進而讓整個 Defi 產業變的更加安全穩固。

## References

- [DeFi flash loans explained](https://www.moonpay.com/learn/defi/defi-flash-loans#what-are-flash-loans)
- [Use-Cases of Flash Loans](https://hypertrader.app/use-cases-of-flash-loans/)
- [All You Need to Know About DeFi Flash Loans](https://medium.com/coinmonks/all-you-need-to-know-about-defi-flash-loans-ca0ff4592d90#b5db)
- [Flash Loans and Price Oracle Attacks](https://chain.link/education-hub/flash-loans#:~:text=Flash%20Loans%20and%20Price%20Oracle%20Attacks)
- [What is a Flash Loan Attack? And How Do I Prevent it?](https://shardeum.org/blog/what-is-a-flash-loan-attack/#How_Do_I_Prevent_a_Flash_Loan_Attack)