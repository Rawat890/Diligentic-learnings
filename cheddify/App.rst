graph TD
    A[App Component Mount] --> B[Update API URL]
    B --> C[Initialize Facebook SDK]
    C --> D[Setup Deep Link Listeners]
    D --> E[Setup AppState Listener]
    E --> F[Check Network Connectivity]
    F --> G[Get User Data from Storage]
    
    G --> H{User Data Exists?}
    H -->|Yes| I[Set isLogin = true]
    H -->|No| J[Set isLogin = false]
    
    I --> K[App Ready]
    J --> K
    
    K --> L{Deep Link Received?}
    L -->|Yes| M[handleDubLink]
    L -->|No| N[Normal App Flow]
    
    M --> O[Call DUB_GET_URL_DETAILS API]
    O --> P[Extract Type & ID from URL]
    P --> Q[moveToDeepLinkScreen]
    
    Q --> R{Check Link Type}
    
    R -->|upsellOffer| S1[Navigate to Open Orders]
    R -->|wallet| S2[Navigate to Wallet Transactions]
    R -->|message| S3[Get User Details<br/>Navigate to Chat]
    R -->|order| S4[Navigate to Open Orders]
    R -->|post| S5{User Logged In?}
    R -->|referrer| S6[Navigate to Signup<br/>with Referrer ID]
    R -->|group| S7[Navigate to Group Details]
    R -->|recommend| S8{Is Own Profile?}
    R -->|profile| S9{Is Own Profile?}
    R -->|offer/requests| S10[Save Referral<br/>Navigate to Offer]
    R -->|eventQr| S11[Parse QR Data<br/>Navigate to Check-in]
    R -->|shareEventDashboard| S12[Parse Event Data<br/>Navigate to Guest List]
    R -->|inviteFriendToOrder| S13[Cache Link]
    R -->|guestReviewOffer| S14[Cache Link]
    
    S5 -->|Yes| S5A[Cache Link for Later]
    S5 -->|No| S5A
    
    S8 -->|Yes| S8A[Navigate to Own Profile]
    S8 -->|No| S8B[Navigate to Friend Profile<br/>Show Recommendations]
    
    S9 -->|Yes| S9A[Navigate to Own Profile]
    S9 -->|No| S9B[Navigate to Friend Profile<br/>Auto-follow]
    
    N --> T{Network Status Check}
    T -->|Connected| U[Render AppNavigator]
    T -->|Disconnected| V[Show No Internet Placeholder]
    
    V --> W[Close Socket Connections]
    
    X[AppState Changes to Background] --> Y[Clear Image Cache]
    Y --> Z[Trigger Garbage Collection]
    
    AA[Component Unmount] --> AB[Remove Deep Link Listener]
    AB --> AC[Remove AppState Listener]
    AC --> AD[Unsubscribe NetInfo]