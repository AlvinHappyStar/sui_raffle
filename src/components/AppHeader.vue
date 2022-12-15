<script setup>
import {onMounted, onUnmounted, watch} from "vue";
import {useWallet} from "../helpers/wallet";
import {useUiStore} from "../stores/ui";
import {moduleAddress, moduleName, casinoAddress, gameAddress} from "../helpers/constants";
const {
  walletProviders,
  isPermissionGranted, logout, permissionGrantedError,verifyWalletPermissions, requestWalletAccess, executeMoveCall, getAddress, getSuitableCoinId } = useWallet();
import { useDark, useToggle } from '@vueuse/core';
import {useAuthStore} from "../stores/auth";
import Modal from "./Modal.vue";

const uiStore = useUiStore();
const authStore = useAuthStore();

const isDark = useDark();
const toggleDark = useToggle(isDark);


onMounted(()=>{

  // we verify wallet permissions after window has loaded to make sure that
  // the wallet extensions have received their state and window[walletProvider] doesnt return undefined.
  window.addEventListener('load', verifyWalletPermissions);
});
onUnmounted(()=>{
  window.removeEventListener('load', verifyWalletPermissions);
})

watch(permissionGrantedError, (val) => {
  if(val) uiStore.setNotification(val);
});

watch(isPermissionGranted, val => {
  if(val){uiStore.setNotification('Successfully logged in.', 'success')}
});

const disconnect = () => {
  logout();
  state.profile = null;
}

onMounted(()=>{

})

const withdraw = async () => {

if(!authStore.casinoAdmin.isAdmin) return; // only admins can deposit to casino.
const address = getAddress();
if(!address) return;

return executeMoveCall({
      packageObjectId: moduleAddress,
      module: moduleName,
      typeArguments: [],
      arguments: [authStore.casinoAdmin.objectAddress, casinoAddress, gameAddress],
      function: 'draw',
      gasBudget: 1000
}).then(res=>{
  const status = res?.effects?.status?.status;

    if(status === 'success'){
      uiStore.setNotification("successfully withdraw", "success");
    }else{
      uiStore.setNotification(res?.effects?.status?.error);
    }
  
}).catch(e=>{
  uiStore.setNotification(e.message);
})
}

const reward = async () => {

const address = getAddress();
if(!address) return;

const coinId = getSuitableCoinId(0);

return executeMoveCall({
      packageObjectId: moduleAddress,
      module: moduleName,
      typeArguments: [],
      arguments: [casinoAddress, coinId],
      function: 'take_reward',
      gasBudget: 1000
}).then(res=>{
  const status = res?.effects?.status?.status;

    if(status === 'success'){
      uiStore.setNotification("successfully reward", "success");
      authStore.winner = 0;
    }else{
      uiStore.setNotification(res?.effects?.status?.error);
    }
  
}).catch(e=>{
  uiStore.setNotification(e.message);
})
}
</script>

<template>
  <header id="header" class="container">

      <div class="grid grid-cols-12 gap-5 py-6 items-center">

        <div class="col-span-4">

          <h3 class="text-2xl">
            SUI Raffle
          </h3>


        </div>

        <div class="col-span-8 justify-self-end flex items-center">


          <button @click="toggleDark()" class="mr-3">
            <svg v-if="!isDark" id="theme-toggle-dark-icon" class=" w-5 h-5" fill="currentColor" viewBox="0 0 20 20" xmlns="http://www.w3.org/2000/svg">
              <path d="M17.293 13.293A8 8 0 016.707 2.707a8.001 8.001 0 1010.586 10.586z"></path></svg>
            <svg v-if="isDark" id="theme-toggle-light-icon" class=" w-5 h-5" fill="currentColor" viewBox="0 0 20 20" xmlns="http://www.w3.org/2000/svg">
              <path d="M10 2a1 1 0 011 1v1a1 1 0 11-2 0V3a1 1 0 011-1zm4 8a4 4 0 11-8 0 4 4 0 018 0zm-.464 4.95l.707.707a1 1 0 001.414-1.414l-.707-.707a1 1 0 00-1.414 1.414zm2.12-10.607a1 1 0 010 1.414l-.706.707a1 1 0 11-1.414-1.414l.707-.707a1 1 0 011.414 0zM17 11a1 1 0 100-2h-1a1 1 0 100 2h1zm-7 4a1 1 0 011 1v1a1 1 0 11-2 0v-1a1 1 0 011-1zM5.05 6.464A1 1 0 106.465 5.05l-.708-.707a1 1 0 00-1.414 1.414l.707.707zm1.414 8.486l-.707.707a1 1 0 01-1.414-1.414l.707-.707a1 1 0 011.414 1.414zM4 11a1 1 0 100-2H3a1 1 0 000 2h1z"
                                                                                                                                                               fill-rule="evenodd" clip-rule="evenodd">
            </path>
            </svg>
          </button>

          <button v-if="authStore.casinoAdmin.isAdmin" class="bg-gray-800 dark:bg-gray-800 flex items-center text-white px-5 py-2 mr-2 rounded-full"
                  @click="withdraw">Withdraw</button>
          <button v-if="authStore.winner != 0" class="bg-gray-800 dark:bg-gray-800 flex items-center text-white px-5 py-2 mr-2 rounded-full"
                  @click="reward">Reward</button>

          <button v-if="!authStore.hasWalletPermission"
                  class="bg-gray-800 dark:bg-gray-800 flex items-center text-white px-5 py-2 rounded-full"
                  @click="authStore.toggleWalletAuthModal = true"
          >
            Connect Your Wallet
          </button>

          <button v-else class="bg-gray-300 text-gray-900 px-3 py-2 rounded-full" @click="disconnect">
            Logout
          </button>
        </div>
      </div>


    <Modal body-classes="!w-[400px] !max-w-[95%]" v-if="authStore.toggleWalletAuthModal && !authStore.hasWalletPermission"
           @closeModal="authStore.toggleWalletAuthModal = false">


      <p class="text-center mb-6">Please select your wallet provider</p>
      <button v-for="provider in walletProviders" :key="provider.key"
              class="w-full justify-center bg-gray-200 dark:bg-gray-700 flex items-center px-5 py-3 rounded-lg my-3"
      @click="requestWalletAccess(provider.key)">
          <div v-html="provider.logo" class="logo-icon"></div> Connect {{provider.title}}
      </button>




    </Modal>
  </header>
</template>

<style>
.logo-icon svg{
  @apply w-[20px] h-[20px]
}
</style>

