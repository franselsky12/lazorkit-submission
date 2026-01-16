# 📚 LazorKit Integration Tutorials

## Tutorial 1: How to Add Passkey Login
To enable biometric login in your app, follow these 3 simple steps:

**Step 1: Install the SDK**
```bash
npm install @lazorkit/wallet @solana/web3.js
import { LazorkitClient } from '@lazorkit/wallet';

const client = new LazorkitClient({
  network: 'devnet',
  rpcUrl: '[https://api.devnet.solana.com](https://api.devnet.solana.com)'
});
const handleLogin = async () => {
  const user = await client.connect();
  console.log("User logged in:", user.address);
};
const client = new LazorkitClient({
  network: 'devnet',
  rpcUrl: '[https://api.devnet.solana.com](https://api.devnet.solana.com)',
  paymasterUrl: "[https://kora.devnet.lazorkit.com](https://kora.devnet.lazorkit.com)", 
  configPaymaster: { 
    paymasterUrl: "[https://kora.devnet.lazorkit.com](https://kora.devnet.lazorkit.com)" 
  }
});
const sendMoney = async () => {
  await client.sendTransaction({
    to: "RECIPIENT_ADDRESS", // Replace with receiver address
    amount: 0.001,
    token: "SOL",
    mode: "gasless" // <--- This enables 0 fee for the user
  });
};
