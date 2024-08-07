<script setup lang="ts">
import { QBtn } from "Quasar";
import { ref } from "vue";
import DockPanel from "./components/DockPanel.vue";

// 1. Connecting the Metamask wallet to the site and displaying its address
const address = ref<string | null>(null);

const connectWallet = async () => {
  if (window.ethereum) {
    try {
      // Request account access
      const accounts = await (window as any).ethereum.request({
        method: "eth_requestAccounts",
      });
      // Set the first account as the connected address
      address.value = accounts[0];
    } catch (error) {
      console.error(
        "User denied account access or other error occurred:",
        error
      );
    }
  } else {
    alert("MetaMask is not installed. Please install MetaMask and try again.");
  }
};

// 2. Textarea block where you can enter text and by clicking the button get sha256 from the entered text
const text = ref("");
const hash = ref<string | null>(null);

// Function to compute SHA-256 hash
const getHash = async () => {
  if (text.value) {
    try {
      // Convert text to ArrayBuffer
      const encoder = new TextEncoder();
      const data = encoder.encode(text.value);

      // Compute the hash
      const hashBuffer = await crypto.subtle.digest("SHA-256", data);

      // Convert hash to hexadecimal string
      const hashArray = Array.from(new Uint8Array(hashBuffer));
      hash.value = hashArray
        .map((byte) => byte.toString(16).padStart(2, "0"))
        .join("");
    } catch (error) {
      console.error("Error computing hash:", error);
    }
  } else {
    hash.value = null;
  }
};
</script>

<template>
  <q-card class="q-col-gutter-md column">
    <q-card-section class="q-pa-none">
      <p>MetaMask Wallet</p>
      <div class="column items-center on-right btn">
        <q-btn
          color="primary"
          label="Connect MetaMask"
          @click="connectWallet"
        ></q-btn>
        <p v-if="address" class="q-ma-sm">
          Connected Wallet Address: {{ address }}
        </p>
      </div>
    </q-card-section>
    <q-card-section>
      <p>SHA-256</p>
      <textarea
        v-model="text"
        rows="4"
        cols="50"
        placeholder="Enter text here..."
      ></textarea>
      <br />
      <q-btn color="primary" label="Get SHA-256 Hash" @click="getHash"></q-btn>
      <p v-if="hash">SHA-256 Hash: {{ hash }}</p>
    </q-card-section>
    <q-card-section>
      <dock-panel />
    </q-card-section>
  </q-card>
</template>

<style scoped>
.q-card {
  max-width: 100vw;
}

.btn {
  display: flex;
  align-items: center;
  justify-content: flex-end;
}

.q-btn {
  margin-right: 16px;
}
</style>
