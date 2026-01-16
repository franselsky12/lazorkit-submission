Step 2: Create the Challenge
The relying party (your application) must provide a challenge and user details to the authenticator.

const publicKey = {
    challenge: generatedChallenge,
    rp: { name: "LazorKit Wallet", id: window.location.hostname },
    user: { id: userId, name: username, displayName: username },
    pubKeyCredParams: [{ alg: -7, type: "public-key" }],
    authenticatorSelection: {
        authenticatorAttachment: "platform", // Forces FaceID/TouchID
        userVerification: "required"
    }
};
