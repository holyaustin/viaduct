<script setup lang="ts">
import { onMounted, ref } from 'vue';
import { ConnectWalletButton, useMetaMaskWallet, Jazzicon } from 'vue-connect-wallet'
import { useWeb3Store } from '../stores/web3Store';
import { ethers } from 'ethers'
import { storeToRefs } from 'pinia';
import SafeImage from '~/components/SafeImage.vue';
import { loadHistory } from '~/utils/history';
import CryptoJS from 'crypto-js';
// import { connectors }from './banana.vue'

const wallet = useMetaMaskWallet()
const store = useWeb3Store()

const { address, transactions } = storeToRefs(store)

const open = ref(false)

const history = ref([] as any[])

onMounted(init)

async function init() {
    const accounts = await wallet.getAccounts()
    if (accounts.length > 0) connectWallet()
    wallet.onAccountsChanged(() => connectWallet())
    wallet.onChainChanged(() => connectWallet())

    history.value = loadHistory()
}

async function connectWallet() {
    console.log({ address: address.value, open: open.value });

    if (address.value) {
        open.value = true
    }

    console.log({ address: address.value, open: open.value });
    const accounts = await wallet.connect()
    if (typeof accounts === 'string') return

    const web3Provider = new ethers.providers.Web3Provider(window.ethereum, 'any')
    const chainId = (await web3Provider.getNetwork()).chainId

    store.$patch({
        address: accounts[0],
        chainId
    })
}

function closeModal() {
    open.value = false
}
</script>

<template>
    <nav class="bg-blue-900 m-2 rounded-lg ">
            <div class="h-6 bg-gradient-to-r from-primary-500 to-primary-600 rounded-b-lg">
        </div>
        <div class="flex justify-between items-center px-14 py-2 mx-">
            <img src="/favicon.ico" alt="logo" width="100"  >
            <div class="py-2">
                <h1 class="logo">Viaduct Protocol</h1>
                <h3 class="px-3 text-gray-300">  XChain Bridging for DeFi </h3>

            </div>



            <ul class="list-none flex space-x-2 items-center text-2xl">
                <li class="z-0">               
                <a href="https://sandbox.monerium.dev/partners/661c0725-44d3-11ed-adac-b2efc0e6677d/auth?code_challenge=K-A2XhPbn1C3-6hRG-3TAVEyg9MOchELRwhOV8bScj8&code_challenge_method=S256&response_type=code&client_id=654c9c30-44d3-11ed-adac-b2efc0e6677d&redirect_uri=https://viaduct-protocol.vercel.app/" target="_blank">
                Euro-OnRamp
                </a>
                   
                </li>
            </ul>

            <ul class="list-none flex space-x-2 items-center text-2xl">
                <li class="z-0">

                <a href="https://create.request.network/" target="_blank">
                Request
                </a>
                   
                </li>
            </ul>

                        <ul class="list-none flex space-x-2 items-center text-2xl">
                <li class="z-0">

                <a href="https://app.buidler.etherspot.io/" target="_blank">
                Etherspot
                </a>
                   
                </li>
            </ul>
                        <ul class="list-none flex space-x-2 items-center">
                <li class="z-0">
                    <button @click=""
                        class="w-full mt-2 bg-primary-500 hover:bg-primary-600 transition rounded-lg py-2 px-3 font-bold text-xl">Banana Wallet</button>
                </li>
            </ul>
            <ul class="list-none flex space-x-2 items-center">
                <li class="z-0">
                    <ConnectWalletButton @click="connectWallet" :address="address ?? ''" :dark="true" :txn-count="transactions.length" class="w-full transition rounded-lg px-3 font-bold text-base" />
                </li>
            </ul>
        </div>
        <div class="h-6 bg-gradient-to-r from-primary-500 to-primary-600 rounded-b-lg">
        </div>
    </nav>
    <Modal v-model="open" :close="closeModal">
        <div class="bg-true-gray-500 text-white rounded-lg w-5/12">
            <header class="p-5 bg-dark-500 rounded-t-lg flex items-center justify-between" v-if="address">
                <div class="md:(flex items-center space-x-3)">
                    <Jazzicon :address="address" diameter="40" />
                    <h1 :title="address" class="font-bold mb-1"> {{
                            address.substring(0, 10) +
                            "..." +
                            address.substring(address.length - 9)
                    }}
                    </h1>
                </div>
                <div class="uppercase border border-primary-500 text-primary-500 px-3 py-1.5 rounded-full text-xs">
                    Connected
                </div>
            </header>
            <div class="p-5 overflow-y-auto max-h-96">
                <h1 class="font-semibold">Past transactions</h1>
                <div class="mt-1">
                    <div v-for="item in history" :key="item.hash" class=" border border-gray-500 rounded-lg p-5 my-3">
                        <div class="flex items-center justify-between">
                            <div>
                                <div class="flex items-center space-x-3">
                                    <div class="flex items-center space-x-1.5">
                                        <SafeImage :src="item.fromChain.icon" class="w-5 h-5 rounded-full"
                                            :title="item.fromChain.name" />
                                        <h1 class="text-sm">{{ item.fromChain.name }}</h1>
                                    </div>
                                    <div>&rarr;</div>
                                    <div class="flex items-center space-x-1.5">
                                        <SafeImage :src="item.toChain.icon" class="w-5 h-5 rounded-full"
                                            :title="item.toChain.name" />
                                        <h1 class="text-sm">{{ item.toChain.name }}</h1>
                                    </div>
                                </div>

                                <div class="mt-3 flex flex-col">
                                    <div class="flex items-center space-x-1.5">
                                        <SafeImage :src="item.fromToken.icon" class="w-5 h-5 rounded-full"
                                            :title="item.fromToken.name" />
                                        <p class="text-xl font-mono">{{ item.fromToken.amount }}</p>
                                    </div>
                                    <div class="ml-0.5">&darr;</div>
                                    <div class="flex items-center space-x-1.5">
                                        <SafeImage :src="item.toToken.icon" class="w-5 h-5 rounded-full"
                                            :title="item.toToken.name" />
                                        <p class="text-xl font-mono">{{ item.toToken.amount }}</p>
                                    </div>
                                </div>

                            </div>
                            <div>
                                <div class="text-sm">
                                    {{ new Date(item.time).toDateString() }}
                                </div>
                            </div>
                        </div>
                        <footer class="mt-3 text-xs font-mono text-primary-400">
                            {{ item.hash }}
                        </footer>

                    </div>
                </div>
            </div>
        </div>
    </Modal>
</template>

<style scoped>
.logo {
    @apply bg-gradient-to-r from-primary-300 to-primary-400 px-3 py-1 tracking-widest font-black text-4xl;
    -webkit-background-clip: text;
    background-clip: text;
    -webkit-text-fill-color: transparent;
}
</style>