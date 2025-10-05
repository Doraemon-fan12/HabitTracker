# 🎨 Custom Avatar Upload - Visual Flow Diagram

## Overall Architecture
```
┌─────────────────────────────────────────────────────────────────┐
│                         USER INTERFACE                          │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │         EnhancedAvatarPickerDialog                      │   │
│  │  ┌──────────┬──────────┬──────────┐                    │   │
│  │  │ Avatar 1 │ Avatar 2 │ Avatar 3 │   [Upload Icon]    │   │
│  │  ├──────────┼──────────┼──────────┤                    │   │
│  │  │ Avatar 4 │ Avatar 5 │ Avatar 6 │                    │   │
│  │  ├──────────┼──────────┼──────────┤                    │   │
│  │  │ Avatar 7 │ Avatar 8 │  My⭐   │                    │   │
│  │  └──────────┴──────────┴──────────┘                    │   │
│  └─────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────────┐
│                      VIEW MODEL LAYER                           │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │           AvatarPickerViewModel                         │   │
│  │  • loadAvatars()                                        │   │
│  │  • uploadCustomAvatar(uri)                             │   │
│  │  • deleteCustomAvatar(url)                             │   │
│  │  • UiState (loading, uploading, errors)                │   │
│  └─────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────────┐
│                    BUSINESS LOGIC LAYER                         │
│  ┌─────────────────────────────────────────────────────────┐   │
│  │              AvatarManager                              │   │
│  │  • getDefaultAvatars()                                  │   │
│  │  • uploadCustomAvatar(uri) → Result                    │   │
│  │  • deleteCustomAvatar(url) → Boolean                   │   │
│  │  • getAllAvatars() → List<AvatarItem>                  │   │
│  │  • resetAvatar()                                        │   │
│  └─────────────────────────────────────────────────────────┘   │
│                            │                                    │
│            ┌───────────────┴───────────────┐                   │
│            ▼                               ▼                   │
│  ┌─────────────────────┐      ┌─────────────────────────┐     │
│  │ GitHubAvatarUploader│      │   AvatarConfig          │     │
│  │ • uploadAvatar()    │      │ • initialize(token)     │     │
│  │ • deleteAvatar()    │      │ • isUploadEnabled()     │     │
│  │ • listUserAvatars() │      │ • autoInitialize()      │     │
│  └─────────────────────┘      └─────────────────────────┘     │
└─────────────────────────────────────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────────┐
│                       DATA LAYER                                │
│                                                                 │
│  ┌──────────────────┐              ┌──────────────────────┐    │
│  │  GitHub API      │              │   Firestore          │    │
│  │  habit-tracker-  │              │   users/{uid}/       │    │
│  │  avatar-repo     │              │   customAvatar       │    │
│  │                  │              │                      │    │
│  │  avatars/users/  │              │   "https://raw..."   │    │
│  │    {userId}/     │              │                      │    │
│  │      avatar.png  │              │                      │    │
│  └──────────────────┘              └──────────────────────┘    │
└─────────────────────────────────────────────────────────────────┘
```

## Upload Flow
```
┌─────────────────────────────────────────────────────────────────┐
│ 1. USER SELECTS IMAGE                                           │
│    User taps upload icon → Image picker opens                   │
└─────────────────────────────────────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────────┐
│ 2. IMAGE PROCESSING                                             │
│    ┌──────────────────────────────────────────────────────┐    │
│    │ Original Image (any size)                            │    │
│    └──────────────────────────────────────────────────────┘    │
│                            │                                    │
│                            ▼                                    │
│    ┌──────────────────────────────────────────────────────┐    │
│    │ Resize to max 512x512 (maintain aspect ratio)        │    │
│    └──────────────────────────────────────────────────────┘    │
│                            │                                    │
│                            ▼                                    │
│    ┌──────────────────────────────────────────────────────┐    │
│    │ Compress to PNG (85% quality)                        │    │
│    └──────────────────────────────────────────────────────┘    │
│                            │                                    │
│                            ▼                                    │
│    ┌──────────────────────────────────────────────────────┐    │
│    │ Convert to Base64                                    │    │
│    └──────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────────┐
│ 3. UPLOAD TO GITHUB                                             │
│    PUT /repos/{owner}/{repo}/contents/{path}                    │
│    Authorization: token {github_token}                          │
│    Body: { "message": "...", "content": "{base64}" }           │
│                                                                 │
│    Path: avatars/users/{userId}/avatar_{timestamp}.png         │
└─────────────────────────────────────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────────┐
│ 4. GET DOWNLOAD URL                                             │
│    Response: { "content": { "download_url": "https://..." } }  │
│                                                                 │
│    URL Format: https://raw.githubusercontent.com/               │
│                {owner}/{repo}/main/avatars/users/{uid}/...      │
└─────────────────────────────────────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────────┐
│ 5. UPDATE FIRESTORE                                             │
│    Firestore.collection("users")                                │
│            .document(userId)                                    │
│            .update("customAvatar", downloadUrl)                 │
└─────────────────────────────────────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────────┐
│ 6. UI UPDATE                                                    │
│    • Avatar appears in grid with ⭐ badge                       │
│    • StateFlow triggers recomposition                           │
│    • User can select the new avatar                            │
│    • Avatar displays across all screens                        │
└─────────────────────────────────────────────────────────────────┘
```

## Delete Flow
```
┌─────────────────────────────────────────────────────────────────┐
│ 1. USER INITIATES DELETE                                        │
│    User taps delete icon → Confirmation dialog shows            │
└─────────────────────────────────────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────────┐
│ 2. GET FILE SHA                                                 │
│    GET /repos/{owner}/{repo}/contents/{path}                    │
│    Response: { "sha": "abc123..." }                            │
└─────────────────────────────────────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────────┐
│ 3. DELETE FROM GITHUB                                           │
│    DELETE /repos/{owner}/{repo}/contents/{path}                 │
│    Body: { "message": "Delete...", "sha": "abc123..." }        │
└─────────────────────────────────────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────────┐
│ 4. UI UPDATE                                                    │
│    • Avatar removed from grid                                   │
│    • List refreshed automatically                               │
└─────────────────────────────────────────────────────────────────┘
```

## Avatar Display Priority
```
┌────────────────────────────────────────────────────────────┐
│ AVATAR DISPLAY LOGIC                                       │
│                                                            │
│  Is user signed in with Google?                           │
│          │                                                 │
│          ├─ YES → Has custom avatar?                      │
│          │        │                                        │
│          │        ├─ YES → Show Custom Avatar ⭐         │
│          │        │                                        │
│          │        └─ NO  → Show Google Photo 📸          │
│          │                                                 │
│          └─ NO  → Has custom avatar?                      │
│                   │                                        │
│                   ├─ YES → Show Custom Avatar ⭐          │
│                   │                                        │
│                   └─ NO  → Show Default Icon 👤           │
│                                                            │
└────────────────────────────────────────────────────────────┘
```

## User Experience Journey
```
┌──────────────────────────────────────────────────────────────┐
│ STEP 1: ACCESS AVATAR PICKER                                 │
│                                                              │
│  Profile Screen                                              │
│  ┌────────────────────┐                                     │
│  │   [Profile Photo]  │  ← User taps here                   │
│  │                    │                                     │
│  │   User Name        │                                     │
│  └────────────────────┘                                     │
└──────────────────────────────────────────────────────────────┘
                    │
                    ▼
┌──────────────────────────────────────────────────────────────┐
│ STEP 2: VIEW AVATARS                                         │
│                                                              │
│  ┌────────────── Avatar Picker ──────────────┐             │
│  │  Choose Your Avatar          [Upload 📤]  │             │
│  │  ┌────┬────┬────┐                         │             │
│  │  │ 1  │ 2  │ 3  │  ← Default avatars       │             │
│  │  ├────┼────┼────┤                         │             │
│  │  │ 4  │ 5  │ 6  │                         │             │
│  │  ├────┼────┼────┤                         │             │
│  │  │ 7  │ 8  │⭐9 │  ← Custom with badge    │             │
│  │  └────┴────┴────┘                         │             │
│  │                                            │             │
│  │  [Close]                                   │             │
│  └────────────────────────────────────────────┘             │
└──────────────────────────────────────────────────────────────┘
                    │
        ┌───────────┴───────────┐
        ▼                       ▼
┌────────────────┐    ┌─────────────────────┐
│ OPTION A:      │    │ OPTION B:           │
│ Select Avatar  │    │ Upload New          │
│                │    │                     │
│ ┌────┐        │    │  1. Tap upload icon │
│ │ ✓  │        │    │  2. Pick image      │
│ └────┘        │    │  3. Wait for upload │
│                │    │  4. See progress    │
│ Instant update │    │  5. Avatar appears  │
└────────────────┘    └─────────────────────┘
```

## GitHub Repository View
```
github.com/gongobongofounder/habit-tracker-avatar-repo
│
├── avatars/
│   └── users/
│       ├── uid_abc123/
│       │   ├── avatar_1697123456789.png  ← User 1's avatar
│       │   └── avatar_1697234567890.png  ← User 1's 2nd avatar
│       │
│       ├── uid_def456/
│       │   └── avatar_1697345678901.png  ← User 2's avatar
│       │
│       └── uid_ghi789/
│           └── avatar_1697456789012.png  ← User 3's avatar
```

## Security Flow
```
┌──────────────────────────────────────────────────────────────┐
│ TOKEN SECURITY FLOW                                          │
│                                                              │
│  Developer Setup                                             │
│  ┌─────────────────────────────────────────────────────┐   │
│  │ 1. Generate GitHub Token                            │   │
│  │ 2. Store in secure location                         │   │
│  └─────────────────────────────────────────────────────┘   │
│                    │                                        │
│                    ▼                                        │
│  ┌─────────────────────────────────────────────────────┐   │
│  │ SecureTokenStorage.storeToken()                     │   │
│  │   ↓                                                 │   │
│  │ EncryptedSharedPreferences                         │   │
│  │   ↓                                                 │   │
│  │ AES256-GCM Encryption                              │   │
│  │   ↓                                                 │   │
│  │ Android Keystore                                   │   │
│  └─────────────────────────────────────────────────────┘   │
│                    │                                        │
│                    ▼                                        │
│  ┌─────────────────────────────────────────────────────┐   │
│  │ App Runtime                                         │   │
│  │   • Token retrieved on demand                       │   │
│  │   • Used for GitHub API calls                       │   │
│  │   • Never exposed to UI or logs                     │   │
│  │   • Stored only in memory when active               │   │
│  └─────────────────────────────────────────────────────┘   │
└──────────────────────────────────────────────────────────────┘
```

## Error Handling Flow
```
┌──────────────────────────────────────────────────────────────┐
│ ERROR SCENARIOS & HANDLING                                   │
│                                                              │
│  Upload Attempt                                              │
│        │                                                     │
│        ├─ No Internet                                        │
│        │     └─> Show: "No internet connection"             │
│        │                                                     │
│        ├─ Invalid Token                                      │
│        │     └─> Show: "Authentication failed"              │
│        │                                                     │
│        ├─ Image Too Large                                    │
│        │     └─> Compress & Retry                           │
│        │                                                     │
│        ├─ GitHub API Error                                   │
│        │     └─> Show: "Upload failed, try again"           │
│        │                                                     │
│        └─ Success                                            │
│              └─> Update UI & Firestore                       │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

## State Management
```
┌──────────────────────────────────────────────────────────────┐
│ AVATAR PICKER STATE                                          │
│                                                              │
│  AvatarPickerUiState                                         │
│  ┌────────────────────────────────────────────────────┐     │
│  │ avatars: List<AvatarItem>                          │     │
│  │   ├─ Default avatars (8)                           │     │
│  │   └─ Custom avatars (user-specific)                │     │
│  │                                                     │     │
│  │ isLoading: Boolean                                 │     │
│  │   └─ true: Show loading spinner                    │     │
│  │                                                     │     │
│  │ isUploading: Boolean                               │     │
│  │   └─ true: Show progress bar                       │     │
│  │                                                     │     │
│  │ errorMessage: String?                              │     │
│  │   └─ non-null: Show error banner                   │     │
│  └────────────────────────────────────────────────────┘     │
│                                                              │
│  State Transitions:                                          │
│  Initial → Loading → Loaded                                  │
│           Loading → Error → Retry                            │
│           Loaded → Uploading → Uploaded → Reloading          │
└──────────────────────────────────────────────────────────────┘
```

This visual guide helps understand the complete flow of the custom avatar upload feature! 🎨
