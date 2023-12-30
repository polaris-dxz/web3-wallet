<template>
  <h1>账户信息</h1>
  <van-divider />
  <p>address: {{ address }}</p>
  <p>privateKey: {{ privateKey }}</p>
  <p>mount: {{ mount }}</p>
  <h1>转账操作</h1>
  <van-divider />
  <van-button type="primary" @click="transaction">开始转账</van-button>
</template>

<script setup> 
import { ref } from 'vue'
import Web3 from 'web3';
import Tx from 'ethereumjs-tx'

const address = ref("0x5ff2b512A49b0718D45C09C5fD60f4c4D2d765D6")
const privateKey = ref("0x2a3d7f1985c928c956ac5e40da831ba55458ad3f56677028070f0091b79cd217")

// 创建 web3 实例
const web3 = new Web3(Web3.givenProvider || "wss://goerli.infura.io/ws/v3/286e8e81bf9346b4abe7766d6fcf1c2f");
console.log('web3', web3)

// 创建账户（不用每次都创建 ）
// const account =  web3.eth.accounts.create('123')
// console.log('account', account)

// 余额获取
const mount = ref(0)
web3.eth.getBalance(address.value).then(res => {
  mount.value = web3.utils.fromWei(res, 'ether')
})
console.log('mount', mount)

// 单位转换
// const num1 = Web3.utils.toWei('0.3')
// const num2 = web3.utils.toWei('0.3')
const num3 = web3.utils.fromWei(3000000, 'ether')
console.log(num3)

// 转账操作
const transaction = async () => {
  //  1. 构建转账参数
  // 获取账号转账交易次数
  const nonce = await web3.eth.getTransactionCount(address.value)

  // 获取预计转账 gas 费 (wei)
  const gasPrice = await web3.eth.getGasPrice()

  // 转账金额，以 wei 为单位
  const value = web3.utils.toWei("0.01")

  // 转账
  const rawTx = {
    from: address.value,
    to:  '',
    nonce,
    gasPrice,
    value,
    data: '0x0000', // 代理
  }

  // 2. 生成 序列化 tx
  // 转换私钥
  const pKey = Buffer(privateKey.value.slice(2), "hex")
  console.log(pKey)

  // gas 换算
  const gas = await web3.eth.estimateGas(rawTx)
  rawTx.gas =  gas

  // ethereumjs-tx 私钥加密
  const tx = new Tx(rawTx)
  tx.sign(pKey)

  // 生成 serializedTx
  const serializedTx = '0x' + tx.serialize().toString('hex')
  console.log('serializedTx', serializedTx)

  // 3. 发送转账交易，获取交易 ID
  const trans = web3.eth.sendSignedTransaction(serializedTx)

  // 监听事件
  trans.on('transactionHash', (txId) => {
    console.log('交易 id', txId)
    console.log(`https://goerli.etherscan.io/tx/${txId}`)
  })

  trans.on('receipt', res => {
    console.log('第一个节点确认', res)
  })

  trans.on('confirmation',() => {
    console.log('第n个节点确认')
  })

}
</script>

<style lang="less">
</style>
