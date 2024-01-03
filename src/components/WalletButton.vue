<template>
  <van-space>
    <van-button type="primary" @click="createWallet">创建钱包</van-button>
    <van-button type="primary" @click="importWallet">导入钱包</van-button>
  </van-space>

  <van-dialog v-model:show="showCreateWalletDialog" title="请输入密码" show-cancel-button @confirm="confirmPassword">
    <van-cell-group inset>
      <van-field
        v-model="password"
        type="password"
        name="密码"
        label="密码"
        placeholder="密码"
        :rules="[{ required: true, message: '请填写密码' }]"
      />
    </van-cell-group>
  </van-dialog>
</template>

<script setup> 
import { ref } from 'vue';
import 'vant/es/dialog/style'
import Web3 from 'web3';

const showCreateWalletDialog = ref(false)
const password = ref("")
const mnemonic = ref('resemble trend evolve tiny jewel unit cable like pelican away olympic mango')


const createWallet = () => {
  showCreateWalletDialog.value = true
}

const confirmPassword = () => {
  alert(password.value)
  if (!password.value) {
    return
  }
}

const importWallet = () => {

}

</script>

<style lang="less"></style>
