<script setup lang="ts">
import { ref } from "vue";
import { ethers } from "ethers";
import { QBtn, QInput, QSelect } from "Quasar";

const address = ref<string | null>(null);
const recipientAddress = ref<string>("");
const amountToSend = ref<number>(0);
const erc20ContractAddress = ref<string>("");
const transactionId = ref<string | null>(null);
const selectedType = ref<"ETH" | "ERC20">("ETH");

const transactionOptions = ref([
  { label: "ETH", value: "ETH" },
  { label: "ERC20", value: "ERC20" },
]);

const connectWallet = async () => {
  if (window.ethereum) {
    try {
      const provider = new ethers.BrowserProvider(window.ethereum);
      const accounts = await provider.send("eth_requestAccounts", []);
      address.value = accounts[0];
    } catch (error) {
      console.error("Error connecting to MetaMask:", error);
      alert("Failed to connect MetaMask. Check console for details.");
    }
  } else {
    alert("MetaMask is not installed. Please install MetaMask.");
  }
};

const executeTransaction = async () => {
  if (!address.value || !recipientAddress.value || !amountToSend.value) {
    alert("Please fill out all required fields and connect your wallet.");
    return;
  }

  try {
    const provider = new ethers.BrowserProvider(window.ethereum);
    const signer = provider.getSigner();

    if (selectedType.value === "ETH") {
      // Sending ETH
      const tx = {
        to: recipientAddress.value,
        value: ethers.parseEther(amountToSend.value.toString()),
      };

      const txResponse = await signer.sendTransaction(tx);
      transactionId.value = txResponse.hash;
      await txResponse.wait();
      alert("ETH transaction successful!");
    } else if (selectedType.value === "ERC20") {
      // Sending ERC20 Tokens
      if (!erc20ContractAddress.value) {
        alert("ERC20 contract address is required.");
        return;
      }

      const erc20Abi = [
        "function transfer(address recipient, uint256 amount) public returns (bool)",
      ];
      const tokenContract = new ethers.Contract(
        erc20ContractAddress.value,
        erc20Abi,
        signer
      );

      const txResponse = await tokenContract.transfer(
        recipientAddress.value,
        ethers.parseUnits(amountToSend.value.toString(), 18)
      );
      transactionId.value = txResponse.hash;
      await txResponse.wait();
      alert("ERC20 token transaction successful!");
    }
  } catch (error) {
    console.error("Transaction error:", error);
    alert("Transaction failed. Check the console for details.");
  }
};
</script>
<template>
  <div class="send-transaction">
    <q-btn @click="connectWallet" color="primary" class="q-mb-md">
      Connect To MetaMask
    </q-btn>
    <p v-if="address">Wallet: {{ address }}</p>
    <q-select
      v-model="selectedType"
      :options="transactionOptions"
      label="Select Transaction Type"
      color="primary"
      class="q-mb-md"
      style="color: red"
    />
    <q-input
      v-model="recipientAddress"
      label="Recipient Address"
      class="q-mb-md"
    />
    <q-input
      v-model.number="amountToSend"
      type="number"
      label="Amount"
      class="q-mb-md"
    />
    <q-input
      v-if="selectedType === 'ERC20'"
      v-model="erc20ContractAddress"
      label="ERC20 Contract Address"
      class="q-mb-md"
    />
    <q-btn @click="executeTransaction" color="secondary">
      Send Transaction
    </q-btn>
    <p v-if="transactionId" class="q-mt-md">
      Transaction Hash:
      <a :href="`https://etherscan.io/tx/${transactionId}`" target="_blank">{{
        transactionId
      }}</a>
    </p>
  </div>
</template>

<style scoped>
.send-transaction {
  max-width: 600px;
  margin: auto;
  padding: 20px;
  border: 1px solid #ddd;
  border-radius: 8px;
}

.send-transaction p {
  font-size: 1.2em;
  color: #333;
}
</style>
