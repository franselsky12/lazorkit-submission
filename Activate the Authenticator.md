Step 3: Activate the Authenticator
Invoke the browser's native WebAuthn API to trigger the biometric prompt and request user verification.

await navigator.credentials.create({ publicKey });

