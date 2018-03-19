# Understanding Oraclize, an oracle service - Part 01
This repo is to understand and use Oracles in depth

I am going to divide this topic in couple of blogs. So that reader will get complete exposure of oracle services providers, like Oraclize in particular. In my first blog post I will try to give a brief overview of Oracle services. Basically dealing with questions like, What are Oracles? What is Oraclize? What is the use of such services? Etc. In the subsequent blog, we will try to use the Oraclize service with our smart contract and further we will take a deeper look into, how it works under the hood?

<!--more-->

### Overview:

As we know the problem with EVM -  Ethereum Virtual Machine is, it is difficult to perform complex problems on EVM due to its limitation. One of the limitation of EVM is, having its own set of opcodes, there is no direct access to advanced libraries which many complex applications would need to use. For example in financial system, the application may needs to use financial libraries, data and algorithms to perform some complex operations. Due to inability of smart contracts to talk to outside world (fetch data off chain), this leads to a very serious problem. And even if we try to perform, we end up spending a lot of ‘Gas’. The more complexity our smart contract has to handle, the more ‘Gas’ we will end up spending. 


So, it might be useful to be able to perform certain type of computation off chain and then simply relay the result back onto the blockchain. Infact smart contracts may needs reliable data from outside, based on that it can perform certain task. For example, an insurance contract, that says the flight company will refund 30% of fare, if flight is delayed by 30 mins. And to refund 30% of fare to the customer, the contracts needs to know from reliable source that whether the flight is delayed or not. 

This is where the use of Oracles are. Oraclize is one of the company that provides oracles services.

#### How does it works ? :

If our contracts needs data from outside world -

1. First of all our contract needs to inherit *usingOraclize* contract (assuming that we have imported oraclizeAPI.sol in our contract). E.g.  `contract ExampleContract is usingOraclize`.
2. Now our ExampleContract will trigger the oracle service with certain parameters like, data source type (e.g url, ipfs, wolframalpha etc), query (an array of parameters which need to evaluated in order to complete a specific data source type request) and optionally authenticity proof. E.g.  `oraclize_query("URL", "json(https://api.fixer.io/latest?symbols=USD,INR).rates.INR")`.
3. The oracle service will be responsible getting data form outside world in a **reliable way** (I will come to this later). 
4. Once oracle service has the reliable data from outside world, it will call a callback function in the smart contract to send this data. The callback function looks something like this : `function __callback(bytes32 myid, string result)`.

**Note** : 

-  Now this will happen in sometime in future. So it's very possible, the contract callback call that oracle service makes in our smart contract to send data back, is mined in different block than the initial contract call that smart contract performed to trigger the oracle service.
-  One of the oracle service providers is **Oraclize**

As per [Oraclize documentation](http://docs.oraclize.it/#home) - 

> "Oraclize is the leading oracle service for smart contracts and blockchain applications, serving thousands of requests for day every day on Ethereum, Bitcoin and Rootstock."

**reliable way** :

To understand this topic, lets focus on this question. *What is the guarantee that the data returned by these Oracles are correct?*

Oraclize answers this question by implementing concept called, **Authenticity Proof** - The solution developed by Oraclize is instead to demonstrate that the data fetched from the original data-source is genuine and untampered. This is accomplished by accompanying the returned data together with a document called authenticity proof. 

Smart contracts can request authenticity proofs together with their data by calling the `oraclize_setProof` function available in the *usingOraclize* - Oraclize smart contract that our smart contract will inherit. 

When a smart contract requests for an authenticity proof, it must define a different callback function with the following arguments: `function __callback(bytes32 queryId, string result, bytes proof)`.

The oraclize_setProof function expects the following format: `oraclize_setProof(proofType_ | proofStorage_ )`
Both *proofType* and *proofStorage* are of type byte constant defined in *usingOraclize*. 

Available proofType are : 
- proofType_NONE : authenticity proof can be disabled by calling oraclize_setProof(proofType_NONE).
- proofType_TLSNotary
- proofType_Android
- proofType_Native
- proofType_Ledger

And proofStorage is :
- proofStorage_IPFS

For Example, `oraclize_setProof(proofType_TLSNotary)` will return the full *TLSNotary* proof bytes as the proof argument in callback method. If instead `oraclize_setProof(proofType_TLSNotary | proofStorage_IPFS)` is used, then Oraclize will return only the *base58-decoded* IPFS multihash as the proof argument. To obtain the IPFS multihash, the bytes must be encoded to base58.  We can learn more about TLSNotary from [here](https://tlsnotary.org/TLSNotary.pdf).

**Note:**
- Authenticity concept is made optional since calling it may consume a decent amount of gas. 
- The authenticity proofs may be relatively large files, of up to a few kilobytes. Delivering such proofs directly within the result of the data payload in an Ethereum transaction can get quite expensive, in terms of EVM execution costs, and may even be impossible for larger data.Therefore the proof is uploaded and saved to IPFS, a decentralized and distributed storage system.

That’s all for this blog. In the next blog we will have two objectives :
1. We will try to see how to use oraclize with our smart contract using one example.
2. Once we get the idea of using oraclize, we will go further to understand how it works under the hood. I mean what exactly happens inside `oraclize_query` method. We will take a deeper look in `usingOraclize` contract and will try to understand this.

References: 
- [Oraclize Doc](http://docs.oraclize.it/#background)
- [TLSNotary WhitePaper](https://tlsnotary.org/TLSNotary.pdf)
- [Very helpful youtube video](https://www.youtube.com/watch?v=04WiDy_Of2A)
- [How oraclize handles TLSNotary](https://ethereum.stackexchange.com/questions/201/how-does-oraclize-handle-the-tlsnotary-secret)
 


