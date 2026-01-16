# Tutorial 1: Creating a Passkey-Based Smart Wallet

LazorKit allows users to create a non-custodial wallet using WebAuthn (FaceID/TouchID) without needing a seed phrase.

## Core Concept
Instead of generating a private key that the user must hide, LazorKit generates a **Passkey** on the user's device (Secure Enclave). This passkey signs a message to deploy a **Smart Account** (PDA) on the Solana blockchain.

## Step-by-Step Implementation

### 1. Wrap Your App with `LazorkitProvider`
In `src/main.jsx` or `src/App.jsx`, we wrap the application to provide wallet context.

```javascript
import { LazorkitProvider } from '@lazorkit/wallet-adapter-react';

const config = {
  rpcUrl: '[https://api.devnet.solana.com](https://api.devnet.solana.com)',
  portalUrl: '[https://portal.lazor.sh](https://portal.lazor.sh)', // Auth portal
  paymasterUrl: '[https://api.lazorkit.com/paymaster](https://api.lazorkit.com/paymaster)' // For gasless setup
};

<LazorkitProvider config={config}>
  <App />
</LazorkitProvider>
