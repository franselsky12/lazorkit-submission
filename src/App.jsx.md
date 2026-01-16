import React, { useState, useEffect, useRef } from 'react';
import { 
  Loader2, Fingerprint, ScanFace, Eye, Mic, Smartphone, Usb, Key, QrCode, Nfc, 
  CheckCircle, ArrowRight, ShieldCheck, LogOut, Globe, CreditCard, TrendingUp, Zap, 
  Laptop, Bluetooth, Watch, Gamepad2, HandMetal, X, ArrowUpRight, ArrowDownLeft, 
  Github, BookOpen, MessageCircle, Layers, Copy, Send, Wallet, Twitter, Upload, Camera,
  UserPlus, ChevronRight
} from 'lucide-react';

/**
 * LAZORKIT FINAL: REALISTIC GOOGLE LOGIN FLOW
 * 1. Google Account Chooser Step added.
 * 2. WebAuthn Real Integration (Platform & Cross-Platform).
 * 3. Custom Avatar Upload.
 * 4. Full UI Polish.
 */

// --- KONFIGURASI GAMBAR ---
const IMG_LAZORKIT = "/lazorkit-logo.jpg";
const IMG_SOLANA_BANNER = "/Powered by Solana.jpg"; 
const IMG_GOOGLE_G = "https://upload.wikimedia.org/wikipedia/commons/c/c1/Google_%22G%22_logo.svg";

// --- TEMA WARNA ---
const THEME = {
    primary: '#6D28D9',
    secondary: '#EC4899',
    bg: '#050505',
    cardBg: '#121212',
    textMuted: '#a1a1aa',
    success: '#10b981'
};

const LAZOR_LINKS = {
    docs: 'https://docs.lazorkit.com/',
    github: 'https://github.com/lazor-kit/lazor-kit',
    telegram: 'https://t.me/lazorkit',
    twitter: 'https://x.com/lazorkit'
};

// --- DUMMY ACCOUNTS (Simulasi Akun Tersimpan) ---
const SAVED_ACCOUNTS = [
    { name: "Faris", email: "faris@gmail.com", avatar: "https://api.dicebear.com/7.x/avataaars/svg?seed=Faris" },
    { name: "Lazor Developer", email: "dev@lazorkit.com", avatar: "https://api.dicebear.com/7.x/avataaars/svg?seed=Dev" }
];

// --- SECURITY METHODS ---
const SECURITY_METHODS = [
    { id: 'face', label: 'Face ID', icon: ScanFace, color: '#00f2ff', type: 'platform' },
    { id: 'finger', label: 'Fingerprint', icon: Fingerprint, color: '#EC4899', type: 'platform' },
    { id: 'device', label: 'This Device', icon: Laptop, color: '#ffffff', type: 'platform' },
    { id: 'windows', label: 'Win Hello', icon: ShieldCheck, color: '#0ea5e9', type: 'platform' },
    
    { id: 'phone', label: 'Phone Link', icon: Smartphone, color: '#22c55e', type: 'cross-platform' },
    { id: 'usb', label: 'USB Key', icon: Usb, color: '#ff5555', type: 'cross-platform' },
    { id: 'qr', label: 'QR Auth', icon: QrCode, color: '#aaaaaa', type: 'cross-platform' },
    { id: 'passkey', label: 'Passkey', icon: Key, color: '#d946ef', type: 'cross-platform' },

    { id: 'iris', label: 'Iris Scan', icon: Eye, color: '#a855f7', type: 'sim' },
    { id: 'voice', label: 'Voice', icon: Mic, color: '#ffd700', type: 'sim' },
    { id: 'bluetooth', label: 'Bluetooth', icon: Bluetooth, color: '#3b82f6', type: 'sim' },
    { id: 'watch', label: 'Watch Link', icon: Watch, color: '#f97316', type: 'sim' },
    { id: 'nfc', label: 'NFC Tag', icon: Nfc, color: '#14b8a6', type: 'sim' },
    { id: 'palm', label: 'Palm Vein', icon: HandMetal, color: '#ef4444', type: 'sim' },
    { id: 'controller', label: 'Gamepad', icon: Gamepad2, color: '#6366f1', type: 'sim' },
];

const AVATARS = Array.from({ length: 36 }, (_, i) => 
    `https://api.dicebear.com/7.x/avataaars/svg?seed=LazorUltra${i}&backgroundColor=b6e3f4,c0aede,d1d4f9`
);

const DUMMY_WALLET = "8xG9...kL2m5ZqR";

// --- WEBAUTHN FUNCTION ---
const registerPasskey = async (username, attachmentType) => {
    if (!window.PublicKeyCredential) {
        alert("WebAuthn not supported.");
        return false;
    }
    try {
        const challenge = new Uint8Array(32);
        window.crypto.getRandomValues(challenge);
        const userId = Uint8Array.from(username, c => c.charCodeAt(0));

        const publicKey = {
            challenge: challenge,
            rp: { name: "LazorKit Wallet", id: window.location.hostname },
            user: { id: userId, name: username, displayName: username },
            pubKeyCredParams: [{ alg: -7, type: "public-key" }, { alg: -257, type: "public-key" }],
            timeout: 60000,
            attestation: "direct",
            authenticatorSelection: {
                authenticatorAttachment: attachmentType, 
                userVerification: "required"
            }
        };

        await navigator.credentials.create({ publicKey });
        return true;
    } catch (err) {
        console.log("Auth cancelled or failed:", err);
        return false;
    }
};

// --- MODAL COMPONENT ---
const Modal = ({ children }) => (
    <div className="modal-overlay">
        <div className="modal-box animate-pop">
            {children}
        </div>
    </div>
);

const LazorkitApp = () => {
  // Navigation: landing -> google_chooser -> (google_input) -> loading -> security_hub -> ...
  const [view, setView] = useState('landing'); 
  const [user, setUser] = useState(null);
  const [tempName, setTempName] = useState('');
  const [activeMethod, setActiveMethod] = useState(null);
  const [securityStep, setSecurityStep] = useState('idle'); 
  const [showProfileEdit, setShowProfileEdit] = useState(false);
  const [dashView, setDashView] = useState('overview'); 

  // Transaction States
  const [showSend, setShowSend] = useState(false);
  const [showReceive, setShowReceive] = useState(false);
  const [recipient, setRecipient] = useState('');
  const [amount, setAmount] = useState('');
  const [isSending, setIsSending] = useState(false);
  const [sendSuccess, setSendSuccess] = useState(false);
  const [copied, setCopied] = useState(false);

  // Upload Ref
  const fileInputRef = useRef(null);

  // Favicon Effect
  useEffect(() => {
    const setFavicon = () => {
        const link = document.querySelector("link[rel~='icon']") || document.createElement('link');
        link.type = 'image/jpeg';
        link.rel = 'icon';
        link.href = IMG_LAZORKIT;
        document.getElementsByTagName('head')[0].appendChild(link);
    };
    setFavicon();
  }, []);

  // --- LOGIN LOGIC ---
  
  // 1. Start Login -> Show Chooser
  const handleStartLogin = () => setView('google_chooser');

  // 2a. Select Existing Account
  const handleSelectAccount = (account) => {
      setView('loading');
      setTimeout(() => {
          setUser(account);
          setView('security_hub');
      }, 1500);
  };

  // 2b. Select "Use Another" -> Show Input
  const handleUseAnother = () => setView('google_input');

  // 2c. Process Manual Input
  const processManualLogin = () => {
    if (!tempName) return;
    setView('loading');
    setTimeout(() => {
        setUser({
            name: tempName,
            email: `${tempName.toLowerCase().replace(/\s/g, '')}@gmail.com`,
            avatar: AVATARS[0]
        });
        setView('security_hub');
    }, 1500);
  };

  const handleImageUpload = (event) => {
      const file = event.target.files[0];
      if (file) {
          const reader = new FileReader();
          reader.onloadend = () => {
              setUser({ ...user, avatar: reader.result });
              setShowProfileEdit(false);
          };
          reader.readAsDataURL(file);
      }
  };

  const handleMethodSelect = async (method) => {
    setActiveMethod(method);
    if (method.type === 'platform' || method.type === 'cross-platform') {
        setSecurityStep('permission');
        setTimeout(async () => {
            const success = await registerPasskey(user.name, method.type);
            if (success) {
                setSecurityStep('scanning');
                setTimeout(() => {
                    setSecurityStep('success');
                    setTimeout(() => {
                        setView('dashboard');
                        setSecurityStep('idle');
                        setDashView('overview');
                    }, 1000);
                }, 1000);
            } else {
                setActiveMethod(null);
                setSecurityStep('idle');
            }
        }, 500);
    } else {
        setSecurityStep('permission'); 
        setTimeout(() => grantPermission(), 1500);
    }
  };

  const grantPermission = () => {
    setSecurityStep('scanning');
    setTimeout(() => {
        setSecurityStep('success');
        setTimeout(() => {
            setView('dashboard');
            setSecurityStep('idle');
            setDashView('overview');
        }, 1500);
    }, 2000);
  };

  const executeSend = () => {
      if(!recipient || !amount) return;
      setIsSending(true);
      setTimeout(() => {
          setIsSending(false);
          setSendSuccess(true);
          setTimeout(() => {
              setSendSuccess(false);
              setShowSend(false);
              setRecipient('');
              setAmount('');
          }, 2000);
      }, 2500);
  };

  const executeCopy = () => {
      setCopied(true);
      setTimeout(() => setCopied(false), 2000);
  };

  const openLink = (url) => window.open(url, '_blank');

  return (
    <>
      <style>{`
        @import url('https://fonts.googleapis.com/css2?family=Plus+Jakarta+Sans:wght@300;400;500;600;700;800&display=swap');
        * { box-sizing: border-box; }
        body { margin: 0; background: ${THEME.bg}; color: white; font-family: 'Plus Jakarta Sans', sans-serif; overflow-x: hidden; }
        .w-full { width: 100%; }
        .flex-between { display: flex; justify-content: space-between; align-items: center; }
        .text-muted { color: ${THEME.textMuted}; }
        .font-bold { fontWeight: 700; }
        .input-field { width: 100%; padding: 16px; border-radius: 12px; border: 1px solid #333; background: #000; color: white; fontSize: 16px; outline: none; transition: 0.2s; }
        .input-field:focus { border-color: ${THEME.primary}; }
        .btn-primary { background: linear-gradient(90deg, ${THEME.primary}, ${THEME.secondary}); color: white; padding: 14px 32px; border-radius: 50px; border: none; font-size: 16px; font-weight: 700; cursor: pointer; transition: 0.2s; box-shadow: 0 10px 30px rgba(109, 40, 217, 0.4); display: flex; align-items: center; justify-content: center; gap: 8px; }
        .btn-primary:hover { transform: scale(1.05); }
        .btn-outline { background: transparent; border: 1px solid rgba(255,255,255,0.2); color: white; padding: 10px 24px; border-radius: 50px; cursor: pointer; font-weight: 600; transition:0.2s; }
        .btn-outline:hover { background: rgba(255,255,255,0.1); border-color: white; }
        
        /* Account Chooser Styles */
        .account-item { display: flex; alignItems: center; gap: 15px; padding: 15px; border-bottom: 1px solid #333; cursor: pointer; transition: 0.2s; text-align: left; }
        .account-item:hover { background: rgba(255,255,255,0.05); }
        .account-item:last-child { border-bottom: none; }
        
        .landing-wrap { min-height: 100vh; background: radial-gradient(circle at top, #1e1b4b 0%, #000 60%); padding: 20px 20px 60px; }
        .hero-title { font-size: 64px; font-weight: 800; line-height: 1.1; margin: 20px 0; letter-spacing: -2px; text-align: center; }
        .text-gradient { background: linear-gradient(135deg, ${THEME.primary}, ${THEME.secondary}); -webkit-background-clip: text; -webkit-text-fill-color: transparent; }
        .link-grid { display: grid; grid-template-columns: repeat(auto-fit, minmax(200px, 1fr)); gap: 20px; width: 100%; max-width: 900px; margin: 40px auto 0; }
        .link-card { background: rgba(255,255,255,0.03); border: 1px solid rgba(255,255,255,0.1); padding: 25px; border-radius: 20px; cursor: pointer; transition: 0.3s; display: flex; flex-direction: column; align-items: center; gap: 15px; }
        .link-card:hover { background: rgba(109, 40, 217, 0.2); border-color: ${THEME.primary}; transform: translateY(-5px); }
        .link-icon { width: 50px; height: 50px; background: rgba(109, 40, 217, 0.2); border-radius: 12px; display: flex; align-items: center; justify-content: center; color: ${THEME.primary}; }
        
        .modal-overlay { position: fixed; top: 0; left: 0; width: 100%; height: 100%; background: rgba(0,0,0,0.85); backdrop-filter: blur(15px); z-index: 999; display: flex; align-items: center; justify-content: center; padding: 20px; }
        .modal-box { background: #121212; border: 1px solid #333; border-radius: 24px; padding: 40px; width: 100%; max-width: 450px; text-align: center; position: relative; box-shadow: 0 50px 100px rgba(0,0,0,0.9); }
        .animate-pop { animation: popIn 0.3s cubic-bezier(0.175, 0.885, 0.32, 1.275); }
        @keyframes popIn { from { transform: scale(0.9); opacity: 0; } to { transform: scale(1); opacity: 1; } }
        
        .sec-grid { display: grid; grid-template-columns: repeat(3, 1fr); gap: 10px; margin-top: 20px; max-height: 350px; overflow-y: auto; padding-right: 5px; }
        .sec-item { background: #1a1a1a; padding: 15px 5px; border-radius: 12px; cursor: pointer; border: 1px solid transparent; transition: 0.2s; display: flex; flex-direction: column; align-items: center; gap: 8px; }
        .sec-item:hover { background: #222; border-color: ${THEME.primary}; transform: translateY(-3px); }
        
        .app-shell { display: flex; flex-direction: column; min-height: 100vh; background: #050505; }
        .top-bar { height: 70px; border-bottom: 1px solid #222; display: flex; align-items: center; justify-content: space-between; padding: 0 30px; background: rgba(5,5,5,0.8); backdrop-filter: blur(10px); position: sticky; top: 0; z-index: 50; }
        .main-content { width: 100%; max-width: 1200px; margin: 0 auto; padding: 40px 20px; flex: 1; }
        .dashboard-grid { display: grid; grid-template-columns: 2fr 1fr; gap: 30px; }
        @media (max-width: 900px) { .dashboard-grid { grid-template-columns: 1fr; } }
        .card { background: ${THEME.cardBg}; border: 1px solid #222; border-radius: 20px; padding: 25px; position: relative; overflow: hidden; }
        .balance-card { background: linear-gradient(135deg, ${THEME.primary}, ${THEME.secondary}); border: none; display: flex; flex-direction: column; justify-content: center; min-height: 200px; }
        .tx-item { display: flex; align-items: center; justify-content: space-between; padding: 15px 0; border-bottom: 1px solid #222; }
        .menu-icon-box { background:#1a1a1a; padding: 15px; border-radius: 12px; text-align: center; cursor: pointer; border: 1px solid transparent; transition: 0.2s; }
        .menu-icon-box:hover, .menu-icon-box.active { background: #222; border-color: ${THEME.primary}; transform: translateY(-2px); }
        .scan-bar { width: 100%; height: 2px; background: ${THEME.primary}; animation: scan 1.5s infinite ease-in-out; }
        @keyframes scan { 0%{transform:translateY(-50px)} 100%{transform:translateY(50px)} }
      `}</style>

      {/* ================= VIEW: LANDING PAGE ================= */}
      {view === 'landing' && (
        <div className="landing-wrap">
            <div style={{display:'flex', justifyContent:'space-between', alignItems:'center', maxWidth:'1100px', margin:'0 auto'}}>
                <div style={{display:'flex', alignItems:'center', gap:'8px', fontSize:'24px', fontWeight:'800', letterSpacing:'-0.5px'}}>
                    <img src={IMG_LAZORKIT} style={{height:'32px', width:'auto', borderRadius:'8px'}} alt="L" />
                    LazorKit
                </div>
                <button className="btn-outline" onClick={handleStartLogin}>Sign In</button>
            </div>

            <div style={{maxWidth:'1100px', margin:'60px auto', textAlign:'center', display:'flex', flexDirection:'column', alignItems:'center'}}>
                <div style={{background:'rgba(255,255,255,0.1)', padding:'8px 16px', borderRadius:'30px', fontSize:'14px', marginBottom:'20px', display:'inline-flex', alignItems:'center', gap:'8px'}}>
                    <Zap size={16} fill="currentColor"/> The Ultimate Solana UX
                </div>
                <h1 className="hero-title">Unlock the <span className="text-gradient">Future</span><br/>of Biometric Wallets</h1>
                <p style={{fontSize:'18px', color: THEME.textMuted, maxWidth:'600px', marginBottom:'40px', lineHeight:'1.6'}}>
                    No seed phrases. No extensions. Access your Solana assets securely using FaceID, Fingerprint, or your Device itself.
                </p>
                <button className="btn-primary" onClick={handleStartLogin}>Launch App <ArrowRight size={20}/></button>

                <div style={{display:'flex', gap:'40px', marginTop:'80px', flexWrap:'wrap', justifyContent:'center', borderBottom:'1px solid rgba(255,255,255,0.1)', paddingBottom:'60px', width:'100%'}}>
                    <div className="text-center"><div style={{background:'#1a1a1a', width:60, height:60, borderRadius:'16px', display:'flex', alignItems:'center', justifyContent:'center', margin:'0 auto 15px'}}><Fingerprint color="#ec4899"/></div><h3>Biometric</h3></div>
                    <div className="text-center"><div style={{background:'#1a1a1a', width:60, height:60, borderRadius:'16px', display:'flex', alignItems:'center', justifyContent:'center', margin:'0 auto 15px'}}><Zap color="#eab308"/></div><h3>Gasless</h3></div>
                    <div className="text-center"><div style={{background:'#1a1a1a', width:60, height:60, borderRadius:'16px', display:'flex', alignItems:'center', justifyContent:'center', margin:'0 auto 15px'}}><Layers color="#3b82f6"/></div><h3>Smart Account</h3></div>
                </div>

                <div style={{marginTop:'60px', width:'100%'}}>
                    <h2 className="text-center" style={{fontSize:'32px'}}>Build & Resources</h2>
                    <div className="link-grid">
                        <div className="link-card" onClick={()=>openLink(LAZOR_LINKS.docs)}><div className="link-icon"><BookOpen size={28}/></div><div className="font-bold text-lg">Documentation</div></div>
                        <div className="link-card" onClick={()=>openLink(LAZOR_LINKS.github)}><div className="link-icon"><Github size={28}/></div><div className="font-bold text-lg">GitHub Repo</div></div>
                        <div className="link-card" onClick={()=>openLink(LAZOR_LINKS.telegram)}><div className="link-icon"><MessageCircle size={28}/></div><div className="font-bold text-lg">Community</div></div>
                        <div className="link-card" onClick={()=>openLink(LAZOR_LINKS.twitter)}><div className="link-icon"><Twitter size={28}/></div><div className="font-bold text-lg">X (Twitter)</div></div>
                    </div>
                </div>
                
                <div style={{marginTop:'80px', paddingTop:'30px', borderTop:'1px solid rgba(255,255,255,0.1)', width:'100%', display:'flex', flexDirection:'column', alignItems:'center', gap:'10px'}}>
                    <span style={{color: THEME.textMuted, fontSize:'14px', fontWeight:'500', letterSpacing:'0.5px'}}>Powered by Solana</span>
                    <img src={IMG_SOLANA_BANNER} alt="Solana Banner" style={{height:'50px', objectFit:'contain', opacity: 0.9}} />
                </div>
            </div>
        </div>
      )}

      {/* ================= MODALS ================= */}
      
      {/* 1. GOOGLE ACCOUNT CHOOSER (NEW FEATURE) */}
      {view === 'google_chooser' && (
        <Modal>
            <div style={{textAlign:'center'}}>
                <img src={IMG_GOOGLE_G} width="48" style={{marginBottom:'20px'}}/>
                <h3 style={{fontSize:'24px', margin:'0 0 10px'}}>Choose an account</h3>
                <p className="text-muted" style={{marginBottom:'30px'}}>to continue to LazorKit</p>
            </div>
            
            <div style={{border:'1px solid #333', borderRadius:'12px', overflow:'hidden'}}>
                {/* Simulated Saved Accounts */}
                {SAVED_ACCOUNTS.map((acc, idx) => (
                    <div key={idx} className="account-item" onClick={() => handleSelectAccount(acc)}>
                        <img src={acc.avatar} width="36" style={{borderRadius:'50%'}}/>
                        <div>
                            <div style={{fontWeight:'600', fontSize:'14px'}}>{acc.name}</div>
                            <div style={{fontSize:'12px', color:'#888'}}>{acc.email}</div>
                        </div>
                    </div>
                ))}
                
                {/* Use Another Account Option */}
                <div className="account-item" onClick={handleUseAnother}>
                    <div style={{width:'36px', height:'36px', borderRadius:'50%', background:'#333', display:'flex', alignItems:'center', justifyContent:'center'}}>
                        <UserPlus size={18} color="#ccc"/>
                    </div>
                    <div style={{fontWeight:'600', fontSize:'14px'}}>Use another account</div>
                </div>
            </div>
            <button className="btn-outline w-full" style={{marginTop:'30px', border:'none'}} onClick={()=>setView('landing')}>Cancel</button>
        </Modal>
      )}

      {/* 2. MANUAL INPUT (If "Use another account" is selected) */}
      {view === 'google_input' && (
        <Modal>
            <img src={IMG_GOOGLE_G} width="48" style={{marginBottom:'20px'}}/>
            <h3 style={{fontSize:'24px', margin:'0 0 10px'}}>Sign in</h3>
            <p className="text-muted" style={{marginBottom:'20px'}}>Use your Google Account</p>
            <input autoFocus placeholder="Email or phone" value={tempName} onChange={(e)=>setTempName(e.target.value)} className="input-field" style={{marginBottom:'20px'}}/>
            <div style={{display:'flex', justifyContent:'space-between', alignItems:'center'}}>
                <button className="btn-outline" style={{border:'none', padding:0, color:THEME.primary}} onClick={()=>setView('google_chooser')}>Back</button>
                <button className="btn-primary" onClick={processManualLogin}>Next</button>
            </div>
        </Modal>
      )}

      {view === 'loading' && (
        <Modal>
            <Loader2 size={50} color={THEME.primary} className="animate-spin" style={{margin:'0 auto 20px'}}/>
            <h3>Authenticating...</h3>
        </Modal>
      )}

      {view === 'security_hub' && user && (
        <Modal>
            <div style={{display:'flex', alignItems:'center', gap:'15px', marginBottom:'20px', background:'#222', padding:'10px', borderRadius:'50px'}}>
                <img src={user.avatar} width="40" style={{borderRadius:'50%', height:'40px', objectFit:'cover'}}/>
                <div style={{textAlign:'left'}}>
                    <div className="font-bold">{user.name}</div>
                    <div style={{fontSize:'12px', color:'#888'}}>Select Method</div>
                </div>
            </div>
            <div className="sec-grid" style={{maxHeight:'300px', overflowY:'auto', paddingRight:'5px'}}>
                {SECURITY_METHODS.map(m => (
                    <div key={m.id} className="sec-item" onClick={()=>handleMethodSelect(m)}>
                        <m.icon size={24} color={m.color}/>
                        <span style={{fontSize:'10px', color:'#ccc'}}>{m.label}</span>
                    </div>
                ))}
            </div>
        </Modal>
      )}

      {view === 'security_process' && activeMethod && (
        <Modal>
            {securityStep === 'permission' ? (
                <div style={{textAlign:'left'}}>
                    <div style={{display:'flex', alignItems:'center', gap:'10px', marginBottom:'20px', color: THEME.textMuted}}>
                        <ShieldCheck size={18}/> {activeMethod.type.includes('cross') ? 'Cross-Device Security' : 'Windows Security'}
                    </div>
                    <h3 style={{fontSize:'20px', marginBottom:'10px'}}>Use {activeMethod.label}?</h3>
                    <p className="text-muted" style={{marginBottom:'30px', lineHeight:'1.5'}}>
                        {activeMethod.type === 'cross-platform' 
                            ? "Scan the QR code or insert your security key when prompted by the browser."
                            : "LazorKit wants to verify your identity using built-in security."}
                    </p>
                    <div style={{display:'flex', gap:'10px'}}>
                        <button className="btn-outline w-full" onClick={()=>setView('security_hub')}>Deny</button>
                        <button className="btn-primary w-full" style={{opacity:0.7, cursor:'default'}}>Waiting Input...</button>
                    </div>
                </div>
            ) : securityStep === 'scanning' ? (
                <div>
                    <div style={{width:80, height:80, border:`2px solid ${activeMethod.color}`, borderRadius:'16px', margin:'0 auto 20px', overflow:'hidden', display:'flex', alignItems:'center', justifyContent:'center'}}>
                        <activeMethod.icon size={40} color={activeMethod.color}/>
                        <div style={{position:'absolute', width:80}} className="scan-bar"></div>
                    </div>
                    <h3>Verifying...</h3>
                </div>
            ) : (
                <div>
                    <CheckCircle size={80} color={THEME.success} style={{margin:'0 auto 20px'}}/>
                    <h3>Access Granted</h3>
                </div>
            )}
        </Modal>
      )}

      {/* MODAL EDIT PROFILE DENGAN UPLOAD */}
      {showProfileEdit && (
        <Modal>
            <div className="flex-between" style={{marginBottom:'20px'}}>
                <h3>Select Avatar</h3>
                <X style={{cursor:'pointer'}} onClick={()=>setShowProfileEdit(false)}/>
            </div>
            
            {/* Tombol Upload */}
            <div style={{marginBottom:'20px'}}>
                <input type="file" accept="image/*" ref={fileInputRef} style={{display:'none'}} onChange={handleImageUpload} />
                <button className="btn-outline w-full" onClick={()=>fileInputRef.current.click()} style={{display:'flex', alignItems:'center', justifyContent:'center', gap:'10px'}}>
                    <Camera size={18}/> Upload Your Own Photo
                </button>
            </div>

            <div style={{display:'grid', gridTemplateColumns:'repeat(6, 1fr)', gap:'10px', maxHeight:'300px', overflowY:'auto', padding:'10px'}}>
                {AVATARS.map(url => (
                    <img key={url} src={url} style={{width:'100%', borderRadius:'50%', cursor:'pointer', border:'2px solid transparent', transition:'0.2s'}} 
                         onClick={()=>{setUser({...user, avatar: url}); setShowProfileEdit(false)}}
                         onMouseOver={(e)=>e.currentTarget.style.cssText='border-color:'+THEME.primary+'; transform:scale(1.1)'}
                         onMouseOut={(e)=>e.currentTarget.style.cssText='border-color:transparent; transform:scale(1)'}
                    />
                ))}
            </div>
        </Modal>
      )}

      {showSend && (
        <Modal>
            <div className="flex-between" style={{marginBottom:'20px'}}>
                <h3 style={{margin:0}}>Send SOL</h3>
                <X style={{cursor:'pointer'}} onClick={()=>{setShowSend(false); setSendSuccess(false);}}/>
            </div>
            {!sendSuccess ? (
                <>
                    <div style={{textAlign:'left', marginBottom:'15px'}}>
                        <label style={{fontSize:'12px', color:THEME.textMuted, display:'block', marginBottom:'8px'}}>Recipient Address</label>
                        <input placeholder="Enter Solana address" value={recipient} onChange={e=>setRecipient(e.target.value)} className="input-field"/>
                    </div>
                    <div style={{textAlign:'left', marginBottom:'30px'}}>
                        <label style={{fontSize:'12px', color:THEME.textMuted, display:'block', marginBottom:'8px'}}>Amount (SOL)</label>
                        <input placeholder="0.00" type="number" value={amount} onChange={e=>setAmount(e.target.value)} className="input-field"/>
                    </div>
                    <button className="btn-primary w-full" onClick={executeSend} disabled={isSending || !recipient || !amount}>
                        {isSending ? <Loader2 className="animate-spin"/> : <Send size={18}/>} 
                        {isSending ? 'Sending...' : 'Confirm Send'}
                    </button>
                </>
            ) : (
                <div style={{textAlign:'center', padding:'20px 0'}}>
                    <CheckCircle size={60} color={THEME.success} style={{margin:'0 auto 20px'}}/>
                    <h3 style={{color:THEME.success}}>Transaction Sent!</h3>
                </div>
            )}
        </Modal>
      )}

      {showReceive && (
        <Modal>
            <div className="flex-between" style={{marginBottom:'20px'}}>
                <h3 style={{margin:0}}>Receive SOL</h3>
                <X style={{cursor:'pointer'}} onClick={()=>setShowReceive(false)}/>
            </div>
            <div style={{background:'white', padding:'20px', borderRadius:'20px', display:'inline-block', marginBottom:'30px'}}>
                <QrCode size={120} color="black"/>
            </div>
            <div style={{background:'#222', padding:'15px', borderRadius:'12px', display:'flex', alignItems:'center', justifyContent:'space-between', gap:'10px'}}>
                <div style={{fontFamily:'monospace', fontSize:'14px', overflow:'hidden', textOverflow:'ellipsis'}}>{DUMMY_WALLET}</div>
                <button style={{background: copied ? THEME.success : '#333', border:'none', color:'white', padding:'8px', borderRadius:'8px', cursor:'pointer'}} onClick={executeCopy}>
                    {copied ? <CheckCircle size={18}/> : <Copy size={18}/>}
                </button>
            </div>
        </Modal>
      )}

      {view === 'dashboard' && user && (
        <div className="app-shell">
            <div className="top-bar">
                <div style={{display:'flex', alignItems:'center', gap:'8px'}}>
                    <img src={IMG_LAZORKIT} style={{height:'28px', width:'auto', borderRadius:'6px'}} alt="L" />
                    <span style={{fontWeight:'800', fontSize:'20px', letterSpacing:'0px'}}>LazorKit</span>
                    <div style={{background:'rgba(109, 40, 217, 0.2)', color:THEME.primary, padding:'4px 8px', borderRadius:'6px', fontSize:'10px', fontWeight:'bold', border:`1px solid ${THEME.primary}`, marginLeft:'10px'}}>Devnet</div>
                </div>
                <div style={{display:'flex', alignItems:'center', gap:'20px'}}>
                    <div style={{display:'flex', alignItems:'center', gap:'10px', cursor:'pointer'}} onClick={()=>setShowProfileEdit(true)}>
                        <img src={user.avatar} width="36" style={{borderRadius:'50%', height:'36px', objectFit:'cover', border:`2px solid ${THEME.primary}`}}/>
                    </div>
                    <button style={{background:'#222', border:'none', color:'white', padding:'8px', borderRadius:'8px', cursor:'pointer'}} onClick={()=>setView('landing')}><LogOut size={18}/></button>
                </div>
            </div>

            <div className="main-content">
                <div className="dashboard-grid">
                    <div style={{display:'flex', flexDirection:'column', gap:'20px'}}>
                        {dashView === 'overview' && (
                            <>
                                <div className="card balance-card">
                                    <div style={{display:'flex', justifyContent:'space-between', alignItems:'flex-start', marginBottom:'20px'}}>
                                        <div>
                                            <p style={{margin:0, opacity:0.8, fontSize:'14px'}}>Total Balance</p>
                                            <h2 style={{fontSize:'48px', fontWeight:'800', margin:'5px 0'}}>$84,500.25</h2>
                                            <div style={{fontSize:'14px', opacity:0.8}}>≈ 450.5 SOL</div>
                                        </div>
                                    </div>
                                    <div style={{display:'flex', gap:'15px'}}>
                                        <button className="btn-primary" onClick={()=>setShowSend(true)}><ArrowUpRight size={18}/> Send</button>
                                        <button className="btn-outline" style={{border:'none', background:'rgba(255,255,255,0.2)', display:'flex', alignItems:'center', gap:'8px'}} onClick={()=>setShowReceive(true)}><ArrowDownLeft size={18}/> Receive</button>
                                    </div>
                                </div>
                                <div className="card">
                                    <h3 style={{margin:'0 0 20px', fontSize:'18px'}}>Recent History</h3>
                                    <div>
                                        <div className="tx-item"><div style={{display:'flex', alignItems:'center'}}><div style={{width:40, height:40, borderRadius:'50%', background:'#222', display:'flex', alignItems:'center', justifyContent:'center', marginRight:15}}><ArrowDownLeft size={18} color="#10b981"/></div><div><div className="font-bold">Received SOL</div><div style={{fontSize:'12px', color:'#888'}}>From: Coinbase</div></div></div><div style={{fontWeight:'bold', color:'#10b981'}}>+ 5.2 SOL</div></div>
                                        <div className="tx-item" style={{border:0}}><div style={{display:'flex', alignItems:'center'}}><div style={{width:40, height:40, borderRadius:'50%', background:'#222', display:'flex', alignItems:'center', justifyContent:'center', marginRight:15}}><ArrowUpRight size={18} color={THEME.secondary}/></div><div><div className="font-bold">Minted NFT</div><div style={{fontSize:'12px', color:'#888'}}>Gasless Transaction</div></div></div><div className="font-bold">- 0.0 SOL</div></div>
                                    </div>
                                </div>
                            </>
                        )}
                        {dashView !== 'overview' && (
                            <div className="card" style={{minHeight:'400px', display:'flex', alignItems:'center', justifyContent:'center', flexDirection:'column', textAlign:'center'}}>
                                <h2 style={{fontSize:'28px', margin:'0 0 10px'}}>{dashView.charAt(0).toUpperCase() + dashView.slice(1)}</h2>
                                <p className="text-muted">Coming Soon.</p>
                                <button className="btn-outline" style={{marginTop:'20px'}} onClick={()=>setDashView('overview')}>Back to Overview</button>
                            </div>
                        )}
                    </div>

                    <div style={{display:'flex', flexDirection:'column', gap:'20px'}}>
                        <div className="card">
                            <h3 style={{margin:'0 0 20px', fontSize:'16px'}}>Menu</h3>
                            <div style={{display:'grid', gridTemplateColumns:'1fr 1fr', gap:'10px'}}>
                                <div className={`menu-icon-box ${dashView==='browser'?'active':''}`} onClick={()=>setDashView('browser')}><Globe size={24} color={dashView==='browser'?THEME.primary:"#00f2ff"} style={{margin:'0 auto 10px'}}/><div style={{fontSize:'12px'}}>Browser</div></div>
                                <div className={`menu-icon-box ${dashView==='cards'?'active':''}`} onClick={()=>setDashView('cards')}><CreditCard size={24} color={dashView==='cards'?THEME.primary:THEME.secondary} style={{margin:'0 auto 10px'}}/><div style={{fontSize:'12px'}}>Cards</div></div>
                                <div className={`menu-icon-box ${dashView==='staking'?'active':''}`} onClick={()=>setDashView('staking')}><TrendingUp size={24} color={dashView==='staking'?THEME.primary:"#10b981"} style={{margin:'0 auto 10px'}}/><div style={{fontSize:'12px'}}>Staking</div></div>
                                <div className={`menu-icon-box ${dashView==='audit'?'active':''}`} onClick={()=>setDashView('audit')}><ShieldCheck size={24} color={dashView==='audit'?THEME.primary:"#f59e0b"} style={{margin:'0 auto 10px'}}/><div style={{fontSize:'12px'}}>Audit</div></div>
                            </div>
                        </div>
                        <div style={{background: 'rgba(16, 185, 129, 0.1)', padding:'15px', borderRadius:'16px', display:'flex', alignItems:'center', gap:'15px', border:'1px solid rgba(16, 185, 129, 0.2)'}}>
                            <ShieldCheck size={20} color="#10b981"/>
                            <div><div style={{fontSize:'12px', fontWeight:'bold', color:'#10b981'}}>System Secure</div><div style={{fontSize:'10px', color:'#888'}}>Connected via {activeMethod.label}</div></div>
                        </div>
                    </div>
                </div>
            </div>
        </div>
      )}
    </>
  );
};

export default LazorkitApp;
