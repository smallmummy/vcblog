---
layout:     post
title:      Arbitrage Model with Cryptocurrency and Currency Exchange
subtitle:   the model to seek the arbitrage chances between cryptocurrency and triditional currency
author:     vincent_c
image: assets/images/post-bg-hacker.jpg
categories: [ cryptocurrency, python ]
catalog: true
tags: [Crawler, featured, stick_post]
---  
This project was the one I had implemented in 2018. Actually, after 2007, I've entered the cryptocurrency market, I insist on this area using my spare time.  

This program was developed based on Python(Platform)+Zabbix(display)+Wechat(Notification), which can monitor the arbitrage chances caused by the gap between cryptocurrency exchanges and bank.

## 1.Brief of the mechanism

To be honest, this mechanism is the same as the traditional trade. The mechanism in traditional trading is buying from the east and selling to the west, and nowadays, this underlying mechanism is not changed.

There is a simple arbitrage term named "moving bricks", which means buying some cryptocurrencies using legal tender in one exchanges, then sell those cryptocurrencies in anther exchanges to get legal tender back, like moving the brick from a place to another. Due to the gap of quotation between different exchanges, you can gain some profit using this way.

In this article, there is another factor added: different kind of legal tender. Then the rate of traditional currency exchange is involved, as the price of the legal tender which is used to buy is different from that of one which is used to sell. Such organization like the bank or money changer, or even individual, like the underground bank was in this process to finish the last step: exchange the money back to the former legal tender.

you might still be confused, no worries, the illustration in the following chapter will give you the detail.

## 2. The whole process

Notes:  

* To simplify the explanation of the whole process, I specify the CNY (Chinese Yuan - Chinese legal tender) as the legal tender at the initial step,  and specify the IDR (Indonesia Rupiah - Indonesia legal tender) as the legal tender at the end step, and BTC (bitcoin) as the cryptocurrency.  
* In the last step --- "exchange the money back to former legal tender", I explain the process using the bank. Actually, it can be a money changer or even an underground bank.
* There is two way of circles in the arbitrage, one called positive arbitrage, another is inverse arbitrage. However, it wasn't defined with the direction from one specific currency to another; it is just a na

the process of positive arbitrage:  
![](https://cl.ly/659c7ba8d2d7/1st-process%252520of%252520arbitrage.png)
<center><div style="border-bottom: 1px solid #d9d9d9;display: inline-block;color: #999;">
Figure: process of positive arbitrage</div></center>

There are seven steps as we can see from the above figure:

1. to deposit some CNY in the exchanges in China. No fee in this step.
1. buy some BTC using CNY. There is a trading fee in this step, and the ratio depends on the exchanges.
1. withdraw the BTC from exchanges in China, and transfer it to the exchanges in Indonesia. There is some withdrawal fee required by the exchanges in China, and settlement with BTC.
1. to sell the BTC in exchanges of Indonesia, then get IDR in the account of exchanges. There is a trading fee in this step.
1. withdraw the IDR from exchanges in Indonesia, and the money of IDR will be deposited to the account of Indonesia local bank. There is some withdrawal fee in this step.
1. transfer the money to the bank using IDR.
1. get CNY back to your account in a Chinese bank.

The process of inverse arbitrage:
![](https://cl.ly/5289a520f54e/2nd-process%252520of%252520arbitrage.png)
<center><div style="border-bottom: 1px solid #d9d9d9;display: inline-block;color: #999;">
Figure: process of inverse arbitrage</div></center>


## 3. the detail formula  

### 3.1 the deduction of the formula:

1. The amount of CNY (Chinese currency) you use to invest in this arbitrage:

	$$ M_{CNY} $$

1. The asking price in exchanges of China, means you need to use CNY (Chinese currency) to buy BTC  

	$$ P\_ASK_{china} $$  

1. The exchange fee when you trade in China:

	$$ Fee_{china} $$

1. after trade in exchanges of China, the BTC you got is:

	$$ \frac{M_{CNY}}{P\_ASK_{china}} * (1-Fee_{china}) $$

1. then you need to transfer the BTC which you bought in China to exchange of Indonesia to sell it. In this step, you need to pay the fee of transferring, which is with the settlement of BTC.  

	we mark the fee of transferring is:
	
	$$ Fee_{trans} $$

1. after transfer in , the amount in your exchanges of Indonesia is:

	$$ \frac{M_{CNY}}{P\_ASK_{china}} * (1-Fee_{china}) - Fee_{trans}$$  
    
1. the price of bidding in exchanges of Indonesia:

	$$ P\_BID_{indo}$$    

1. the IDR (Indonesia currency) you got after sold:

	$$ (\frac{M_{CNY}}{P\_ASK_{china}} * (1-Fee_{china}) - Fee_{trans})*P\_BID_{indo} $$  

1. the fee of withdrawal in exchanges of Indonesia:

	$$ Fee_{withindo} $$

1. the IDR you got in your bank after you withdraw the IDR from exchanges of Indonesia, we marked this amount as:

	$$ M_{IDR} $$

	then it should be:
	
	$$ M_{IDR} = (\frac{M_{CNY}}{P\_ASK_{china}} * (1-Fee_{china}) - Fee_{trans})*P\_BID_{indo}*(1-Fee_{withindo}) $$  

1. until this step, you already accomplish the exchange from CNY to IDR via BTC trading.

    now we need to consider exchange IDR back to CNY to complete the whole arbitrage processes if there is any benefit space.  

	the ask price of Bank which can help us exchange IDR back to CNY. 
	
	$$ Bank_{ask} $$


1. the amount of CNY which was exchanged from IDR is:

	$$ \frac{M_{IDR}}{Bank_{ask}} $$

1. if the preceding amount is greater than our initial amount of CNY, that means there is profit:

	$$Profit = \frac{M_{IDR}}{Bank_{ask}} - M_{CNY} $$


### 3.2 final formula:

#### 3.2.1 Profit
let's simplify the above formula, and the clear one is like:

$$ Profit = M_{CNY}*[\frac{(1-Fee_{china})*P\_BID_{indo}*(1-Fee_{withindo})}{P\_ASK_{china}*Bank_{ask}}-1] - \frac{Fee_{trans}*P\_BID_{indo}*(1-Fee_{withindo})}{Bank_{ask}} $$

if the value of the above formula is greater than 0, means we can do it.

#### 3.2.2 The \\(Bank_{ask}\\)  

However, in the actual operation, the value of \\(Bank_{ask}\\) is realtime as well. Then the actual operation step is like: you should be aware of how much is the value of it, which can help you benefit, then you acquire the value from the bank or somewhere. At last, you decide if you perform the implementation of the arbitrage base on those value.

the value of \\(Bank_{ask}\\) should meet:

$$ Bank_{ask} < \frac{M_{IDR}}{M_{CNY}}$$

after simplify, it is:

$$ Bank_{ask} < [\frac{1-Fee_{china}}{P\_ASK_{china}}- \frac{Fee_{trans}}{M_{CNY}})*P\_BID_{indo}*(1-Fee_{withindo}) $$  

### 3.3 parameters in formula:

Let's analyze all the parameters above and classify them. It will be better for us to design our programme.

there are two kinds of parameters:  

1. fixed value which should be defined as initial:

	Name|Meaning
	----|----
	\\(M_{CNY}\\)|initial money by CNY
	\\(Fee_{china}\\)|the exchange fee when you trade in China
	\\(Fee_{withindo}\\)|the fee of withdrawal in exchanges of Indonesia
	\\(Fee_{trans}\\)|fee of transferring cryptocurrency
	\\(Bank_{ask}\\)|the CNY ask price of Bank

1. realtime value which should be fetched from exchanges or somewhere:

	Name|Meaning
	----|----
	\\(P\_BID_{indo}\\)|the price of bidding in exchanges of Indonesia
	\\(P\_ASK_{china}\\)|the asking price in exchanges of China

## 4. Design of programme 

### 4.1 Mode one: getting Bank rate in realtime

In this mode, if you can get the rate of the bank, namely the value of \\(Bank_{ask}\\) in the above formula, then you can design the programme like following flow:   

![](https://cl.ly/35ace4b4545a/Image%2525202019-06-23%252520at%25252012.42.02%252520%2525E4%2525B8%25258B%2525E5%25258D%252588.png)
<center><div style="border-bottom: 1px solid #d9d9d9;display: inline-block;color: #999;">
Figure: Program Flow</div></center>

### 4.2 Mode two: calculate the threshold of \\(Bank_{ask}\\)

In the actual operation, I couldn't fetch the value of the rate of the bank in realtime due to several reasons, such as the opening hour of bank, and there is no realtime interface from the underground bank and etc. So I prefer this mode: calculating the threshold base on all current information, then save it, and remind me if the value crosses a certain threshold.  Then the flow becomes:

![](https://cl.ly/f6c8805ad6a9/Image%2525202019-06-25%252520at%25252010.04.01%252520%2525E4%2525B8%25258A%2525E5%25258D%252588.png)
<center><div style="border-bottom: 1px solid #d9d9d9;display: inline-block;color: #999;">
Figure: Program Flow</div></center>

## 5. Monitoring Graphs

Since I chose the mode two, then I developed the programme and saved the value into DB, and displayed in ZABBIX like this:

![](https://cl.ly/2acd949cc66e/Image%2525202019-06-25%252520at%25252010.53.49%252520%2525E4%2525B8%25258A%2525E5%25258D%252588.png)
<center><div style="border-bottom: 1px solid #d9d9d9;display: inline-block;color: #999;">
Figure: Zabbix GUI</div></center>