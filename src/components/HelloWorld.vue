<script setup>
import { ref } from 'vue';
import Web3 from "web3";
const { ethereum } = window;
const web3 = new Web3(ethereum);

const account = ref(null);
const nativeBalance = ref(null);
const signedMessage = ref(null);
const chainId = ref(null);


const connectWallet = async () => {
  try {
    if (account.value) {
      account.value = null;
      return await ethereum.request({
        method: "wallet_revokePermissions",
        params: [{ "eth_accounts": {} }]
      });
    }
    if (!ethereum) {
      return console.log('Install Metamask first');
    }
    const connect = await ethereum.request({ method: "eth_requestAccounts" });
    if (connect.length) {
      account.value = connect[0];
      await checkChainId();
      await checkNativeBalance(account.value);
    }
  } catch (error) {
    console.log(error.message)
  }

}

const checkNativeBalance = async (address) => {
  try {
    const balance = await web3.eth.getBalance(address, "latest");
    nativeBalance.value = web3.utils.fromWei(balance, 'ether');
  } catch (error) {
    console.log(error.message)
  }
}

const checkChainId = async () => {
  try {
    chainId.value = await web3.eth.net.getId();
  } catch (error) {
    console.log(error.message)
  }
}

const signMessage = async () => {
  try {
    const message = 'Sign this test message';
    signedMessage.value = await ethereum.request({
      method: "personal_sign",
      params: [message, account.value]
    });
  } catch (error) {
    console.log(error.message)
  }
}

const sendTransaction = async () => {
  try {
    const amount = web3.utils.toWei('0.001', 'ether');
    const toAddress = '0x1A2Bb5Ba4A8D53B3B1f6C59b483ED4a10dAfc3ad'
    const nonce = await web3.eth.getTransactionCount(account.value);
    const gasPrice = await web3.eth.getGasPrice();
    const gasLimit = await web3.eth.estimateGas({
      from: account.value,
      to: toAddress,
      nonce: web3.utils.toHex(nonce),
    });
    const transactionPromise = web3.eth.sendTransaction({
      to: toAddress,
      value: amount,
      from: account.value,
      nonce: web3.utils.toHex(nonce),
      gasPrice: gasPrice,
      gas: gasLimit
    });
    const transactionReceipt = await transactionPromise;

    console.log(transactionReceipt.transactionHash)
  } catch (error) {
    console.log(error.message)
  }
}
</script>

<template>
  <div class="greetings">
    <div v-if="account"> Active ChainID: {{ chainId }} </div>
    <div v-if="account"> Address: {{ account }} </div>
    <div v-if="nativeBalance && account"> Native Balance: {{ nativeBalance }} </div>
    <div v-if="signedMessage && account"> Sign Message: {{ signedMessage }} </div>
    <div class="connectWallet">
      <button v-if="account" class="button" @click="signMessage()">Sign Message</button>
      <button v-if="account" class="button" @click="sendTransaction()">Send Transaction</button>
      <button class="button" @click="connectWallet()">{{ account ? 'Disconnect' : 'Connect Wallet' }}</button>
    </div>
  </div>
</template>

<style scoped>
.address {
  margin-bottom: 30px
}

.connectWallet {
  margin-top: 30px
}

.button {
  margin-right: 5px;
}
</style>
