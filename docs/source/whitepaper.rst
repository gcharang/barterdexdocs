Whitepaper
==========

BarterDEX - a truly decentralized exchange

Abstract
--------

In this whitepaper we describe the implementation of a decentralized exchange that would allow people to trade coins without a counterparty risk. A fully featured service would need to decentralize order matching, trade clearing, and settlement. The order matching would be done through low level pubkey to pubkey messaging protocol and the final settlement through an atomic cross chain protocol. Like any exchange, a decentralized alternative also needs liquidity providers to ensure it has enough liquidity.

Introduction
------------

Money should be exchanged freely and safely from person to person, and currently the most practical method for it is a central exchange. Such centralized solution requires the funds to be exchanged into IOU tokens that are backed by the exchange service provider. In doing so, the clients are under a constant risk of their assets being stolen either by an inside theft or an outside hack. In order to remove these risks, a decentralized alternative must be established.

Among all the exchanges, the trading tends to be centralized in a few exchanges – and there is a reason for it. The speed advantage of trading the exchange IOU attracts the traders, and the traders attract liquidity, which creates the best prices, which attracts even more traders. This network effect is the reason that a few centralized exchanges have been dominating the trading volumes, while all smaller exchanges, both centralized and decentralized, are suffering from lack of liquidity.

A decentralized trading environment was created in 2014 by a service called MultiGateway, that utilized the Nxt Asset Exchange. With the Nxt Asset Exchange it is possible to do decentralized trading, using proxy tokens to represent external cryptocurrencies like Bitcoin. This hybrid solution is being increasingly used by many other blockchain platforms.

One problem with this type of solution, however, is that it loses the speed of a centralized exchange. And another problem is that the storage of external cryptocurrencies is not decentralized, but only distributed at best, which maintains a degree of counterparty risk for the users. This, combined with the need to use a set of gateways to convert the external, native coins to and from the proxy tokens has made it an impractical solution.

The conclusion is that a decentralized alternative that lacks speed cannot compete with the convenience of existing centralized exchanges, and that simply reducing the counterparty risks it is not enough to attract liquidity.

The complete solution
---------------------

BarterDEX combines three key components: order matching, trade clearing and liquidity provision into a single integrated system that allows users to make a coin conversion request, find a suitable match and complete the trade using an atomic cross-chain protocol. Additionally, there is a privacy layer in the order matching so that two nodes can do a peer-to-peer atomic swap without any direct IP contact between them.

Order matching is the process of pairing buy orders with sell orders. It is done by algorithms that define how the orders are paired, and in which order they get filled. A trade is said to become ‘executed’ once it gets filled.

After execution comes trade clearing, which is the process from the promise of payment to the actual money transfer or settlement, where the assets are swapped between the trading parties.

A liquidity provider (LP) is a trading party that acts as a market maker buying and selling assets. They provide liquidity to the exchange and make their profit from the spread between bid and ask orders. LP's bring price stability and makes it easier for traders to execute trades.

Order Matching
--------------

Before we get into the atomic swap details, there is another aspect of barterDEX that is quite critical: the decentralized orderbook. In order to achieve this, a custom peer to peer network is created that has the analogue of a full relay node and a node that doesn't relay. Network load on this network is minimized using a combination of hierarchical transmission of the orderbooks, along with fetching of data. Also, there are several different methods to obtain data to maximize the number of nodes that are able to fully connect to the barterDEX network.

One aside to mention is that it is possible to create a totally separate set of BarterDEX nodes by seeding the network with a totally independent set of seed nodes. This enables scaling to arbitrary levels as, if any scaling limit is reached, it is a matter to create a separate network that doesn't directly interconnect with the other. At such scale, the assumption is that there is plenty of inventory in the orderbooks, especially if the partitioning is done on a per-reference coin basis. In the event it is desired to cross-pollinate orderbook entries from disparate barterDEX networks, it would be possible to have special bridge nodes that cherry pick the desired entries from one network to the other.

Anybody that wants to can run a full relaying node, there is no specific payment to do this and it does require more bandwidth. However, by being a full relaying node, you have better connectivity with all the other nodes and thus a higher percentage chance of having a trade started and completed. This increase in reliability would be enough for active traders and also for significant owners of the DEX asset, making sure there are enough full nodes is a good thing to do.

A non-relaying node is able to do everything a full relaying node can do, so we expect that the vast majority of nodes will be non-relaying nodes and this will enable the barterDEX network to scale to a large number of total nodes. With 100 full nodes, thousands of non-relay nodes can be supported, possibly tens of thousands, though that number has not been reached in practice, so we will have to wait and see what the real world limitations are.

Atomic Swaps
------------

BarterDEX implements the Tier Nolan protocol as described in the `bitcointalk thread`_.

Overview of the atomic swap protocol
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. _bitcointalk thread: https://bitcointalk.org/index.php?topic=1364951

While the thread is quite technical, it gives a very good background into the tradeoffs that went into selecting the atomic swap protocol. The important thing to note is that at each step of the protocol there are incentives/disincentives to proceed to the next step and that regardless of where the protocol stops, each party ends up with what they should get. The understanding is that if you don’t follow the protocol, you will end up paying some amount of penalty.

In order to achieve this, the liquidity provider, who we call bob, needs to have a deposit to ensure his completion of the protocol. This means that bob needs two UTXOs to do an atomic swap. alice also needs two UTXOs, but her additional UTXO is the dexfee that is required to prevent spamming the orderbook. Without this, alice could initiate an unlimited number of atomic swaps and the bobs would all be simply stuck waiting for the time period to expire to get a refund of their deposit. With the dexfee, there is a financial cost to alice for such bad behavior since there is no financial gain for alice to be annoying, we expect that there won't be much intentional spamming.

Ignoring all the validation details of each step, the atomic swap consists of up to 7 transactions (in some cases, it would be less). The following shows the mainstream SWAP complete! the sequence for both sides:

    1. alice sends dexfee
    2. bob sends bobdeposit
    3. alice sends alicepayment
    4. bob sends bobpayment
    5. alice spends the bobpayment
    6. bob spends the alicepayment
    7. bob refunds his own deposit

While it seems a bit inefficient to have 7 transactions for a swap that could be done with just 2 transactions, this is what is required to make it trustless and have the characteristic that at any step, there are incentives to go to the next step and where both sides end up with the right amounts regardless of where things stop.

Let us see what happens if things just stop at a certain step:

    1. alice sends dexfee. If bob does not send the bobdeposit, alice is out a dexfee, which is 1/777 of the transaction amount. This will then give bob a bad reputation and very quickly nobody will trade with bob. As long as the frequency of bob failing to deposit is low, the occasional extra dexfee is a minor issue. Contingency plans are in place to provide refunds if a particular alice node experiences a materially large amount of lost dexfees.

    2. bob sends bobdeposit. If alice doesn't send the alicepayment, then alice loses not only the dexfee but gains a bad reputation and soon nobody would trade with alice. We don't expect this to happen that often.

    3. alice sends alicepayment. If bob doesn't send in payment, after 4 hours, alice can claim the bobdeposit, which is 12.5% larger than the payment, so alice ends up with a nice bonus in this case. I would not be surprised if the alice nodes are eager for this case of atomic swap protocol.

    4. bob sends bobpayment. If alice doesn't spend the bobpayment, then after 2 hours he can reclaim his payment and then after 4 hours refund his deposit. Once bob refunds his own deposit, then alice is able to reclaim her payment. It is all intricately interconnected as the spending of a specific transaction enables the other party to spend their counterpart.

    5. alice spends the bobpayment. If bob doesn't spend the alicepayment, alice is already done with the trading, so there is no more she needs. Bob is sleeping and doesn't spend the alicepayment, then he is out the alicepayment until he spends it. this is up to bob, but it is a bit dangerous as if he does the refund of the deposit before spending alicepayment, alice would get the info needed to reclaim her payment. It is important for both participants to continue running the atomic swap protocol until it completes. If after 4 hours, bob is still sleeping, then alice is able to claim the deposit and become a happy camper. alice spending the bob deposit, however, does not give her the information needed to reclaim her own payment, so bob is still able to do this when he wakes up.

    6. bob spends the alicepayment. Similar to above, if bob doesn't refund his own deposit, it is his loss and purely his responsibility. If after 4 hours, he still hasnt, then alice will be able to claim the deposit.

    7. bob refunds his own deposit. As you can see at each step, the side that needs to do something is motivated to do so, with greater and greater urgency toward the end.

BarterDEX implements the above in a cross-platform way, across almost a hundred coins, using either native coin daemon or SPV electrum servers. A swap that is not completed during one session can be completed as long as BarterDEX is run before the time expires. It is best to not trade a very large amount unless you are sure of your node's reliability, especially regarding the internet connection.

Believe it or not, doing the above atomic swap protocol with all the cryptographic validations in between along with a fancy key exchange, is less than half the difficulty of barterDEX. Relatively speaking, it is 'easy' to do an atomic swap in isolation between two test nodes with carefully prepared UTXOs made especially for the test. It is an entirely different matter to be able to let anybody start trading with anybody else and have things like orderbooks and ordermatching happen. Due to the peer to peer nature, it is impossible to guarantee a successful swap, however, a failed swap to start is just a few seconds of lost time and there is no cost to try to start a swap. Just like in normal trading, there is no guarantee that you can get the trade you want, similarly, there is no guarantee with barterDEX.

Details of the atomic swap protocol
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Let us see what is required in a bit more detail as we now have the context of the atomic swap protocol. In order to even start an atomic swap, there needs to be a pair from alice such that the dexfee and alicepayment can be created and also two from bob, such that the bobdeposit and bobpayment can be created. BarterDEX requires all four of these UTXOs to be specified before the start of the atomic swap protocol.

Here comes the first user issue. Most users don't know what an UTXO is and most view their balance as a single blob of coins they can spend at the satoshi level. The reality is that the bitcoin protocol maintains a list of Unspent Transaction Outputs (UTXO) of specific values and to make a transaction, there needs to be inputs sufficient to pay for the outputs. Any excess goes into a change output (let us ignore txfees for now, even though BarterDEX automatically calculates the current BTC txfee to pay that will get confirmed quickly).

It is impractical to have the user specify which UTXO pair to use, and it is not possible for alice to even know what UTXOs bob has available at the moment of negotiating a trade. What barterDEX does is an atomic swap negotiation protocol as follows:



    1. alice sends a "request" to a specific bob with her pair of UTXOs, price and volume

    2. bob validates the "request" to make sure the alice UTXOs are valid and that the price is acceptable, then bob scans all his UTXOs for the most efficient way to create both the bobpayment and bobdeposit UTXOs. The constraint is that it needs to match the price alice wants to pay and the volume and the deposit at least 12.5% bigger than the payment. If all these things are met, then bob sends back a "reserved" packet to alice. By doing this just in time, bob minimizes the funds that are tied up doing deposit duty.

    3. alice validates the "reserved" packet from bob, making sure all the UTXOs are valid, the price and volumes are acceptable and if so sends a "connect" packet to bob with the same parameters as in the "reserved". Between the "request" being sent and the "reserved" being received or a 10-seconds timeout, alice is prevented from making any other trade request. It is important to make sure the current pending atomic swap is properly started and this prevention is also part of the whale resistance property for dICO (decentralized initial coin offering).

    4. bob validates the "connect" and if all is well, starts a new thread to do the atomic swap.

    5. alice receives the "connect" and if all is well, starts a new thread to do the atomic swap.

There is one more "negotiation" step that is needed between alice and bob and while it could have been part of the 5 steps above, due to legacy reasons it ended up inside the atomic swap protocol itself. In the event there is no consensus on the coin confirmations to use, the atomic swap aborts without any payments being sent. No harm, no foul.

DEX fee - dexfee
~~~~~~~~~~~~~~~~

People will notice that there is a small dexfee, paid by alice, as part of the barterDEX protocol. This is 1/777 of the transaction amount and it is calibrated to make spam attacks impractical. Impractical as in costing real money. Without this spam prevention, BarterDEX could be cost-effectively DOS attacked at the protocol level.

The 1/777 ends up being 0.1287%. This is less than almost all the centralized exchanges, in many cases by a significant margin. Please note that the central exchanges charge both sides of the trade, so even if they charge 0.2%, it is actually 0.4% total fees.

The dexfee helps secure the barterDEX network and it is set at a level that is less than the central exchanges. It is possible that some trades can start without completing and since the dexfee is charged first in the protocol. In this sense, there would be a dexfee charged for these failed atomic swaps. While the decentralized exchanges using proxies charge for every bid and every ask, even if a trade doesn't happen at all. This is a big negative for trading on proxy-powered DEX's.

However, it should not be looked upon in isolation. The barterDEX protocol is based on statistics and statistically, there will be some percentage of atomic swaps that are started that won't complete. Let us say this is a 15% failure rate (this is much higher than we are seeing in testing, where 95%+ of atomic swaps that are started complete, or reach the bobdeposit phase at the minimum), then the effective dexfee cost is still around 0.15%

This is why even though the 1/777 ratio is 0.1287%, it is better to state the dexfee as 0.15%

Fees are collected and distributed to DEX asset holders, which is an assetchain. Any DEX assetchain unit holder will receive this DEX fee as a dividend, paid in KMD.

Transaction Confirmations
~~~~~~~~~~~~~~~~~~~~~~~~~

Since barterDEX is trading real coins and not just updating an internal database (or a proxy tokens account balance for proxy DEX), both sides need to wait for coin confirmations to the level they are comfortable. Since the payments sent on one chain won't be reorganized if the other chain does, it is important to have enough confirmations for the size of the trade being done. In order to enable this, there is a setconfirms API call that can be called for each coin. This needs to be done before the atomic swap is started as the current numconfirms for the coin will be sent to the other side and the larger of alice or bob's numconfirms will be used. There is also a maxconfirms value to prevent one side from specifying an abnormal high amount of numconfirms.

Zero Transaction Confirmations
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

To be able to offer the user a trading experience similar to existing centralized exchanges, zero confirmation mode can be enabled. It is quite risky to do this, especially for fast block confirmation time coins with low hashrate. For zero confirmation swaps to be enabled, both sides of a trade need to have so-called zcredits, a locked amount of KMD that is more than the value of that trade. 

Efficiency of orderbook
-----------------------

We have now worked backward from the atomic swap details to the ordermatch process and this leaves the efficient orderbook propagation as the only part left to describe. barterDEX uses a convention of base/rel meaning base currency converted to rel currency. Buying a base/rel means to use rel currency to buy base currency, price denominated in base/rel: buy 1 base for x rel, x being the price.

In order to construct an orderbook a node needs to have price information and since everything is pubkey based, this means a price from a pubkey. Ultimately a specific txid/vout (UTXO) is needed, but a single node could have hundreds of UTXOs and this would use up a lot of network bandwidth to propagate it globally. barterDEX, therefore, uses a hierarchical orderbook, where the skeleton of it is just the pubkey/price for a particular base/rel pair. Note that a buy of base/rel at price is the same as a sell of rel/base at 1/price. So all that is needed to populate the orderbook skeleton is for a node to broadcast its pubkey and price for a base/rel pair. Given this, nodes that are running a local coin daemon can find the possible list of UTXOs via listunspent on demand.

Critical information is fully signed to prevent spoofing, so all nodes can verify the smartaddress associated with a pubkey and also that the price being broadcast is a valid price. The electrum SPV coins do a specific SPV validation for all UTXO before they are approved for trading.

If all nodes were always broadcasting all their UTXOs to all other nodes, it would rapidly lead to congestion. Most of the time barterDEX just relies on the pubkey/prices and this is enough to create useful orderbooks. Since there are N*N possible orderbooks given N currencies, it is not practical to be updating all possible orderbooks, instead, they are created when requested from the raw data. During the orderbook creation, if the top entries in the orderbook don't have any listunspent data, a request for it is made to the network.

This process ensures that by the time a trade is done, already an orderbook has been requested which in turn requests the listunspent data for the most likely pubkeys. The actual ordermatch process then iterates through the orderbook scanning all the locally known UTXOs to find a high probability counterparty to make the "request" offer to. In practice, we are seeing the nearly instantaneous response when all the parameters are properly met.

References
----------

barterDEX – A Practical Native DEX (https://github.com/KomodoPlatform/KomodoPlatform/wiki/BarterDEX-%E2%80%93-A-Practical-Native-DEX)

Nakamoto Satoshi (2008): Bitcoin: A peer-to-peer electronic cash system. (http://www.bitcoin.org/bitcoin.pdf)

Mtchl (2014): The math of Nxt forging (https://www.docdroid.net/ahms/forging0-4-1.pdf.html)

King Sunny, Nadal Scott (2012): PPCoin: Peer-to-Peer Crypto-Currency with Proof-of-Stake (https://peercoin.net/assets/paper/peercoin-paper.pdf)

Delegated Proof-of-Stake Consensus (https://bitshares.org/technology/delegated-proof-of-stake-consensus/)

Miers Ian, Garman Christina, Green Matthew, Rubin Aviel: Zerocoin: Anonymous Distributed E-Cash from Bitcoin (https://isi.jhu.edu/~mgreen/ZerocoinOakland.pdf)

Ben-Sasson Eli, Chiesa Alessandro, Garman Christina, Green Matthew, Miers Ian, Troer Eran, Virza Madars (2014): Zerocash: Decentralized Anonymous Payments from Bitcoin (http://zerocash-project.org/media/pdf/zerocash-extended-20140518.pdf)

Ben-Sasson Eli, Chiesa Alessandro, Green Matthew, Tromer Eran, Virza Madars (2015): Secure Sampling of Public Parameters for Succinct Zero Knowledge Proofs (https://www.diyhpl.us/~bryan/papers2/bitcoin/snarks/Secure%20sampling%20of%20public%20parameters%20for%20succinct%20zero%20knowledge%20proofs.pdf)

NXT Community: NXT Whitepaper (https://nxtwiki.org/wiki/Whitepaper:Nxt)

Larimer Daniel, Scott Ned, Zavgorodnev Valentine, Johnson Benjamin, Calfee James, Vandeberg

Michael (March 2016): Steem, An incentivized, blockchain-based social media platform.(https://steem.io/SteemWhitePaper.pdf)

BitFury Group (Sep 13, 2015): Proof of Stake versus Proof of Work White Paper (http://bitfury.com/content/5-white-papers-research/pos-vs-pow-1.0.2.pdf)




