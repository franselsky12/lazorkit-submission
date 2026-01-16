Sending Transactions

When sending a transaction, set feeMode to paymaster.

JavaScript

import { useWallet } from '@lazorkit/wallet-adapter-react';
import { SystemProgram, Transaction } from '@solana/web3.js';

const { sendTransaction } = useWallet();

const handleGaslessTransfer = async () => {
  const transaction = new Transaction().add(
    SystemProgram.transfer({
      fromPubkey: userPublicKey,
      toPubkey: recipientPublicKey,
      lamports: 1000000 // 0.001 SOL
    })
  );

  // The 'feeMode: paymaster' flag tells the SDK to use the relayer
  const signature = await sendTransaction(transaction, connection, {
    feeMode: 'paymaster' 
  });

  console.log("Gasless Tx Success:", signature);
};
