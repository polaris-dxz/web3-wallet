<template>
  <h1>账户系统</h1>
  <van-divider />
  <p>助记词: {{ mnemonic }}</p>
  <p>路径: {{ derivePath }}</p>
  <p>账号地址: {{ address }}</p>
  <p>账号私钥: {{ privateKey }}</p>
</template>

<script setup> 
import { ref } from 'vue'
import Web3 from 'web3';
import * as bip39 from 'bip39';
import ethwallet, { hdkey } from 'ethereumjs-wallet';

// 创建助记词
const mnemonic = ref('resemble trend evolve tiny jewel unit cable like pelican away olympic mango')
// mnemonic.value = bip39.generateMnemonic()
console.log('mnemonic', mnemonic)

// bip32 路径中定义了以下五个级别：m/purpse'/coin_type'/account'/change/address_index
const derivePath = ref("m/44'/60'/0'/0/0")
const address = ref('')
const privateKey = ref('')

const web3 = new Web3(Web3.givenProvider || "wss://goerli.infura.io/ws/v3/286e8e81bf9346b4abe7766d6fcf1c2f");

// 生成密钥对 key-pair
const generateMnemonic = async () => {
  // 根据助记词生成密钥对 key-pair
  const seed =  await bip39.mnemonicToSeed(mnemonic.value)
  console.log('seed', seed)
  
  const hdWallet = hdkey.fromMasterSeed(seed)
  const keyPair = hdWallet.derivePath(derivePath.value) // 一个助记词可以创建 n 个账号
  console.log('keyPair', keyPair)

  // 获取私钥
  // 1. 获取钱包对象
  const wallet = keyPair.getWallet()

  // 2. 获取钱包地址
  const lowercaseAddress = wallet.getAddressString()
  console.log('lowercaseAddress', lowercaseAddress)

  // 3. 获取钱包的校验地址(校验码是大写)
  const checkAddress = wallet.getChecksumAddressString()
  console.log('checkAddress', checkAddress)

  // 4. 获取私钥(没有 0x)
  const pKey= wallet.getPrivateKey().toString("hex")
  console.log('privateKey', pKey)

  // 导出 keystore
  const password = "111111"
  
  // 1. web3.js
  const keystore1 = web3.eth.accounts.encrypt(pKey, password)
  console.log("keystore1", JSON.stringify(keystore1))

  // 2. wallet 对象
  const keystore2 = await wallet.toV3(password)
  console.log("keystore2", JSON.stringify(keystore2))

  // 通过 keystore 获取私钥
  // 1. web3
  const res = web3.eth.accounts.decrypt(keystore1, password)
  console.log("keystore1 privateKey", res.privateKey)

  // 2. wallet
  const res2 = await ethwallet.fromV3(keystore2, password)
  const key = res2.getPrivateKey().toString("hex")
  console.log("keystore2 privateKey", key)

  // 通过私钥获取地址
  const priKey = Buffer(pKey, "hex")
  console.log('privateKey Buffer', priKey)

  const wallet3 = ethwallet.fromPrivateKey(priKey)
  const lowercaseAddress2 = wallet.getAddressString()
  console.log('lowercaseAddress2', lowercaseAddress2)
}

generateMnemonic()


</script>

<style lang="less">
</style>
