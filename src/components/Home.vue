<script setup lang="ts">
import { ethers } from "ethers";
import { reactive } from 'vue';
import HeaderView from "../components/Header.vue";

import { ElMessage } from "element-plus";

import BigNumber from "bignumber.js";

import { precision } from "../components/Types";

defineProps<{ msg: string }>()

let state = reactive({ input: '', contract: null, balance: '', epoch: 0 })

const contractAddr = "0xc7C6C6a1c8F6D86A85db1c5dD5B8072610d75D57";
const abi = [
  "function deposit(address addr)",
  "function withdraw()",
]

async function depositAction() {
  const accounts = await window.ethereum.request({ method: 'eth_requestAccounts' });
  const account = accounts[0];

  let contract = new ethers.Contract(contractAddr, abi, window.singer)

  contract.deposit(account, { value: ethers.parseEther(state.input) }).then((val) => {
    console.log(val)

    ElMessage.info("deposit successfully.")
  }).catch((err) => {
    console.log(err)
  })
}

async function withdrawAction() {
  let contract = new ethers.Contract(contractAddr, abi, window.singer)

  contract.withdraw().then((val) => {
    console.log(val)
    queryBal()

    ElMessage.info("withdraw successfully.")
  }).catch((err) => {
    console.log(err)
    queryBal()
  })
}

function queryBal() {
  if (window.provider) {
    const accounts = window.ethereum.request({ method: 'eth_requestAccounts' });
    const account = accounts[0];

    window.provider.getBalance(account).then((val: any) => {
      console.log(val)

      let bal = new BigNumber(val)
      let intt = new BigNumber(precision)
      let hh = bal.div(intt).toFixed(2)
      state.balance = hh
    })
  }
}
</script>

<template>
  <div>
    <HeaderView class="header-view" />

    <div class="search-view">
      <el-row>
        <a class="search-a" style="text-decoration: none;" href="javascript:void(0)" @click="depositAction"> Deposit </a>

        <el-input class="input-class" v-model="state.input" placeholder="deposit value(FIL)" type="number" maxlength="32"
          minlength="4" @keyup.enter.native="depositAction" />
      </el-row>

      <br>
      <br>

      <el-row>
        <a class="search-a" style="text-decoration: none;" href="javascript:void(0)" @click="withdrawAction"> Withdraw
        </a>

        <span class="balance-view" v-if="state.balance">{{ state.balance }}</span>
      </el-row>

    </div>
  </div>
</template>

<style scoped>
.header-view {
  width: 100%;
}

.search-view {
  max-width: 1200px;
  margin: 0 auto;
  margin-top: 40px;
}

.input-class {
  width: 80%;
  height: 64px;
  font-size: 20px;
  margin-left: 10px;
}

.search-a {
  display: block;
  width: 120px;
  height: 64px;
  background: #4540D6;
  border-radius: 8px;
  text-align: center;
  line-height: 64px;
  color: white;
}

.balance-view {
  line-height: 64px;
  margin-left: 10px;
}
</style>
