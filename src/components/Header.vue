<script setup lang="ts">
import { ethers } from "ethers";
import { onMounted, reactive } from 'vue';
import { RetVal } from '../components/Types';

import { useRequest } from 'vue-request';

import BigNumber from "bignumber.js";

function reloadPage() {
  location.reload();
}

let state = reactive({ account: '', balance: '', epoch: 0 })

async function connectAction() {
  if (typeof window.ethereum === 'undefined') {
    alert("Matamask is not installed!")
    return
  }

  const accounts = await window.ethereum.request({ method: 'eth_requestAccounts' });
  const account = accounts[0];
  console.log(account)

  state.account = account

  let network = new ethers.Network('Filecoin - Calibration testnet', 314159)

  // Connect to the MetaMask EIP-1193 object. This is a standard
  // protocol that allows Ethers access to make all read-only
  // requests through MetaMask.
  let provider = new ethers.BrowserProvider(window.ethereum, network)
  window.provider = provider

  // It also provides an opportunity to request access to write
  // operations, which will be performed by the private key
  // that MetaMask manages for the user.
  let signer = await provider.getSigner();
  window.singer = signer;
}

const queryAction = async function () {
  let epoch = await window.provider.getBlockNumber();
  let bal = await window.provider.getBalance(state.account);

  return {
    epoch: epoch,
    balance: bal,
  } as RetVal
}

onMounted(() => {
  useRequest(queryAction, {
    pollingInterval: 3000,
    pollingWhenHidden: false,
    onSuccess: (val: any) => {
      let bal = new BigNumber(val.balance)
      let intt = new BigNumber('100000000000000')
      let hh = bal.div(intt).toPrecision(3).toString();
      state.balance = hh

      state.epoch = val.epoch;
    },
    onError: err => {
      console.log(err)
    }
  })
})

</script>

<template>
  <div class="container">
    <el-row justify="space-between">
      <el-col :xs="6" :sm="4" :md="4" :lg="2" :xl="2">
        <img class="logo-view" src="https://filecoin.io/images/filecoin-logo.svg"
          style="width: 48px;height: 24px;cursor: pointer;" alt="" @click="reloadPage">
      </el-col>

      <el-col :xs="15" :sm="7" :md="6" :lg="5" :xl="4">
        <el-row>
          <div class="doc-link-view"> {{ "Epoch: " + state.epoch }} </div>
          <div class="doc-link-view" style="margin-left: 10px;"> {{ "Balance: " + state.balance }} </div>

          <div class="doc-link-view" style="margin-left: 10px;font-size: 16px;" @click="connectAction"> {{ state.account ? state.account.substring(0, 4) + '...' +
            state.account.substring(state.account.length - 4, state.account.length) : "Connect" }}</div>
        </el-row>
      </el-col>
    </el-row>

    <div class="slogon-view">
      <img class="solgon-title-view" src="https://filecoin.io/images/filecoin-logo.svg" alt="">
      <div class="solgon-content-view">Deposit safely and Mine conveniently</div>
    </div>
  </div>
</template>

<style scoped>
.container {
  background-image: linear-gradient(180deg, #513eff 0%, #52e5ff 100%);
}

.logo-view {
  margin-top: 10px;
}

.doc-link-view {
  height: 30px;
  border-radius: 15px;
  border-color: #D8D8D8;
  font-size: 14px;
  font-weight: 600;
  color: #FFFFFF;
  line-height: 30px;
  cursor: pointer;
  margin-top: 10px;
}

.slogon-view {
  margin: 0 auto;
  width: 100%;
  text-align: center;
}

.solgon-title-view {
  margin-top: 60px;
  height: 60px;
}

.solgon-content-view {
  margin-top: 10px;
  padding-bottom: 60px;
  height: 25px;
  font-size: 24px;
  font-weight: 400;
  color: #FFFFFF;
  line-height: 25px;
}
</style>
