<script setup lang="ts">
import { QBtn, QCard, QCardSection } from "Quasar";
import { ref } from "vue";
import DockPanel from "./components/DockPanel.vue";
import AesEncryption from "./components/AesEncryption.vue";
import DisplayCurrentBalance from "./components/DisplayCurrentBalance.vue";
import WalletTransaction from "./components/WalletTransaction.vue";

// 1. Connecting the Metamask wallet to the site and displaying its address
const address = ref<string | null>(null);

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

// 2. Textarea block where you can enter text and by clicking the button get sha256 from the entered text
const text = ref("");
const hash = ref<string | null>(null);

const getHash = async () => {
  if (text.value) {
    try {
      const encoder = new TextEncoder();
      const data = encoder.encode(text.value);

      const hashBuffer = await crypto.subtle.digest("SHA-256", data);

      const hashArray = Array.from(new Uint8Array(hashBuffer));
      hash.value = hashArray
        .map((byte) => byte.toString(16).padStart(2, "0"))
        .join("");
    } catch (error) {
      console.error("Error:", error);
    }
  } else {
    hash.value = null;
  }
};

// 3. Dock panel, with a list of applications/sites, which opens when clicked. iframe on your application page. (sites  https://app.uniswap.org/ ,  https://app.1inch.io/ , bugs.denet.pro, revert.finance, you can add something of your own)
// DockPanel Component

const applications = [
  { name: "Uniswap", url: "https://app.uniswap.org/" },
  { name: "1inch", url: "https://app.1inch.io/" },
  { name: "Bugs Denet", url: "https://bugs.denet.pro" },
  { name: "Revert Finance", url: "https://revert.finance" },
  { name: "Google", url: "https://www.google.com" },
];

// 4. Block utility - textarea - the entered text is encrypted with the aes algorithm using a free or specified key; 5. Block utility - textarea - the entered text is decrypted by aes using a free or specified key
// AesEncryption Component

// 6. Loading a list of applications for the dock panel from a .json file (you can make it static in a free format)
// DockPanel Component - jsonFile="/apps.json"

// 7. Display balance in Polygon network or Kovan network. Test Kovan ETH can be requested from the team or from free faucets
// DisplayCurrentBalance Component

// 8. Sending a transaction from the connected wallet of the main token
// 9. Sending a transaction from a connected ERC20 custom token wallet
// WalletTransaction Component
</script>

<template>
  <q-card class="q-col-gutter-md column">
    <q-card-section class="q-pa-none">
      <p style="color: red">MetaMask Wallet</p>
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
      <p style="color: red">SHA-256</p>
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
      <p style="color: red">Dock Panel with provided Array Options</p>
      <DockPanel :appList="applications" />
    </q-card-section>
    <q-card-section>
      <p style="color: red">Encryption & Decryption</p>
      <aes-encryption />
    </q-card-section>
    <q-card-section>
      <p style="color: red">Dock Panel with provided JSON list</p>
      <DockPanel jsonFile="/apps.json" />
    </q-card-section>
    <q-card-section>
      <p style="color: red">Current Balance from MetaMask Wallet</p>
      <DisplayCurrentBalance />
    </q-card-section>
    <q-card-section>
      <p style="color: red">WalletTransaction for MetaMask Wallet</p>
      <WalletTransaction />
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
