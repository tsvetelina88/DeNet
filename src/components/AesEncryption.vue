<script setup lang="ts">
import { ref } from "vue";
import { QBtn, QCard, QCardSection, QInput } from "Quasar";

// 32-byte key for AES-256
const secretKey = new Uint8Array([
  0x00, 0x01, 0x02, 0x03, 0x04, 0x05, 0x06, 0x07, 0x08, 0x09, 0x0a, 0x0b, 0x0c,
  0x0d, 0x0e, 0x0f, 0x10, 0x11, 0x12, 0x13, 0x14, 0x15, 0x16, 0x17, 0x18, 0x19,
  0x1a, 0x1b, 0x1c, 0x1d, 0x1e, 0x1f,
]);

const text = ref("");
const encryptedText = ref("");
const decryptedText = ref("");

// Encryption and Decryption Logic
const encryptText = async () => {
  if (text.value) {
    try {
      const algorithm = { name: "AES-GCM", length: 256 };
      const key = await crypto.subtle.importKey(
        "raw",
        secretKey,
        algorithm,
        false,
        ["encrypt"]
      );

      const encoder = new TextEncoder();
      const encodedText = encoder.encode(text.value);
      const iv = crypto.getRandomValues(new Uint8Array(12));

      const encryptedBuffer = await crypto.subtle.encrypt(
        { name: "AES-GCM", iv: iv },
        key,
        encodedText
      );

      encryptedText.value = `${arrayBufferToBase64(iv)}:${arrayBufferToBase64(
        encryptedBuffer
      )}`;
    } catch (error) {
      console.error("Error:", error);
    }
  }
};

const decryptText = async () => {
  if (encryptedText.value) {
    try {
      const [ivBase64, encryptedBase64] = encryptedText.value.split(":");
      const iv = base64ToArrayBuffer(ivBase64);
      const encryptedBuffer = base64ToArrayBuffer(encryptedBase64);

      const algorithm = { name: "AES-GCM", length: 256 };
      const key = await crypto.subtle.importKey(
        "raw",
        secretKey,
        algorithm,
        false,
        ["decrypt"]
      );

      const decryptedBuffer = await crypto.subtle.decrypt(
        { name: "AES-GCM", iv: iv },
        key,
        encryptedBuffer
      );

      const decoder = new TextDecoder();
      decryptedText.value = decoder.decode(decryptedBuffer);
    } catch (error) {
      console.error("Error:", error);
    }
  }
};

const arrayBufferToBase64 = (buffer: ArrayBuffer) => {
  let binary = "";
  const bytes = new Uint8Array(buffer);
  for (let i = 0; i < bytes.length; i++) {
    binary += String.fromCharCode(bytes[i]);
  }
  return window.btoa(binary);
};

const base64ToArrayBuffer = (base64: string) => {
  const binaryString = window.atob(base64);
  const len = binaryString.length;
  const bytes = new Uint8Array(len);
  for (let i = 0; i < len; i++) {
    bytes[i] = binaryString.charCodeAt(i);
  }
  return bytes.buffer;
};
</script>
<template>
  <q-card class="q-ma-md">
    <q-card-section>
      <q-input
        v-model="text"
        label="Enter Text"
        type="textarea"
        rows="5"
      ></q-input>
    </q-card-section>

    <q-card-section>
      <q-btn @click="encryptText" color="primary" label="Encrypt Input"></q-btn>
      <q-btn
        @click="decryptText"
        color="secondary"
        label="Decrypt Input"
      ></q-btn>
    </q-card-section>

    <q-card-section>
      <q-input
        v-model="encryptedText"
        label="Encryption Output"
        readonly
        type="textarea"
        rows="5"
      ></q-input>
    </q-card-section>

    <q-card-section>
      <q-input
        v-model="decryptedText"
        label="Decryption Output"
        readonly
        type="textarea"
        rows="5"
      ></q-input>
    </q-card-section>
  </q-card>
</template>
<style scoped></style>
