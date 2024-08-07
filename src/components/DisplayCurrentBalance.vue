<script setup lang="ts">
import { ref } from "vue";
import { ethers } from "ethers";
import { QBtn } from "Quasar";

const address = ref<string | null>(null);
const balance = ref<number | string | null>(null);

const connectWallet = async () => {
  if (window.ethereum) {
    try {
      const accounts = await (window as any).ethereum.request({
        method: "eth_requestAccounts",
      });
      address.value = accounts[0];
    } catch (error) {
      console.error("Error:", error);
    }
  } else {
    alert("MetaMask not found");
  }
};

const fetchBalance = async () => {
  if (!address.value) {
    alert("Please connect your wallet first.");
    return;
  }

  try {
    const provider = new ethers.BrowserProvider(window.ethereum);
    const balanceInWei = await provider.getBalance(address.value);
    balance.value = ethers.formatEther(balanceInWei);
  } catch (error) {
    console.error("Failed to fetch balance:", error);
  }
};
</script>
<template>
  <div class="wallet-container">
    <q-btn @click="connectWallet" color="primary" class="q-mb-md">
      Connect to MetaMask
    </q-btn>
    <p v-if="address" class="address">Wallet Address: {{ address }}</p>
    <q-btn @click="fetchBalance" color="secondary" class="q-mb-md">
      Display Current Balance
    </q-btn>
    <div v-if="balance !== null" class="balance-display">
      <p>Current Balance: {{ balance }} ETH</p>
    </div>
  </div>
</template>

<style scoped>
.wallet-container {
  max-width: 600px;
  margin: auto;
}

.wallet-container p {
  font-size: 1.2em;
  color: red;
}

.balance-display p {
  font-size: 1.2em;
  color: red;
}
</style>
