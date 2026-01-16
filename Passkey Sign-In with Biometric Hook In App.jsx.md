import { useWallet } from '@lazorkit/wallet-adapter-react';

const { connect, connected } = useWallet();

const handleLogin = async () => {
  // This triggers the browser's native FaceID/TouchID dialog
  await connect({
    loginMethod: 'passkey'
  });
};
