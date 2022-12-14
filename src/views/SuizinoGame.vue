<script setup>
import {ref, onMounted,onUnmounted, reactive} from 'vue'
import {logo} from "../assets/icons";
import {useAuthStore} from "../stores/auth";
import {casinoAddress, moduleAddress, gameAddress} from "../helpers/constants";
import {useWallet} from "../helpers/wallet";
import {useUiStore} from "../stores/ui";

const authStore = useAuthStore();
const uiStore = useUiStore();
const {executeMoveCall, getAddress, getSuitableCoinId} = useWallet();

const isLoading = ref(false);
const isTicketRemain = ref(0);

let initialSpinInterval = ref();

onMounted(()=>{
  initialSpinInterval.value = setupSpinningInterval(120);
});


const executeGamble = () => {
  const address = getAddress();
  if(!address) return;
  resetGame();
  isLoading.value = true;

  const coinId = getSuitableCoinId(10);
  console.log(coinId);

  console.log(authStore.ticketnum);
  if(authStore.ticketnum == 5)
  {
    uiStore.setNotification("You don't buy ticket any more")
    resetGame();
  }
  else
  {
    executeMoveCall({
    packageObjectId: moduleAddress,
    module: 'Base',
    typeArguments: [],
    arguments: [casinoAddress, gameAddress, coinId, 1],
    function: 'buy_ticket',
    gasBudget: 1000
  })
  .then(res =>{
    const status = res?.effects?.status?.status;

    console.log(res);

    if(status === 'success'){      
        uiStore.setNotification("You bought", "success")
    }else{
      let e = res?.effects?.status?.error;
      
      if(e.includes('2)'))
      {
        uiStore.setNotification("No Tickets");
        isTicketRemain.value = 1;
      }
      else
        uiStore.setNotification(e);
          
      resetGame();
    }
  }).catch(e=>{
    resetGame();

    uiStore.setNotification(e.message);
  });
  }  
}


const setupSpinningInterval = (timeout) => {
  return setInterval(()=> {
    
  }, timeout);
}

const clearSpinningInterval = () => {
  clearInterval(initialSpinInterval.value);
}

onUnmounted(()=>{
  clearSpinningInterval();
});

const resetGame = () => {
  clearSpinningInterval();
  isLoading.value = false;
  setupSpinningInterval(120);
}

</script>

<style>

.lucky-wheel-slot.win{
  @apply border-2 border-green-600;
}
.lucky-wheel-slot.loss{
  @apply border-2 border-red-700;
}

.depth-front {
    -webkit-box-align: center;
    align-items: center;
    background: linear-gradient(to left, rgb(0, 0, 0), rgb(0, 0, 0));
    border-radius: 100%;
    display: flex;
    height: 10em;
    -webkit-box-pack: center;
    justify-content: center;
    transform-style: preserve-3d;
    width: 10em;
    margin:0px auto;
}

.gQRSty {
    background: rgb(42, 116, 202);
    border: 2px solid rgb(250, 250, 250);
    padding: 12px;
    border-top-left-radius: 35px;
    border-top-right-radius: 35px;
    display: -webkit-flex;
    gap: 0.75rem;
    overflow: hidden;
    width: 100%;
    z-index: 10;
    max-width: 700px;
    margin-top: 1.5rem
    
}

.cUYTSd {
    -webkit-box-align: center;
    align-items: center;
    background: rgb(0, 0, 0);
    border-radius: 4px;
    display: flex;
    gap: 2px;
    height: 40px;
    -webkit-box-pack: center;
    justify-content: center;
    position: relative;
    padding: 2px;
    line-height: 0;
    width: 100%;
    z-index: 0;
}

.jyukYf {
    -webkit-box-align: center;
    align-items: center;
    background: linear-gradient(90deg, rgb(255, 255, 255) 0%, rgb(158, 103, 150) 100%);
    border: 1px solid rgb(248, 202, 34);    
    border-radius: 2px;
    color: rgb(23, 23, 23);
    display: flex;
    flex: 1 1 0%;
    gap: 4px;
    height: 100%;
    -webkit-box-pack: center;
    justify-content: center;
    line-height: 1.5rem;
    padding: 0px 16px;
    text-align: center;
    text-transform: uppercase;
    transition: all 300ms ease-in-out 0s;
    user-select: none;
    white-space: nowrap;
}

.jQzBaH {
    -webkit-box-align: center;
    align-items: center;
    background: none;
    border: 1px solid rgb(248, 202, 34);
    border-left-width: 0;
    border-top-width: 0;
    border-bottom-width: 0;
    border-radius: 2px;
    color: white;
    display: flex;
    flex: 1 1 0%;
    gap: 4px;
    height: 100%;
    -webkit-box-pack: center;
    justify-content: center;
    line-height: 1.5rem;
    padding: 0px 16px;
    text-align: center;
    text-transform: uppercase;
    transition: all 300ms ease-in-out 0s;
    user-select: none;
    white-space: nowrap;
}

.gQRSty .coin-side {
    width: fit-content;
}

.gQRSty > div:not(:first-child) {
    width: calc(50% - 0.375rem);
}


.bZJbip {
  -webkit-box-align: center;
  align-items: center;
  background: rgb(38, 48, 59);
  border-radius: 0px 0px 35px 35px;
  border-right: 2px solid rgb(0, 0, 0);
  border-bottom: 2px solid rgb(0, 0, 0);
  border-left: 2px solid rgb(0, 0, 0);
  border-image: initial;
  border-top: none;
  display: flex;
  flex-direction: column;
  -webkit-box-pack: justify;
  justify-content: space-between;
  max-width: 700px;
  padding: 0px 0px 28px;
  position: relative;
  width: 100%;
}

.jJnyom {
  display: flex;
  flex-direction: column;
  gap: 12px;
  list-style: none;
  overflow-y: scroll;
  padding: 20px 20px 24px;
  width: 100%;
  height: calc(100vh - 680px);
  min-height: 200px;
}

.jJnyom li {
  border-bottom: 1px solid rgb(23, 23, 23);
  display: flex;
  gap: 100px;
  padding: 0px 0px 12px;
  width: 100%;
  font-size: 0.875rem;
  color: rgb(255, 255, 255);
  text-align: center;
}

.jJnyom li span:nth-last-child(1) {
  flex: 1 1 0%;
  text-align: right;
}
</style>

<template>
  <main class="container">

<!--    <h2>Please login to continue</h2>-->
    <div class="max-w-[700px] mx-auto mb-20 mt-6">

      <div class="py-6 text-center">
        <h2 class="text-3xl font-bold">
          Welcome to SUI Raffle
        </h2>

      </div>
      


      <div class="mt-6 text-center">

        <button v-if="!authStore.hasWalletPermission" class="bg-gray-800 mx-auto ease-in-out duration-500 hover:px-10 dark:bg-gray-800 flex items-center text-white px-5 py-2 rounded-full" @click="authStore.toggleWalletAuthModal = true">
          <div v-html="logo" class="logo-icon"></div> Connect your wallet to start playing!
        </button>
        <button v-else
                class="bg-gray-800 mx-auto ease-in-out duration-500 hover:px-10 dark:bg-gray-800 flex items-center text-white px-5 py-2 rounded-full"
                :class="isLoading || !authStore.hasWalletPermission ? 'opacity-70 cursor-default hover:px-5': ''"
                @click="executeGamble">

          <div v-if="isLoading">
            <svg aria-hidden="true" class="w-5 h-5 text-gray-200 animate-spin dark:text-white fill-gray-800" viewBox="0 0 100 101" fill="none" xmlns="http://www.w3.org/2000/svg">
              <path d="M100 50.5908C100 78.2051 77.6142 100.591 50 100.591C22.3858 100.591 0 78.2051 0 50.5908C0 22.9766 22.3858 0.59082 50 0.59082C77.6142 0.59082 100 22.9766 100 50.5908ZM9.08144 50.5908C9.08144 73.1895 27.4013 91.5094 50 91.5094C72.5987 91.5094 90.9186 73.1895 90.9186 50.5908C90.9186 27.9921 72.5987 9.67226 50 9.67226C27.4013 9.67226 9.08144 27.9921 9.08144 50.5908Z"
                    fill="currentColor"/>
              <path d="M93.9676 39.0409C96.393 38.4038 97.8624 35.9116 97.0079 33.5539C95.2932 28.8227 92.871 24.3692 89.8167 20.348C85.8452 15.1192 80.8826 10.7238 75.2124 7.41289C69.5422 4.10194 63.2754 1.94025 56.7698 1.05124C51.7666 0.367541 46.6976 0.446843 41.7345 1.27873C39.2613 1.69328 37.813 4.19778 38.4501 6.62326C39.0873 9.04874 41.5694 10.4717 44.0505 10.1071C47.8511 9.54855 51.7191 9.52689 55.5402 10.0491C60.8642 10.7766 65.9928 12.5457 70.6331 15.2552C75.2735 17.9648 79.3347 21.5619 82.5849 25.841C84.9175 28.9121 86.7997 32.2913 88.1811 35.8758C89.083 38.2158 91.5421 39.6781 93.9676 39.0409Z"
                    fill="currentFill"/>
            </svg>
          </div>
          <span v-else class="flex items-center">
            <div v-html="logo" class="logo-icon"></div>
            {{'BUY TICKET'}} <span class="ml-2 text-sm">
        </span>
          </span>

        </button>
        <!-- <p class="text-xs mt-5">If you get all 3 slots equal, you win 25.000 MIST</p> -->
      </div>
    </div>
  </main>
</template>
