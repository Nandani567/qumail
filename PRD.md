[QuMail Backend Product Requirements Document
Quantum-Secured Email Client - Electron.js Application
Version: 1.0.0
Last Updated: January 11, 2026
Status: Draft

Document Information
FieldDetailsProject NameQuMail (Quantum-Secured Email Client)Document TypeBackend Technical PRDPlatformElectron.js Desktop Application (Windows)Target OSWindows 10/11 (Primary)Prepared ByDevelopment TeamStakeholdersEngineering, Security, Product ManagementDocument OwnerBackend Lead Engineer

Table of Contents

Backend Overview
Technology Stack
Electron Architecture & Process Model
IPC Communication Patterns
Security Architecture
QKD & ETSI GS QKD 014 Integration
Multi-Level Security Configuration
Encryption Service Architecture
Email Protocol Integration
Email Operations Architecture
Local Storage & Database Schema
Service-Oriented Architecture
Key Management Service Design
Error Handling & Logging Strategy
Configuration Management
Application State Management
Development Workflow
Testing Strategy
Build & Packaging Configuration
Deployment & Distribution
Security Hardening Checklist
Monitoring & Diagnostics
Documentation Requirements
Modular Architecture for Application Suite
Phase 2 Features (Future Roadmap)
Success Metrics & KPIs
Troubleshooting Guide
Resources & References
Glossary of Terms
Appendix: Sample Code Templates
Document Version History


1. Backend Overview
   1.1 Project Context
   QuMail is a quantum-secured email client application designed to enhance email communication security through integration with Quantum Key Distribution (QKD) services. The application addresses the growing security concerns in the post-quantum era where conventional encryption mechanisms are vulnerable to quantum computing attacks.
   1.2 Core Problem Statement
   Email communication faces multiple security threats:

Eavesdropping on email transmissions
Man-in-the-middle attacks
Data breaches in the post-quantum computing era
Compromise of traditional encryption keys

1.3 Solution Approach
QuMail integrates QKD technology with standard email protocols to provide:

Unconditional security through quantum key distribution
Compatibility with existing email infrastructure (Gmail, Yahoo, etc.)
Multiple security levels from quantum-secure to standard encryption
Modular architecture for future expansion into a full communication suite

1.4 Backend Architecture Philosophy
The QuMail backend follows an Electron.js Main Process architecture with these principles:
Process Separation

Main Process: Core business logic, system integration, security operations
Renderer Process: UI layer (separate PRD)
Secure IPC bridge for inter-process communication

Service-Oriented Design

Modular services with single responsibilities
Dependency injection for testability
Clear separation of concerns

Security-First Approach

Process isolation and privilege separation
Encrypted storage for sensitive data
Secure IPC channels with type-safe contracts
Context isolation and sandboxing

Offline]()-First with Sync

Local SQLite database for email storage
Background synchronization with email servers
Cached quantum keys for offline encryption

1.5 Key Components
┌─────────────────────────────────────────────────────────┐
│                   ELECTRON MAIN PROCESS                  │
├─────────────────────────────────────────────────────────┤
│                                                           │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  │
│  │ IPC Handlers │  │   Services   │  │ Repositories │  │
│  └──────────────┘  └──────────────┘  └──────────────┘  │
│         │                  │                  │          │
│         ▼                  ▼                  ▼          │
│  ┌──────────────────────────────────────────────────┐  │
│  │         External Integration Layer                │  │
│  ├──────────────────────────────────────────────────┤  │
│  │  KM Service  │  Email Servers  │  Local Storage  │  │
│  │  (ETSI API)  │  (SMTP/IMAP)   │   (SQLite)      │  │
│  └──────────────────────────────────────────────────┘  │
│                                                           │
└─────────────────────────────────────────────────────────┘
│
│ Secure IPC
│
┌─────────────────────────────────────────────────────────┐
│              ELECTRON RENDERER PROCESS (UI)              │
└─────────────────────────────────────────────────────────┘
1.6 Scope of This PRD
This document covers:

✅ Electron Main Process architecture
✅ Backend services and business logic
✅ QKD integration (ETSI GS QKD 014)
✅ Email protocol integration
✅ Encryption services
✅ Data persistence
✅ IPC contracts and handlers

Out of Scope:

❌ Renderer Process implementation (UI/UX)
❌ Actual QKD hardware/simulator implementation
❌ Email server infrastructure


2. Technology Stack
   2.1 Core Framework
   TechnologyVersionPurposeElectron.js28.xDesktop application frameworkNode.js20.x LTSRuntime environment (main process)TypeScript5.3.xType-safe developmentWebpack5.xModule bundling
   2.2 Email Integration
   LibraryVersionPurposenodemailer6.9.xSMTP email sendingimap-simple5.xIMAP email retrievalmailparser3.xEmail parsing and MIME handlingaddressparser1.xEmail address validation
   2.3 Cryptography & Security
   LibraryVersionPurposenode:cryptoBuilt-inCore encryption (AES, hashing)tweetnacl1.xPost-quantum cryptography optionbcrypt5.xPassword hashingjsonwebtoken9.xToken-based authenticationnode-forge1.xCertificate management, PKI
   2.4 Data Persistence
   LibraryVersionPurposebetter-sqlite39.xLocal SQLite databaseelectron-store8.xEncrypted configuration storagekeytar7.xSystem keychain integration
   2.5 HTTP & API Integration
   LibraryVersionPurposeaxios1.6.xHTTP client for KM APIhttpsBuilt-inSecure HTTPS connectionsretry-axios3.xAutomatic retry logic
   2.6 Utilities & Helpers
   LibraryVersionPurposezod3.xRuntime type validationdate-fns3.xDate manipulationuuid9.xUnique identifier generationlodash4.xUtility functions
   2.7 Development & Testing
   LibraryVersionPurposeJest29.xUnit testing frameworkts-jest29.xTypeScript Jest integration@playwright/test1.xE2E testing for Electronelectron-builder24.xBuild and packagingelectron-updater6.xAuto-update functionality
   2.8 Logging & Monitoring
   LibraryVersionPurposeelectron-log5.xApplication loggingwinston3.xAdvanced logging (production)@sentry/electron4.xError tracking and monitoring
   2.9 Code Quality
   ToolVersionPurposeESLint8.xCode lintingPrettier3.xCode formattinghusky8.xGit hookslint-staged15.xPre-commit linting
   2.10 Technology Justification
   Why Electron.js?

Cross-platform desktop support (future macOS/Linux)
Rich ecosystem for desktop features
Web technologies (easier to find developers)
Strong security model with proper configuration
Excellent auto-update capabilities

Why TypeScript?

Type safety reduces runtime errors
Better IDE support and autocomplete
Self-documenting code
Easier refactoring
Industry standard for large applications

Why SQLite (better-sqlite3)?

Offline-first architecture
No server dependency
Fast local queries
Synchronous API (simpler error handling)
Portable database file

Why nodemailer + imap-simple?

Industry-standard email libraries
Support for all major email providers
OAuth2 authentication support
Well-maintained and documented
Flexible configuration


3. Electron Architecture & Process Model
   3.1 Electron Multi-Process Architecture
   Electron applications run in two types of processes:
   ┌───────────────────────────────────────────────────────────┐
   │                     ELECTRON APP                           │
   ├───────────────────────────────────────────────────────────┤
   │                                                             │
   │  ┌─────────────────────────────────────────────────────┐  │
   │  │              MAIN PROCESS (Node.js)                  │  │
   │  ├─────────────────────────────────────────────────────┤  │
   │  │  • Full Node.js API access                          │  │
   │  │  • System-level operations                          │  │
   │  │  • Window management                                │  │
   │  │  • Native integrations (file system, network)       │  │
   │  │  • Backend business logic (THIS PRD'S SCOPE)        │  │
   │  │  • Electron APIs (app, BrowserWindow, ipcMain)      │  │
   │  └─────────────────────────────────────────────────────┘  │
   │                          │                                  │
   │                          │ IPC (Inter-Process)              │
   │                          │ Communication                    │
   │                          │                                  │
   │  ┌─────────────────────────────────────────────────────┐  │
   │  │           RENDERER PROCESS (Chromium)                │  │
   │  ├─────────────────────────────────────────────────────┤  │
   │  │  • Web technologies (HTML, CSS, JavaScript)         │  │
   │  │  • React/Vue framework                              │  │
   │  │  • User interface rendering                         │  │
   │  │  • Limited access (sandboxed for security)          │  │
   │  │  • Communicates via IPC only                        │  │
   │  │  • One per window                                   │  │
   │  └─────────────────────────────────────────────────────┘  │
   │                                                             │
   │  ┌─────────────────────────────────────────────────────┐  │
   │  │              PRELOAD SCRIPT (Bridge)                 │  │
   │  ├─────────────────────────────────────────────────────┤  │
   │  │  • Runs before renderer loads                       │  │
   │  │  • Has access to Node.js APIs                       │  │
   │  │  • Exposes safe APIs to renderer via contextBridge  │  │
   │  │  • Security boundary enforcement                    │  │
   │  └─────────────────────────────────────────────────────┘  │
   │                                                             │
   └───────────────────────────────────────────────────────────┘
   3.2 Main Process Responsibilities
   The Main Process is the heart of QuMail's backend and handles:
   System Integration

Email server connections (SMTP, IMAP)
Key Manager API integration (ETSI QKD 014)
File system operations (attachments, logs)
System notifications
Auto-update management

Business Logic

Email encryption/decryption
Key lifecycle management
Email composition and parsing
Account management
Configuration management

Data Management

SQLite database operations
Encrypted storage management
Cache management
Session management

Security Operations

Credential storage (system keychain)
Certificate validation
Secure key storage
Encryption key derivation

Window Management

Creating and managing application windows
Menu bar and tray icon management
Deep linking and protocol handlers

3.3 Renderer Process Responsibilities
Each window runs in its own renderer process:
UI Rendering (Out of scope for this PRD)

Email inbox display
Composition window
Settings interface
Key Manager dashboard

User Interaction

Form handling
User input validation
Visual feedback

State Management

UI state (separate from backend state)
Local component state

3.4 Security Model
QuMail implements Electron's security best practices:
Context Isolation (Enabled)
typescript// main.ts - BrowserWindow configuration
const mainWindow = new BrowserWindow({
width: 1200,
height: 800,
webPreferences: {
contextIsolation: true, // ✅ Isolate renderer from preload
nodeIntegration: false, // ✅ No direct Node.js in renderer
sandbox: true,          // ✅ Full sandbox
preload: path.join(__dirname, 'preload.js')
}
});
Why Context Isolation?

Prevents renderer from accessing Node.js globals
Protects against malicious scripts injecting into window object
Forces all communication through controlled IPC channels

Node Integration Disabled

Renderer cannot use require() or Node.js APIs
Reduces attack surface
Prevents XSS from escalating to system access

Sandbox Mode Enabled

Chromium sandbox restricts renderer privileges
Limits file system and network access
Additional layer of defense

Content Security Policy (CSP)
typescript// Set strict CSP headers
session.defaultSession.webRequest.onHeadersReceived((details, callback) => {
callback({
responseHeaders: {
...details.responseHeaders,
'Content-Security-Policy': [
"default-src 'self'",
"script-src 'self'",
"style-src 'self' 'unsafe-inline'",
"img-src 'self' data:",
"connect-src 'self' https://km-api.example.com"
].join('; ')
}
});
});
```

### **3.5 Process Separation Diagram**
```
Main Process                  Preload Script              Renderer Process
─────────────                ───────────────             ─────────────────

┌─────────────┐                                         ┌─────────────┐
│ Services    │                                         │   React     │
│  - KM       │                                         │     UI      │
│  - Email    │              ┌──────────────┐          │             │
│  - Crypto   │◄─────────────│ contextBridge│──────────►│ window.api  │
│  - Storage  │   IpcMain    │              │ Exposed  │             │
│             │   handlers   │  Safe APIs   │ APIs     │             │
└─────────────┘              └──────────────┘          └─────────────┘
│                             │                         │
│                             │                         │
▼                             ▼                         ▼
Full Node.js API          Limited Node.js API      No Node.js Access
File System Access        No Direct Access         Sandbox Restricted
Network Access           Bridge Only               IPC Only
Database Access          Type-Safe Interface       Event Listeners
3.6 Window Architecture
QuMail uses multiple windows for different functionalities:
typescript// Window types and their purposes
enum WindowType {
MAIN = 'main',           // Primary inbox window
COMPOSER = 'composer',   // Email composition
SETTINGS = 'settings',   // Application settings
KM_DASHBOARD = 'km',     // Key Manager dashboard
VIEWER = 'viewer'        // Email/attachment viewer
}

// Each window gets its own renderer process
class WindowManager {
private windows: Map<string, BrowserWindow> = new Map();

createWindow(type: WindowType, options?: BrowserWindowConstructorOptions) {
const window = new BrowserWindow({
...this.getDefaultOptions(),
...options,
webPreferences: {
contextIsolation: true,
nodeIntegration: false,
sandbox: true,
preload: path.join(__dirname, 'preload.js')
}
});

    this.windows.set(type, window);
    return window;
}
}
```

### **3.7 Process Lifecycle**
```
Application Startup
│
▼
┌──────────────────┐
│ app.on('ready')  │ Main process starts
└──────────────────┘
│
▼
┌──────────────────────────┐
│ Initialize Services      │ - Load config
│                          │ - Connect database
│                          │ - Setup IPC handlers
└──────────────────────────┘
│
▼
┌──────────────────────────┐
│ Create Main Window       │ First renderer spawns
└──────────────────────────┘
│
▼
┌──────────────────────────┐
│ Load Application State   │ - Load accounts
│                          │ - Check for updates
│                          │ - Sync emails
└──────────────────────────┘
│
▼
Application Ready
```

---

## **4. IPC Communication Patterns**

### **4.1 IPC Overview**

Inter-Process Communication (IPC) is the secure bridge between main and renderer processes. QuMail uses a **type-safe, contract-based IPC system**.

### **4.2 IPC Architecture**
```
Renderer Process          Preload Script           Main Process
───────────────          ───────────────          ─────────────

┌─────────────┐          ┌─────────────┐          ┌─────────────┐
│   React     │          │ Context     │          │   IPC       │
│  Component  │─invoke──►│   Bridge    │─invoke──►│  Handler    │
│             │          │             │          │             │
│             │◄─result──│             │◄─result──│             │
└─────────────┘          └─────────────┘          └─────────────┘
│
▼
┌─────────────┐
│  Service    │
│   Layer     │
└─────────────┘
4.3 IPC Contract Definition
All IPC channels use TypeScript interfaces for type safety:
typescript// src/types/ipc-contracts.ts

/**
* IPC Channel Names - Centralized registry
  */
  export const IPC_CHANNELS = {
  // Email Operations
  EMAIL_SEND: 'email:send',
  EMAIL_FETCH_LIST: 'email:fetch-list',
  EMAIL_FETCH_ONE: 'email:fetch-one',
  EMAIL_DELETE: 'email:delete',
  EMAIL_MARK_READ: 'email:mark-read',

// Key Management
KM_AUTHENTICATE: 'km:authenticate',
KM_GET_KEY: 'km:get-key',
KM_LIST_KEYS: 'km:list-keys',
KM_REVOKE_KEY: 'km:revoke-key',

// Account Management
ACCOUNT_ADD: 'account:add',
ACCOUNT_LIST: 'account:list',
ACCOUNT_REMOVE: 'account:remove',
ACCOUNT_UPDATE: 'account:update',

// Configuration
CONFIG_GET: 'config:get',
CONFIG_SET: 'config:set',
CONFIG_RESET: 'config:reset',

// Encryption
ENCRYPT_MESSAGE: 'encrypt:message',
DECRYPT_MESSAGE: 'decrypt:message',

// System
SYSTEM_GET_INFO: 'system:get-info',
SYSTEM_CHECK_UPDATE: 'system:check-update',
} as const;

/**
* Request/Response types for each channel
  */

// Email Operations
export interface SendEmailRequest {
accountId: string;
to: string[];
cc?: string[];
bcc?: string[];
subject: string;
body: string;
attachments?: Array<{
filename: string;
path: string;
contentType?: string;
}>;
securityLevel: SecurityLevel;
}

export interface SendEmailResponse {
success: boolean;
messageId: string;
encryptionUsed: boolean;
keyId?: string;
error?: string;
}

export interface FetchEmailListRequest {
accountId: string;
folder?: string; // 'INBOX', 'SENT', etc.
limit?: number;
offset?: number;
unreadOnly?: boolean;
since?: Date;
}

export interface FetchEmailListResponse {
emails: EmailMetadata[];
total: number;
hasMore: boolean;
}

export interface EmailMetadata {
id: string;
uid: number;
from: string;
to: string[];
subject: string;
date: Date;
hasAttachments: boolean;
isEncrypted: boolean;
isRead: boolean;
size: number;
}

// Key Management
export interface KMAuthenticateRequest {
kmUrl: string;
apiKey?: string;
certificate?: string;
}

export interface KMAuthenticateResponse {
success: boolean;
sessionToken?: string;
expiresAt?: Date;
error?: string;
}

export interface GetKeyRequest {
keyId?: string;
purpose: 'encryption' | 'decryption';
algorithm: 'OTP' | 'AES' | 'PQC';
}

export interface GetKeyResponse {
key: Buffer;
keyId: string;
expiresAt: Date;
usageCount: number;
maxUsage: number;
}

// Account Management
export interface AddAccountRequest {
email: string;
password: string;
displayName: string;
imapConfig: IMAPConfig;
smtpConfig: SMTPConfig;
defaultSecurityLevel: SecurityLevel;
}

export interface IMAPConfig {
host: string;
port: number;
secure: boolean;
authMethod: 'password' | 'oauth2';
}

export interface SMTPConfig {
host: string;
port: number;
secure: boolean;
authMethod: 'password' | 'oauth2';
}

export interface AddAccountResponse {
success: boolean;
accountId: string;
error?: string;
}

// Security Levels
export enum SecurityLevel {
QUANTUM_SECURE = 'quantum-secure',     // OTP
QUANTUM_AIDED_AES = 'quantum-aided',   // AES with quantum key
POST_QUANTUM = 'post-quantum',         // PQC algorithms
STANDARD = 'standard'                   // Traditional encryption
}

// Configuration
export interface GetConfigRequest {
key: string;
defaultValue?: any;
}

export interface SetConfigRequest {
key: string;
value: any;
}
4.4 Main Process - IPC Handlers
typescript// src/main/ipc-handlers/email-handlers.ts

import { ipcMain } from 'electron';
import { IPC_CHANNELS, SendEmailRequest, SendEmailResponse } from '@/types/ipc-contracts';
import { EmailService } from '@/services/email-service';
import { logger } from '@/utils/logger';

export class EmailIPCHandlers {
constructor(private emailService: EmailService) {}

register() {
// Send Email Handler
ipcMain.handle(
IPC_CHANNELS.EMAIL_SEND,
async (event, request: SendEmailRequest): Promise<SendEmailResponse> => {
try {
logger.info('IPC: Sending email', {
accountId: request.accountId,
to: request.to
});

          const result = await this.emailService.sendEmail(request);

          logger.info('IPC: Email sent successfully', { 
            messageId: result.messageId 
          });

          return {
            success: true,
            messageId: result.messageId,
            encryptionUsed: result.encryptionUsed,
            keyId: result.keyId
          };
        } catch (error) {
          logger.error('IPC: Failed to send email', error);
          return {
            success: false,
            messageId: '',
            encryptionUsed: false,
            error: error instanceof Error ? error.message : 'Unknown error'
          };
        }
      }
    );

    // Fetch Email List Handler
    ipcMain.handle(
      IPC_CHANNELS.EMAIL_FETCH_LIST,
      async (event, request: FetchEmailListRequest) => {
        try {
          const result = await this.emailService.fetchEmailList(request);
          return result;
        } catch (error) {
          logger.error('IPC: Failed to fetch emails', error);
          throw error;
        }
      }
    );

    // Fetch Single Email Handler
    ipcMain.handle(
      IPC_CHANNELS.EMAIL_FETCH_ONE,
      async (event, emailId: string) => {
        try {
          const email = await this.emailService.fetchEmailById(emailId);
          return email;
        } catch (error) {
          logger.error('IPC: Failed to fetch email', error);
          throw error;
        }
      }
    );

    // Delete Email Handler
    ipcMain.handle(
      IPC_CHANNELS.EMAIL_DELETE,
      async (event, emailId: string) => {
        try {
          await this.emailService.deleteEmail(emailId);
          return { success: true };
        } catch (error) {
          logger.error('IPC: Failed to delete email', error);
          throw error;
        }
      }
    );

    logger.info('Email IPC handlers registered');
}

unregister() {
ipcMain.removeHandler(IPC_CHANNELS.EMAIL_SEND);
ipcMain.removeHandler(IPC_CHANNELS.EMAIL_FETCH_LIST);
ipcMain.removeHandler(IPC_CHANNELS.EMAIL_FETCH_ONE);
ipcMain.removeHandler(IPC_CHANNELS.EMAIL_DELETE);
logger.info('Email IPC handlers unregistered');
}
}
typescript// src/main/ipc-handlers/km-handlers.ts

import { ipcMain } from 'electron';
import { IPC_CHANNELS } from '@/types/ipc-contracts';
import { KeyManagementService } from '@/services/key-management-service';

export class KMIPCHandlers {
constructor(private kmService: KeyManagementService) {}

register() {
// Authenticate with Key Manager
ipcMain.handle(
IPC_CHANNELS.KM_AUTHENTICATE,
async (event, request) => {
return await this.kmService.authenticate(request);
}
);

    // Get Quantum Key
    ipcMain.handle(
      IPC_CHANNELS.KM_GET_KEY,
      async (event, request) => {
        return await this.kmService.getKey(request);
      }
    );

    // List Available Keys
    ipcMain.handle(
      IPC_CHANNELS.KM_LIST_KEYS,
      async (event) => {
        return await this.kmService.listKeys();
      }
    );
}
}
4.5 Preload Script - Context Bridge
typescript// src/preload/preload.ts

import { contextBridge, ipcRenderer } from 'electron';
import { IPC_CHANNELS } from '@/types/ipc-contracts';

/**
* Expose safe APIs to renderer process
* Only these methods will be available in window.api
  */
  contextBridge.exposeInMainWorld('api', {
  // Email APIs
  email: {
  send: (request: SendEmailRequest) =>
  ipcRenderer.invoke(IPC_CHANNELS.EMAIL_SEND, request),

  fetchList: (request: FetchEmailListRequest) =>
  ipcRenderer.invoke(IPC_CHANNELS.EMAIL_FETCH_LIST, request),

  fetchOne: (emailId: string) =>
  ipcRenderer.invoke(IPC_CHANNELS.EMAIL_FETCH_ONE, emailId),

  delete: (emailId: string) =>
  ipcRenderer.invoke(IPC_CHANNELS.EMAIL_DELETE, emailId),

  markRead: (emailId: string, read: boolean) =>
  ipcRenderer.invoke(IPC_CHANNELS.EMAIL_MARK_READ, emailId, read),
  },

// Key Management APIs
keyManager: {
authenticate: (request: KMAuthenticateRequest) =>
ipcRenderer.invoke(IPC_CHANNELS.KM_AUTHENTICATE, request),

    getKey: (request: GetKeyRequest) => 
      ipcRenderer.invoke(IPC_CHANNELS.KM_GET_KEY, request),
    
    listKeys: () => 
      ipcRenderer.invoke(IPC_CHANNELS.KM_LIST_KEYS),
},

// Account APIsaccount: {
add: (request: AddAccountRequest) =>
ipcRenderer.invoke(IPC_CHANNELS.ACCOUNT_ADD, request),
list: () =>
ipcRenderer.invoke(IPC_CHANNELS.ACCOUNT_LIST),

remove: (accountId: string) =>
ipcRenderer.invoke(IPC_CHANNELS.ACCOUNT_REMOVE, accountId),

update: (accountId: string, updates: Partial<AddAccountRequest>) =>
ipcRenderer.invoke(IPC_CHANNELS.ACCOUNT_UPDATE, accountId, updates),
},
// Configuration APIs
config: {
get: (key: string, defaultValue?: any) =>
ipcRenderer.invoke(IPC_CHANNELS.CONFIG_GET, key, defaultValue),
set: (key: string, value: any) =>
ipcRenderer.invoke(IPC_CHANNELS.CONFIG_SET, key, value),

reset: () =>
ipcRenderer.invoke(IPC_CHANNELS.CONFIG_RESET),
},
// System APIs
system: {
getInfo: () =>
ipcRenderer.invoke(IPC_CHANNELS.SYSTEM_GET_INFO),
checkUpdate: () =>
ipcRenderer.invoke(IPC_CHANNELS.SYSTEM_CHECK_UPDATE),
},
// Event Listeners (one-way communication from main to renderer)
on: (channel: string, callback: (...args: any[]) => void) => {
// Whitelist allowed channels
const allowedChannels = [
'email:new',
'email:sync-progress',
'km:key-expiring',
'system:update-available'
];
if (allowedChannels.includes(channel)) {
ipcRenderer.on(channel, (event, ...args) => callback(...args));
}
},
// Remove listener
off: (channel: string, callback: (...args: any[]) => void) => {
ipcRenderer.removeListener(channel, callback);
}
});
// TypeScript declarations for renderer
export interface ElectronAPI {
email: {
send: (request: SendEmailRequest) => Promise<SendEmailResponse>;
fetchList: (request: FetchEmailListRequest) => Promise<FetchEmailListResponse>;
fetchOne: (emailId: string) => Promise<EmailMessage>;
delete: (emailId: string) => Promise<{ success: boolean }>;
markRead: (emailId: string, read: boolean) => Promise<void>;
};
keyManager: {
authenticate: (request: KMAuthenticateRequest) => Promise<KMAuthenticateResponse>;
getKey: (request: GetKeyRequest) => Promise<GetKeyResponse>;
listKeys: () => Promise<KeyMetadata[]>;
};
account: {
add: (request: AddAccountRequest) => Promise<AddAccountResponse>;
list: () => Promise<EmailAccount[]>;
remove: (accountId: string) => Promise<void>;
update: (accountId: string, updates: Partial<AddAccountRequest>) => Promise<void>;
};
config: {
get: (key: string, defaultValue?: any) => Promise<any>;
set: (key: string, value: any) => Promise<void>;
reset: () => Promise<void>;
};
system: {
getInfo: () => Promise<SystemInfo>;
checkUpdate: () => Promise<UpdateInfo>;
};
on: (channel: string, callback: (...args: any[]) => void) => void;
off: (channel: string, callback: (...args: any[]) => void) => void;
}
declare global {
interface Window {
api: ElectronAPI;
}
}

### **4.6 Renderer Process Usage**
```typescript
// renderer/src/components/EmailComposer.tsx

import React, { useState } from 'react';
import { SendEmailRequest, SecurityLevel } from '@/types/ipc-contracts';

export const EmailComposer: React.FC = () => {
  const [to, setTo] = useState('');
  const [subject, setSubject] = useState('');
  const [body, setBody] = useState('');
  const [sending, setSending] = useState(false);

  const handleSend = async () => {
    setSending(true);
    try {
      const request: SendEmailRequest = {
        accountId: 'account-123',
        to: [to],
        subject,
        body,
        securityLevel: SecurityLevel.QUANTUM_SECURE
      };

      // Call main process via IPC
      const response = await window.api.email.send(request);

      if (response.success) {
        alert(`Email sent! Message ID: ${response.messageId}`);
      } else {
        alert(`Failed to send: ${response.error}`);
      }
    } catch (error) {
      alert(`Error: ${error.message}`);
    } finally {
      setSending(false);
    }
  };

  return (
    <div>
      <input value={to} onChange={e => setTo(e.target.value)} placeholder="To" />
      <input value={subject} onChange={e => setSubject(e.target.value)} placeholder="Subject" />
      <textarea value={body} onChange={e => setBody(e.target.value)} />
      <button onClick={handleSend} disabled={sending}>
        {sending ? 'Sending...' : 'Send Quantum-Secured Email'}
      </button>
    </div>
  );
};
```

### **4.7 Bidirectional Communication**

For real-time updates from main to renderer:
```typescript
// Main Process - Send event to renderer
import { BrowserWindow } from 'electron';

class EmailSyncService {
  async syncEmails() {
    // ... sync logic
    
    // Notify all windows
    BrowserWindow.getAllWindows().forEach(window => {
      window.webContents.send('email:new', {
        count: 5,
        latest: emailMetadata
      });
    });
  }
}

// Renderer Process - Listen for events
useEffect(() => {
  const handleNewEmail = (data: { count: number; latest: EmailMetadata }) => {
    console.log(`${data.count} new emails!`);
    // Update UI
  };

  window.api.on('email:new', handleNewEmail);

  return () => {
    window.api.off('email:new', handleNewEmail);
  };
}, []);
```

### **4.8 IPC Security Best Practices**

**Input Validation**
```typescript
// Always validate data from renderer
ipcMain.handle('email:send', async (event, request) => {
  // Validate using Zod
  const schema = z.object({
    accountId: z.string().uuid(),
    to: z.array(z.string().email()),
    subject: z.string().max(200),
    body: z.string(),
  });

  const validated = schema.parse(request);
  // Proceed with validated data
});
```

**Rate Limiting**
```typescript
// Prevent IPC flooding
const rateLimiter = new Map<string, number>();

ipcMain.handle('email:send', async (event, request) => {
  const now = Date.now();
  const lastCall = rateLimiter.get('email:send') || 0;
  
  if (now - lastCall < 1000) {
    throw new Error('Rate limit exceeded');
  }
  
  rateLimiter.set('email:send', now);
  // Continue...
});
```

---

## **5. Security Architecture**

### **5.1 Security Overview**

QuMail implements defense-in-depth security with multiple layers:
┌─────────────────────────────────────────────────────────┐
│                   Security Layers                        │
├─────────────────────────────────────────────────────────┤
│                                                           │
│  Layer 1: Process Isolation & Sandboxing                │
│  ├─ Context isolation enabled                           │
│  ├─ Node integration disabled                           │
│  └─ Chromium sandbox active                             │
│                                                           │
│  Layer 2: IPC Security                                  │
│  ├─ Type-safe contracts                                 │
│  ├─ Input validation (Zod schemas)                      │
│  ├─ Rate limiting                                       │
│  └─ Channel whitelisting                                │
│                                                           │
│  Layer 3: Data Encryption                               │
│  ├─ Quantum keys (QKD)                                  │
│  ├─ Email encryption (OTP/AES/PQC)                      │
│  ├─ Database encryption (SQLCipher)                     │
│  └─ Configuration encryption (electron-store)           │
│                                                           │
│  Layer 4: Credential Management                         │
│  ├─ System keychain integration (keytar)                │
│  ├─ No plaintext passwords                              │
│  ├─ Token-based authentication                          │
│  └─ Certificate pinning                                 │
│                                                           │
│  Layer 5: Network Security                              │
│  ├─ TLS/SSL for all connections                         │
│  ├─ Certificate validation                              │
│  ├─ CSP headers                                         │
│  └─ CORS policy                                         │
│                                                           │
│  Layer 6: Code Integrity                                │
│  ├─ Code signing (Windows Authenticode)                 │
│  ├─ Auto-update signature verification                  │
│  └─ Subresource integrity                               │
│                                                           │
└─────────────────────────────────────────────────────────┘

### **5.2 Content Security Policy (CSP)**
```typescript
// src/main/security/csp.ts

import { session } from 'electron';

export function setupCSP() {
  const CSP_DIRECTIVES = {
    'default-src': ["'self'"],
    'script-src': ["'self'"],
    'style-src': ["'self'", "'unsafe-inline'"], // Required for some UI frameworks
    'img-src': ["'self'", 'data:', 'https:'],
    'font-src': ["'self'", 'data:'],
    'connect-src': [
      "'self'",
      'https://km-api.example.com', // Key Manager API
      'https://*.googleapis.com',   // Gmail API (if using OAuth2)
    ],
    'frame-src': ["'none'"],
    'object-src': ["'none'"],
    'base-uri': ["'self'"],
    'form-action': ["'self'"],
    'upgrade-insecure-requests': []
  };

  const cspString = Object.entries(CSP_DIRECTIVES)
    .map(([key, values]) => `${key} ${values.join(' ')}`)
    .join('; ');

  session.defaultSession.webRequest.onHeadersReceived((details, callback) => {
    callback({
      responseHeaders: {
        ...details.responseHeaders,
        'Content-Security-Policy': [cspString]
      }
    });
  });
}
```

### **5.3 Secure Storage Architecture**
```typescript
// src/main/security/secure-storage.ts

import Store from 'electron-store';
import keytar from 'keytar';
import crypto from 'crypto';

const SERVICE_NAME = 'QuMail';

/**
 * Encrypted configuration storage
 */
export class SecureConfigStore {
  private store: Store;

  constructor() {
    // Get encryption key from system keychain
    const encryptionKey = this.getOrCreateEncryptionKey();

    this.store = new Store({
      encryptionKey,
      name: 'config',
      clearInvalidConfig: true,
    });
  }

  private getOrCreateEncryptionKey(): string {
    const KEY_NAME = 'config-encryption-key';
    
    // Try to get existing key
    let key = keytar.getPassword(SERVICE_NAME, KEY_NAME);
    
    if (!key) {
      // Generate new key
      key = crypto.randomBytes(32).toString('hex');
      keytar.setPassword(SERVICE_NAME, KEY_NAME, key);
    }
    
    return key;
  }

  get(key: string, defaultValue?: any): any {
    return this.store.get(key, defaultValue);
  }

  set(key: string, value: any): void {
    this.store.set(key, value);
  }

  delete(key: string): void {
    this.store.delete(key);
  }

  clear(): void {
    this.store.clear();
  }
}

/**
 * Credential storage using system keychain
 */
export class CredentialStore {
  /**
   * Store email account credentials
   */
  async setEmailCredentials(
    accountId: string,
    email: string,
    password: string
  ): Promise<void> {
    const account = `email-${accountId}`;
    await keytar.setPassword(SERVICE_NAME, account, JSON.stringify({
      email,
      password,
      storedAt: new Date().toISOString()
    }));
  }

  /**
   * Retrieve email account credentials
   */
  async getEmailCredentials(accountId: string): Promise<{
    email: string;
    password: string;
  } | null> {
    const account = `email-${accountId}`;
    const credentials = await keytar.getPassword(SERVICE_NAME, account);
    
    if (!credentials) return null;
    
    const parsed = JSON.parse(credentials);
    return {
      email: parsed.email,
      password: parsed.password
    };
  }

  /**
   * Store KM API credentials
   */
  async setKMCredentials(apiKey: string): Promise<void> {
    await keytar.setPassword(SERVICE_NAME, 'km-api-key', apiKey);
  }

  /**
   * Get KM API credentials
   */
  async getKMCredentials(): Promise<string | null> {
    return await keytar.getPassword(SERVICE_NAME, 'km-api-key');
  }

  /**
   * Delete credentials
   */
  async deleteCredentials(accountId: string): Promise<void> {
    const account = `email-${accountId}`;
    await keytar.deletePassword(SERVICE_NAME, account);
  }
}
```

### **5.4 Database Encryption**
```typescript
// src/main/database/encrypted-db.ts

import Database from 'better-sqlite3';
import crypto from 'crypto';
import { app } from 'electron';
import path from 'path';

export class EncryptedDatabase {
  private db: Database.Database;
  private encryptionKey: Buffer;

  constructor(filename: string) {
    const dbPath = path.join(app.getPath('userData'), 'data', filename);
    
    // Get encryption key from keychain
    this.encryptionKey = this.getDatabaseKey();
    
    // Open database
    this.db = new Database(dbPath);
    
    // Enable WAL mode for better performance
    this.db.pragma('journal_mode = WAL');
    
    // Note: For full database encryption, consider using SQLCipher
    // This example shows field-level encryption
  }

  private getDatabaseKey(): Buffer {
    // In production, retrieve from system keychain
    // For now, derive from machine-specific data
    const machineId = require('node-machine-id').machineIdSync();
    return crypto.pbkdf2Sync(machineId, 'salt', 100000, 32, 'sha512');
  }

  /**
   * Encrypt sensitive field
   */
  encrypt(data: string): string {
    const iv = crypto.randomBytes(16);
    const cipher = crypto.createCipheriv('aes-256-gcm', this.encryptionKey, iv);
    
    let encrypted = cipher.update(data, 'utf8', 'hex');
    encrypted += cipher.final('hex');
    
    const authTag = cipher.getAuthTag();
    
    // Return: iv + authTag + encrypted
    return iv.toString('hex') + authTag.toString('hex') + encrypted;
  }

  /**
   * Decrypt sensitive field
   */
  decrypt(encryptedData: string): string {
    const iv = Buffer.from(encryptedData.slice(0, 32), 'hex');
    const authTag = Buffer.from(encryptedData.slice(32, 64), 'hex');
    const encrypted = encryptedData.slice(64);
    
    const decipher = crypto.createDecipheriv('aes-256-gcm', this.encryptionKey, iv);
    decipher.setAuthTag(authTag);
    
    let decrypted = decipher.update(encrypted, 'hex', 'utf8');
    decrypted += decipher.final('utf8');
    
    return decrypted;
  }

  getDatabase(): Database.Database {
    return this.db;
  }

  close(): void {
    this.db.close();
  }
}
```

### **5.5 Certificate Pinning for KM API**
```typescript
// src/main/security/certificate-pinning.ts

import https from 'https';
import crypto from 'crypto';
import { Agent } from 'https';

/**
 * Certificate pinning for Key Manager API
 */
export class CertificatePinner {
  private pinnedFingerprints: Set<string>;
  private agent: Agent;

  constructor(fingerprints: string[]) {
    this.pinnedFingerprints = new Set(fingerprints);
    
    this.agent = new Agent({
      // Custom certificate validation
      checkServerIdentity: (hostname, cert) => {
        const fingerprint = crypto
          .createHash('sha256')
          .update(cert.raw)
          .digest('hex');

        if (!this.pinnedFingerprints.has(fingerprint)) {
          throw new Error(
            `Certificate fingerprint mismatch. Expected one of: ${Array.from(this.pinnedFingerprints).join(', ')}`
          );
        }

        // Also perform standard hostname check
        return https.checkServerIdentity(hostname, cert);
      }
    });
  }

  getAgent(): Agent {
    return this.agent;
  }
}

// Usage in KM service
const pinner = new CertificatePinner([
  'sha256-fingerprint-1',
  'sha256-fingerprint-2' // Backup certificate
]);

axios.create({
  httpsAgent: pinner.getAgent()
});
```

### **5.6 Input Validation & Sanitization**
```typescript
// src/main/security/validation.ts

import { z } from 'zod';

/**
 * Validation schemas for all inputs
 */
export const ValidationSchemas = {
  email: z.string().email().max(255),
  
  emailList: z.array(z.string().email()).max(100),
  
  subject: z.string()
    .max(200)
    .refine(val => !/<script|javascript:/i.test(val), {
      message: 'Subject contains potentially malicious content'
    }),
  
  emailBody: z.string()
    .max(1024 * 1024) // 1MB max
    .refine(val => !/<script|javascript:/i.test(val), {
      message: 'Body contains potentially malicious content'
    }),
  
  accountId: z.string().uuid(),
  
  securityLevel: z.nativeEnum(SecurityLevel),
  
  kmUrl: z.string().url().startsWith('https://'),
  
  apiKey: z.string().min(32).max(256),
};

/**
 * Sanitize HTML content
 */
export function sanitizeHtml(html: string): string {
  // Use DOMPurify or similar library
  // For now, simple removal of script tags
  return html
    .replace(/<script\b[^<]*(?:(?!<\/script>)<[^<]*)*<\/script>/gi, '')
    .replace(/on\w+="[^"]*"/gi, ''); // Remove inline event handlers
}

/**
 * Validate file paths (prevent directory traversal)
 */
export function validateFilePath(filepath: string): boolean {
  const normalized = path.normalize(filepath);
  const userDataPath = app.getPath('userData');
  
  // Ensure path is within app data directory
  return normalized.startsWith(userDataPath);
}
```

### **5.7 Protection Against Common Attacks**

**XSS Protection**
```typescript
// All user content is sanitized before rendering
// CSP prevents inline scripts
// Context isolation prevents access to Node.js APIs
```

**SQL Injection Protection**
```typescript
// Use parameterized queries
const stmt = db.prepare('SELECT * FROM emails WHERE id = ?');
const email = stmt.get(emailId);

// Never concatenate user input into SQL
// ❌ WRONG: db.prepare(`SELECT * FROM emails WHERE id = ${emailId}`)
```

**Path Traversal Protection**
```typescript
function saveAttachment(filename: string, data: Buffer) {
  // Sanitize filename
  const safeName = path.basename(filename).replace(/[^a-zA-Z0-9.-]/g, '_');
  const attachmentDir = path.join(app.getPath('userData'), 'attachments');
  const fullPath = path.join(attachmentDir, safeName);
  
  // Verify path is within allowed directory
  if (!fullPath.startsWith(attachmentDir)) {
    throw new Error('Invalid file path');
  }
  
  fs.writeFileSync(fullPath, data);
}
```

**CSRF Protection**
```typescript
// Not applicable for desktop apps (no web-based sessions)
// But implement request signing for KM API calls

function signRequest(payload: any, secret: string): string {
  const timestamp = Date.now();
  const message = JSON.stringify(payload) + timestamp;
  const signature = crypto
    .createHmac('sha256', secret)
    .update(message)
    .digest('hex');
  
  return signature;
}
```

### **5.8 Security Audit Checklist**

- [ ] Context isolation enabled in all BrowserWindows
- [ ] Node integration disabled in renderer processes
- [ ] Sandbox mode enabled
- [ ] CSP headers configured restrictively
- [ ] All IPC channels validated with Zod schemas
- [ ] Rate limiting on sensitive IPC handlers
- [ ] Credentials stored in system keychain (never in code)
- [ ] Database encrypted (field-level or full-disk)
- [ ] Configuration files encrypted (electron-store)
- [ ] Certificate pinning for KM API
- [ ] All HTTPS connections validated
- [ ] Input sanitization on all user inputs
- [ ] No eval() or Function() constructor usage
- [ ] No shell execution with user input
- [ ] Code signed with valid certificate
- [ ] Auto-updates verified with signature
- [ ] Logging doesn't expose sensitive data
- [ ] Error messages don't leak system information

---

## **6. QKD & ETSI GS QKD 014 Integration**

### **6.1 Quantum Key Distribution Overview**

Quantum Key Distribution (QKD) provides information-theoretic security for key exchange based on quantum mechanics principles. Unlike traditional key exchange methods, QKD security is guaranteed by the laws of physics, not computational complexity.

**Key Advantages:**
- Unconditional security (not dependent on computational assumptions)
- Eavesdropping detection (quantum states cannot be copied)
- Future-proof against quantum computing attacks
- Perfect forward secrecy

### **6.2 ETSI GS QKD 014 Protocol**

ETSI GS QKD 014 defines REST-based APIs for key delivery from Key Management systems. QuMail integrates with compliant Key Managers.

**Standard Compliance:**
- ETSI GS QKD 014 v1.1.1 (2019-02)
- RESTful API architecture
- JSON data format
- TLS 1.3 transport security
- Certificate-based authentication

### **6.3 Key Manager Service Interface**
```typescript
// src/types/km-types.ts

/**
 * ETSI GS QKD 014 data structures
 */

export interface ETSIKeyContainer {
  keys: ETSIKey[];
}

export interface ETSIKey {
  key_ID: string;              // Unique key identifier
  key: string;                 // Base64-encoded key material
  key_ID_extension?: string;   // Optional extension for key ID
}

export interface ETSIStatus {
  source_KME_ID: string;       // Source KME identifier
  target_KME_ID: string;       // Target KME identifier
  master_SAE_ID: string;       // Master SAE identifier
  slave_SAE_ID: string;        // Slave SAE identifier
  key_size: number;            // Key size in bits
  stored_key_count: number;    // Number of stored keys
  max_key_count: number;       // Maximum key capacity
  max_key_per_request: number; // Max keys per request
  max_key_size: number;        // Maximum key size in bits
  min_key_size: number;        // Minimum key size in bits
  max_SAE_ID_count: number;    // Maximum number of SAEs
}

export interface ETSIGetKeyRequest {
  number?: number;             // Number of keys requested (default: 1)
  size?: number;               // Key size in bits
  additional_slave_SAE_IDs?: string[]; // Additional slave SAE IDs
  extension_mandatory?: string[];      // Mandatory extensions
  extension_optional?: string[];       // Optional extensions
}

export interface ETSIGetKeyResponse {
  keys: ETSIKey[];
}

export interface ETSIDecKeyRequest {
  key_ID: string;              // Key ID to get DEC key for
}

export interface ETSIDecKeyResponse {
  keys: ETSIKey[];
}
```

### **6.4 Key Manager Service Implementation**
```typescript
// src/services/key-management-service.ts

import axios, { AxiosInstance } from 'axios';
import { app } from 'electron';
import path from 'path';
import fs from 'fs';
import https from 'https';
import { logger } from '@/utils/logger';
import {
  ETSIGetKeyRequest,
  ETSIGetKeyResponse,
  ETSIStatus,
  ETSIKey,
  ETSIDecKeyResponse
} from '@/types/km-types';
import { CredentialStore } from '@/security/secure-storage';
import { CertificatePinner } from '@/security/certificate-pinning';

export interface KMConfig {
  baseUrl: string;              // KM API base URL
  masterSAEId: string;          // This application's SAE ID
  slaveSAEId?: string;          // Target SAE ID (recipient)
  certificatePath?: string;     // Client certificate for mTLS
  certificateKeyPath?: string;  // Client certificate key
  apiKey?: string;              // API key (if used instead of certs)
  pinnedFingerprints?: string[]; // Certificate pinning
}

export class KeyManagementService {
  private client: AxiosInstance;
  private config: KMConfig;
  private credentialStore: CredentialStore;
  private sessionToken?: string;
  private sessionExpiry?: Date;
  private keyCache: Map<string, CachedKey> = new Map();

  constructor(config: KMConfig) {
    this.config = config;
    this.credentialStore = new CredentialStore();
    
    // Create axios client with TLS configuration
    this.client = this.createClient();
  }

  private createClient(): AxiosInstance {
    const httpsAgentOptions: https.AgentOptions = {
      rejectUnauthorized: true,
      minVersion: 'TLSv1.3',
    };

    // Add client certificate if provided
    if (this.config.certificatePath) {
      const certPath = path.resolve(this.config.certificatePath);
      const keyPath = path.resolve(this.config.certificateKeyPath!);
      
      httpsAgentOptions.cert = fs.readFileSync(certPath);
      httpsAgentOptions.key = fs.readFileSync(keyPath);
    }

    // Add certificate pinning if configured
    if (this.config.pinnedFingerprints) {
      const pinner = new CertificatePinner(this.config.pinnedFingerprints);
      httpsAgentOptions.checkServerIdentity = pinner.getAgent().options.checkServerIdentity;
    }

    return axios.create({
      baseURL: this.config.baseUrl,
      httpsAgent: new https.Agent(httpsAgentOptions),
      headers: {
        'Content-Type': 'application/json',
      },
      timeout: 30000, // 30 seconds
    });
  }

  /**
   * Authenticate with Key Manager
   * ETSI GS QKD 014: Section 6.1 - Authentication
   */
  async authenticate(): Promise<void> {
    try {
      logger.info('Authenticating with Key Manager', {
        baseUrl: this.config.baseUrl,
        saeId: this.config.masterSAEId
      });

      // If using API key authentication
      if (this.config.apiKey) {
        this.client.defaults.headers.common['Authorization'] = 
          `Bearer ${this.config.apiKey}`;
      }

      // Verify connection by getting status
      await this.getStatus();

      logger.info('Successfully authenticated with Key Manager');
    } catch (error) {
      logger.error('Failed to authenticate with Key Manager', error);
      throw new Error(`KM authentication failed: ${error.message}`);
    }
  }

  /**
   * Get KM status
   * ETSI GS QKD 014: GET /api/v1/keys/{master_SAE_ID}/status
   */
  async getStatus(): Promise<ETSIStatus> {
    try {
      const response = await this.client.get<ETSIStatus>(
        `/api/v1/keys/${this.config.masterSAEId}/status`
      );

      logger.info('KM Status retrieved', response.data);
      return response.data;
    } catch (error) {
      logger.error('Failed to get KM status', error);
      throw error;
    }
  }

  /**
   * Get encryption key(s) from KM
   * ETSI GS QKD 014: POST /api/v1/keys/{master_SAE_ID}/enc_keys
   */
  async getEncryptionKey(
    slaveSAEId: string,
    request: ETSIGetKeyRequest = {}
  ): Promise<ETSIKey[]> {
    try {
      const endpoint = `/api/v1/keys/${this.config.masterSAEId}/enc_keys`;
      
      const payload = {
        number: request.number || 1,
        size: request.size || 256, // Default 256-bit keys
        ...request
      };

      logger.info('Requesting encryption keys', {
        slaveSAEId,
        count: payload.number,
        keySize: payload.size
      });

      const response = await this.client.post<ETSIGetKeyResponse>(
        endpoint,
        payload,
        {
          headers: {
            'slave_SAE_ID': slaveSAEId
          }
        }
      );

      const keys = response.data.keys;
      
      // Cache keys
      keys.forEach(key => this.cacheKey(key, 'encryption'));

      logger.info(`Retrieved ${keys.length} encryption keys`);
      return keys;
    } catch (error) {
      logger.error('Failed to get encryption keys', error);
      throw new Error(`Failed to retrieve encryption keys: ${error.message}`);
    }
  }

  /**
   * Get decryption key from KM
   * ETSI GS QKD 014: POST /api/v1/keys/{slave_SAE_ID}/dec_keys
   */
  async getDecryptionKey(keyId: string):

Promise<ETSIKey> {
try {
const endpoint = /api/v1/keys/${this.config.masterSAEId}/dec_keys;
  const payload = {
    key_ID: keyId
  };

  logger.info('Requesting decryption key', { keyId });

  const response = await this.client.post<ETSIDecKeyResponse>(
    endpoint,
    payload
  );

  const key = response.data.keys[0];
  
  if (!key) {
    throw new Error(`Decryption key not found for key_ID: ${keyId}`);
  }

  // Cache key
  this.cacheKey(key, 'decryption');

  logger.info('Retrieved decryption key', { keyId: key.key_ID });
  return key;
} catch (error) {
  logger.error('Failed to get decryption key', error);
  throw new Error(`Failed to retrieve decryption key: ${error.message}`);
}
}
/**

Cache quantum key
*/
private cacheKey(key: ETSIKey, purpose: 'encryption' | 'decryption'): void {
const cached: CachedKey = {
key: Buffer.from(key.key, 'base64'),
keyId: key.key_ID,
purpose,
retrievedAt: new Date(),
usageCount: 0,
maxUsage: purpose === 'encryption' ? 1 : Infinity, // OTP: use once for encryption
};

this.keyCache.set(key.key_ID, cached);
// Set expiration (keys should be used quickly)
setTimeout(() => {
  this.keyCache.delete(key.key_ID);
  logger.info('Key expired from cache', { keyId: key.key_ID });
}, 15 * 60 * 1000); // 15 minutes
}
/**

Get cached key
*/
getCachedKey(keyId: string): Buffer | null {
const cached = this.keyCache.get(keyId);

if (!cached) {
  return null;
}

// Check usage limit
if (cached.usageCount >= cached.maxUsage) {
  logger.warn('Key usage limit exceeded', { keyId });
  this.keyCache.delete(keyId);
  return null;
}

// Increment usage
cached.usageCount++;

return cached.key;
}
/**

Clear key cache (for security)
*/
clearKeyCache(): void {
this.keyCache.clear();
logger.info('Key cache cleared');
}

/**

List cached keys metadata
*/
listCachedKeys(): KeyMetadata[] {
return Array.from(this.keyCache.entries()).map(([keyId, cached]) => ({
keyId,
purpose: cached.purpose,
retrievedAt: cached.retrievedAt,
usageCount: cached.usageCount,
maxUsage: cached.maxUsage,
}));
}
}

interface CachedKey {
key: Buffer;
keyId: string;
purpose: 'encryption' | 'decryption';
retrievedAt: Date;
usageCount: number;
maxUsage: number;
}
export interface KeyMetadata {
keyId: string;
purpose: 'encryption' | 'decryption';
retrievedAt: Date;
usageCount: number;
maxUsage: number;
}

### **6.5 Key Lifecycle Workflow**
Encryption Workflow (Sender)
────────────────────────────
┌─────────────────┐
│ User composes   │
│ email           │
└────────┬────────┘
│
▼
┌─────────────────────────────┐
│ Select security level       │
│ (Quantum Secure = OTP)      │
└────────┬────────────────────┘
│
▼
┌──────────────────────────────┐
│ Request encryption key       │
│ from KM                      │
│ POST /api/v1/keys/.../enc_keys│
└────────┬─────────────────────┘
│
▼
┌──────────────────────────────┐
│ KM returns quantum key       │
│ key_ID: "qk-12345"           │
│ key: "base64-encoded-key"    │
└────────┬─────────────────────┘
│
▼
┌──────────────────────────────┐
│ Encrypt email with OTP       │
│ plaintext ⊕ quantum_key      │
└────────┬─────────────────────┘
│
▼
┌──────────────────────────────┐
│ Attach key_ID to email       │
│ X-QuMail-Key-ID: qk-12345    │
└────────┬─────────────────────┘
│
▼
┌──────────────────────────────┐
│ Send encrypted email via     │
│ SMTP                         │
└──────────────────────────────┘
Decryption Workflow (Recipient)
────────────────────────────────
┌──────────────────────────────┐
│ Receive encrypted email      │
│ via IMAP                     │
└────────┬─────────────────────┘
│
▼
┌──────────────────────────────┐
│ Extract key_ID from header   │
│ X-QuMail-Key-ID: qk-12345    │
└────────┬─────────────────────┘
│
▼
┌──────────────────────────────┐
│ Check key cache              │
└────────┬─────────────────────┘
│
├─ Found ──────────────┐
│                      │
│                      ▼
│              ┌────────────────┐
│              │ Use cached key │
│              └───────┬────────┘
│                      │
▼                      │
┌──────────────────────────────┐│
│ Request decryption key       ││
│ from KM                      ││
│ POST /api/v1/keys/.../dec_keys││
│ { key_ID: "qk-12345" }       ││
└────────┬─────────────────────┘│
│                      │
▼                      │
┌──────────────────────────────┐│
│ KM returns same quantum key  ││
│ (synchronized via QKD)       ││
└────────┬─────────────────────┘│
│                      │
└──────────┬───────────┘
▼
┌──────────────────────────────┐
│ Decrypt email with OTP       │
│ ciphertext ⊕ quantum_key     │
└────────┬─────────────────────┘
│
▼
┌──────────────────────────────┐
│ Display plaintext to user    │
└──────────────────────────────┘

### **6.6 Error Handling for KM Integration**
```typescript
// src/services/km-error-handler.ts

export class KMError extends Error {
  constructor(
    message: string,
    public code: KMErrorCode,
    public details?: any
  ) {
    super(message);
    this.name = 'KMError';
  }
}

export enum KMErrorCode {
  CONNECTION_FAILED = 'KM_CONNECTION_FAILED',
  AUTHENTICATION_FAILED = 'KM_AUTH_FAILED',
  KEY_NOT_AVAILABLE = 'KM_KEY_NOT_AVAILABLE',
  INSUFFICIENT_KEYS = 'KM_INSUFFICIENT_KEYS',
  INVALID_SAE_ID = 'KM_INVALID_SAE_ID',
  RATE_LIMIT_EXCEEDED = 'KM_RATE_LIMIT',
  CERTIFICATE_ERROR = 'KM_CERT_ERROR',
  TIMEOUT = 'KM_TIMEOUT',
}

export function handleKMError(error: any): KMError {
  if (error.response) {
    // HTTP error from KM
    const status = error.response.status;
    const data = error.response.data;

    switch (status) {
      case 401:
        return new KMError(
          'Authentication failed with Key Manager',
          KMErrorCode.AUTHENTICATION_FAILED,
          data
        );
      
      case 404:
        return new KMError(
          'Requested key not found',
          KMErrorCode.KEY_NOT_AVAILABLE,
          data
        );
      
      case 429:
        return new KMError(
          'Rate limit exceeded for Key Manager',
          KMErrorCode.RATE_LIMIT_EXCEEDED,
          data
        );
      
      case 503:
        return new KMError(
          'Insufficient quantum keys available',
          KMErrorCode.INSUFFICIENT_KEYS,
          data
        );
      
      default:
        return new KMError(
          `Key Manager error: ${data.message || 'Unknown error'}`,
          KMErrorCode.CONNECTION_FAILED,
          data
        );
    }
  } else if (error.code === 'ECONNREFUSED') {
    return new KMError(
      'Could not connect to Key Manager',
      KMErrorCode.CONNECTION_FAILED
    );
  } else if (error.code === 'ETIMEDOUT') {
    return new KMError(
      'Key Manager request timed out',
      KMErrorCode.TIMEOUT
    );
  } else if (error.code === 'CERT_HAS_EXPIRED') {
    return new KMError(
      'Key Manager certificate has expired',
      KMErrorCode.CERTIFICATE_ERROR
    );
  }

  return new KMError(
    error.message || 'Unknown Key Manager error',
    KMErrorCode.CONNECTION_FAILED
  );
}
```

---

## **7. Multi-Level Security Configuration**

### **7.1 Security Levels Overview**

QuMail provides four security levels to balance security needs with performance and compatibility:

| Level | Name | Encryption Method | Key Source | Security | Performance |
|-------|------|-------------------|------------|----------|-------------|
| **1** | Quantum Secure | One-Time Pad (OTP) | QKD | Unconditional | Slow |
| **2** | Quantum-Aided AES | AES-256-GCM | QKD (seed) | Very High | Fast |
| **3** | Post-Quantum | PQC (e.g., Kyber) | Generated | High | Medium |
| **4** | Standard | AES-256-GCM | Generated | Moderate | Fast |

### **7.2 Security Level Enumeration**
```typescript
// src/types/security-types.ts

export enum SecurityLevel {
  /**
   * Level 1: Quantum Secure (One-Time Pad)
   * - Uses quantum keys directly for XOR encryption
   * - Provides unconditional security (information-theoretic)
   * - Requires fresh quantum key per message
   * - Slowest but most secure
   */
  QUANTUM_SECURE = 'quantum-secure',

  /**
   * Level 2: Quantum-Aided AES
   * - Uses quantum key as seed for AES-256-GCM
   * - Fast symmetric encryption with quantum-enhanced key
   * - Provides very high security with good performance
   * - Recommended for most use cases
   */
  QUANTUM_AIDED_AES = 'quantum-aided',

  /**
   * Level 3: Post-Quantum Cryptography
   * - Uses lattice-based or other PQC algorithms (e.g., Kyber)
   * - Resistant to quantum computing attacks
   * - Does not require QKD infrastructure
   * - Good fallback when QKD is unavailable
   */
  POST_QUANTUM = 'post-quantum',

  /**
   * Level 4: Standard Encryption
   * - Traditional AES-256-GCM with RSA key exchange
   * - Compatible with non-QuMail recipients
   * - Vulnerable to future quantum attacks
   * - Use only for compatibility
   */
  STANDARD = 'standard'
}

export interface EncryptionMetadata {
  securityLevel: SecurityLevel;
  algorithm: string;
  keyId?: string;         // For quantum keys
  iv?: string;            // Initialization vector
  authTag?: string;       // For AEAD modes
  timestamp: Date;
  version: string;
}
```

### **7.3 Encryption Service Factory**
```typescript
// src/services/encryption/encryption-factory.ts

import { SecurityLevel } from '@/types/security-types';
import { OTPEncryptionService } from './otp-encryption-service';
import { QuantumAidedAESService } from './quantum-aided-aes-service';
import { PostQuantumService } from './post-quantum-service';
import { StandardEncryptionService } from './standard-encryption-service';
import { KeyManagementService } from '../key-management-service';

export interface IEncryptionService {
  encrypt(plaintext: Buffer, metadata: any): Promise<EncryptionResult>;
  decrypt(ciphertext: Buffer, metadata: EncryptionMetadata): Promise<Buffer>;
  getSecurityLevel(): SecurityLevel;
}

export interface EncryptionResult {
  ciphertext: Buffer;
  metadata: EncryptionMetadata;
}

export class EncryptionServiceFactory {
  constructor(private kmService: KeyManagementService) {}

  createService(level: SecurityLevel): IEncryptionService {
    switch (level) {
      case SecurityLevel.QUANTUM_SECURE:
        return new OTPEncryptionService(this.kmService);
      
      case SecurityLevel.QUANTUM_AIDED_AES:
        return new QuantumAidedAESService(this.kmService);
      
      case SecurityLevel.POST_QUANTUM:
        return new PostQuantumService();
      
      case SecurityLevel.STANDARD:
        return new StandardEncryptionService();
      
      default:
        throw new Error(`Unknown security level: ${level}`);
    }
  }
}
```

### **7.4 Level 1: Quantum Secure (One-Time Pad)**
```typescript
// src/services/encryption/otp-encryption-service.ts

import crypto from 'crypto';
import { IEncryptionService, EncryptionResult } from './encryption-factory';
import { SecurityLevel, EncryptionMetadata } from '@/types/security-types';
import { KeyManagementService } from '../key-management-service';
import { logger } from '@/utils/logger';

/**
 * One-Time Pad Encryption using Quantum Keys
 * 
 * Security: Information-theoretic (unconditional)
 * Algorithm: plaintext ⊕ quantum_key = ciphertext
 * Key Usage: Each key used exactly once
 */
export class OTPEncryptionService implements IEncryptionService {
  constructor(private kmService: KeyManagementService) {}

  getSecurityLevel(): SecurityLevel {
    return SecurityLevel.QUANTUM_SECURE;
  }

  /**
   * Encrypt data using One-Time Pad
   */
  async encrypt(
    plaintext: Buffer,
    context: { recipientSAEId: string }
  ): Promise<EncryptionResult> {
    try {
      const plaintextLength = plaintext.length;
      
      // Calculate required key size (must equal plaintext size for OTP)
      const keySizeBytes = plaintextLength;
      const keySizeBits = keySizeBytes * 8;

      logger.info('Requesting quantum key for OTP encryption', {
        plaintextSize: plaintextLength,
        keySize: keySizeBits
      });

      // Request quantum key from KM
      const keys = await this.kmService.getEncryptionKey(
        context.recipientSAEId,
        {
          number: 1,
          size: keySizeBits
        }
      );

      const quantumKey = Buffer.from(keys[0].key, 'base64');
      const keyId = keys[0].key_ID;

      // Verify key length matches plaintext length
      if (quantumKey.length < plaintextLength) {
        throw new Error(
          `Quantum key too short: got ${quantumKey.length}, need ${plaintextLength}`
        );
      }

      // Perform XOR encryption (OTP)
      const ciphertext = Buffer.alloc(plaintextLength);
      for (let i = 0; i < plaintextLength; i++) {
        ciphertext[i] = plaintext[i] ^ quantumKey[i];
      }

      const metadata: EncryptionMetadata = {
        securityLevel: SecurityLevel.QUANTUM_SECURE,
        algorithm: 'OTP',
        keyId: keyId,
        timestamp: new Date(),
        version: '1.0.0'
      };

      logger.info('OTP encryption successful', { keyId, size: plaintextLength });

      return {
        ciphertext,
        metadata
      };
    } catch (error) {
      logger.error('OTP encryption failed', error);
      throw error;
    }
  }

  /**
   * Decrypt data using One-Time Pad
   */
  async decrypt(
    ciphertext: Buffer,
    metadata: EncryptionMetadata
  ): Promise<Buffer> {
    try {
      if (!metadata.keyId) {
        throw new Error('Missing key ID in encryption metadata');
      }

      logger.info('Retrieving quantum key for OTP decryption', {
        keyId: metadata.keyId
      });

      // Try to get key from cache first
      let quantumKey = this.kmService.getCachedKey(metadata.keyId);

      // If not cached, request from KM
      if (!quantumKey) {
        const key = await this.kmService.getDecryptionKey(metadata.keyId);
        quantumKey = Buffer.from(key.key, 'base64');
      }

      // Verify key length
      if (quantumKey.length < ciphertext.length) {
        throw new Error('Quantum key too short for decryption');
      }

      // Perform XOR decryption (same operation as encryption)
      const plaintext = Buffer.alloc(ciphertext.length);
      for (let i = 0; i < ciphertext.length; i++) {
        plaintext[i] = ciphertext[i] ^ quantumKey[i];
      }

      logger.info('OTP decryption successful', { keyId: metadata.keyId });

      return plaintext;
    } catch (error) {
      logger.error('OTP decryption failed', error);
      throw error;
    }
  }
}
```

### **7.5 Level 2: Quantum-Aided AES**
```typescript
// src/services/encryption/quantum-aided-aes-service.ts

import crypto from 'crypto';
import { IEncryptionService, EncryptionResult } from './encryption-factory';
import { SecurityLevel, EncryptionMetadata } from '@/types/security-types';
import { KeyManagementService } from '../key-management-service';
import { logger } from '@/utils/logger';

/**
 * AES-256-GCM Encryption with Quantum-Derived Key
 * 
 * Security: Very high (quantum-enhanced symmetric encryption)
 * Algorithm: AES-256-GCM
 * Key Derivation: HKDF from quantum seed
 * Performance: Fast (hardware AES acceleration)
 */
export class QuantumAidedAESService implements IEncryptionService {
  private readonly ALGORITHM = 'aes-256-gcm';
  private readonly IV_LENGTH = 12; // 96 bits for GCM
  private readonly AUTH_TAG_LENGTH = 16; // 128 bits
  private readonly KEY_LENGTH = 32; // 256 bits

  constructor(private kmService: KeyManagementService) {}

  getSecurityLevel(): SecurityLevel {
    return SecurityLevel.QUANTUM_AIDED_AES;
  }

  /**
   * Encrypt data using Quantum-Aided AES
   */
  async encrypt(
    plaintext: Buffer,
    context: { recipientSAEId: string }
  ): Promise<EncryptionResult> {
    try {
      logger.info('Quantum-Aided AES encryption started');

      // Request quantum seed from KM (256 bits)
      const keys = await this.kmService.getEncryptionKey(
        context.recipientSAEId,
        {
          number: 1,
          size: 256 // 256-bit quantum seed
        }
      );

      const quantumSeed = Buffer.from(keys[0].key, 'base64');
      const keyId = keys[0].key_ID;

      // Derive AES key from quantum seed using HKDF
      const aesKey = this.deriveKey(quantumSeed);

      // Generate random IV
      const iv = crypto.randomBytes(this.IV_LENGTH);

      // Create cipher
      const cipher = crypto.createCipheriv(this.ALGORITHM, aesKey, iv);

      // Encrypt
      const encrypted = Buffer.concat([
        cipher.update(plaintext),
        cipher.final()
      ]);

      // Get authentication tag
      const authTag = cipher.getAuthTag();

      const metadata: EncryptionMetadata = {
        securityLevel: SecurityLevel.QUANTUM_AIDED_AES,
        algorithm: 'AES-256-GCM',
        keyId: keyId,
        iv: iv.toString('base64'),
        authTag: authTag.toString('base64'),
        timestamp: new Date(),
        version: '1.0.0'
      };

      logger.info('Quantum-Aided AES encryption successful', { keyId });

      return {
        ciphertext: encrypted,
        metadata
      };
    } catch (error) {
      logger.error('Quantum-Aided AES encryption failed', error);
      throw error;
    }
  }

  /**
   * Decrypt data using Quantum-Aided AES
   */
  async decrypt(
    ciphertext: Buffer,
    metadata: EncryptionMetadata
  ): Promise<Buffer> {
    try {
      if (!metadata.keyId || !metadata.iv || !metadata.authTag) {
        throw new Error('Missing required metadata for decryption');
      }

      logger.info('Quantum-Aided AES decryption started', {
        keyId: metadata.keyId
      });

      // Get quantum seed
      let quantumSeed = this.kmService.getCachedKey(metadata.keyId);

      if (!quantumSeed) {
        const key = await this.kmService.getDecryptionKey(metadata.keyId);
        quantumSeed = Buffer.from(key.key, 'base64');
      }

      // Derive same AES key
      const aesKey = this.deriveKey(quantumSeed);

      // Parse IV and auth tag
      const iv = Buffer.from(metadata.iv, 'base64');
      const authTag = Buffer.from(metadata.authTag, 'base64');

      // Create decipher
      const decipher = crypto.createDecipheriv(this.ALGORITHM, aesKey, iv);
      decipher.setAuthTag(authTag);

      // Decrypt
      const decrypted = Buffer.concat([
        decipher.update(ciphertext),
        decipher.final()
      ]);

      logger.info('Quantum-Aided AES decryption successful');

      return decrypted;
    } catch (error) {
      logger.error('Quantum-Aided AES decryption failed', error);
      throw error;
    }
  }

  /**
   * Derive AES key from quantum seed using HKDF
   */
  private deriveKey(quantumSeed: Buffer): Buffer {
    const salt = Buffer.from('QuMail-Quantum-Aided-AES');
    const info = Buffer.from('AES-256-Key-Derivation');

    // HKDF-SHA256
    return crypto.hkdfSync(
      'sha256',
      quantumSeed,
      salt,
      info,
      this.KEY_LENGTH
    );
  }
}
```

### **7.6 Level 3: Post-Quantum Cryptography**
```typescript
// src/services/encryption/post-quantum-service.ts

import { IEncryptionService, EncryptionResult } from './encryption-factory';
import { SecurityLevel, EncryptionMetadata } from '@/types/security-types';
import { logger } from '@/utils/logger';
// Note: In production, use a proper PQC library like liboqs or pqcrypto
import * as tweetnacl from 'tweetnacl';

/**
 * Post-Quantum Cryptography Encryption
 * 
 * Security: High (resistant to quantum attacks)
 * Algorithm: Placeholder (use Kyber, Dilithium, etc. in production)
 * Key Exchange: PQC key encapsulation mechanism
 * Fallback: When QKD infrastructure unavailable
 */
export class PostQuantumService implements IEncryptionService {
  getSecurityLevel(): SecurityLevel {
    return SecurityLevel.POST_QUANTUM;
  }

  async encrypt(
    plaintext: Buffer,
    context: any
  ): Promise<EncryptionResult> {
    try {
      logger.info('Post-Quantum encryption started');

      // Placeholder: In production, use actual PQC library (liboqs)
      // For now, using NaCl's secretbox as demonstration
      const key = tweetnacl.randomBytes(32);
      const nonce = tweetnacl.randomBytes(24);

      const encrypted = tweetnacl.secretbox(
        new Uint8Array(plaintext),
        nonce,
        key
      );

      const metadata: EncryptionMetadata = {
        securityLevel: SecurityLevel.POST_QUANTUM,
        algorithm: 'NaCl-Secretbox', // In production: 'Kyber-1024'
        iv: Buffer.from(nonce).toString('base64'),
        // In production, encapsulate key using recipient's PQC public key
        timestamp: new Date(),
        version: '1.0.0'
      };

      logger.info('Post-Quantum encryption successful');

      return {
        ciphertext: Buffer.from(encrypted),
        metadata
      };
    } catch (error) {
      logger.error('Post-Quantum encryption failed', error);
      throw error;
    }
  }

  async decrypt(
    ciphertext: Buffer,
    metadata: EncryptionMetadata
  ): Promise<Buffer> {
    try {
      logger.info('Post-Quantum decryption started');

      // Placeholder implementation
      // In production: use PQC key decapsulation + symmetric decryption

      logger.warn('Post-Quantum decryption - placeholder implementation');
      throw new Error('PQC decryption not fully implemented - use production PQC library');
    } catch (error) {
      logger.error('Post-Quantum decryption failed', error);
      throw error;
    }
  }
}
```

### **7.7 Security Level Selection Logic**
```typescript
// src/services/security-level-selector.ts

import { SecurityLevel } from '@/types/security-types';
import { KeyManagementService } from './key-management-service';
import { logger } from '@/utils/logger';

export class SecurityLevelSelector {
  constructor(private kmService: KeyManagementService) {}

  /**
   * Select appropriate security level based on context
   */
  async selectSecurityLevel(
    preferredLevel: SecurityLevel,
    context: {
      recipientHasQKD: boolean;
      messageSize: number;
      isUrgent: boolean;
    }
  ): Promise<SecurityLevel> {
    // If preferred level is Quantum Secure or Quantum-Aided
    if (
      preferredLevel === SecurityLevel.QUANTUM_SECURE ||
      preferredLevel === SecurityLevel.QUANTUM_AIDED_AES
    ) {
      // Check if recipient has QKD capability
      if (!context.recipientHasQKD) {
        logger.warn('Recipient does not support QKD, downgrading to PQC');
        return SecurityLevel.POST_QUANTUM;
      }

      // Check if KM is available
      try {
        await this.kmService.getStatus();
      } catch (error) {
        logger.warn('Key Manager unavailable, downgrading to PQC');
        return SecurityLevel.POST_QUANTUM;
      }

      // For OTP, check if message size is reasonable
      if (preferredLevel === SecurityLevel.QUANTUM_SECURE) {
        const MAX_OTP_SIZE = 10 * 1024 * 1024; // 10MB
        if (context.messageSize > MAX_OTP_SIZE) {
          logger.warn('Message too large for OTP, using Quantum-Aided AES');
          return SecurityLevel.QUANTUM_AIDED_AES;
        }
      }
    }

    return preferredLevel;
  }

  /**
   * Get recommended security level for recipient
   */
  getRecommendedLevel(recipientEmail: string): SecurityLevel {
    // Check if recipient is in QuMail network
    // For now, default to Quantum-Aided AES (best balance)
    return SecurityLevel.QUANTUM_AIDED_AES;
  }
}
```

---

## **8. Encryption Service Architecture**

### **8.1 Email Encryption Workflow**
┌─────────────────────────────────────────────────────────────┐
│           Email Encryption Pipeline                          │
├─────────────────────────────────────────────────────────────┤
│                                                               │
│  ┌────────────┐      ┌──────────────┐      ┌─────────────┐ │
│  │ Plaintext  │─────►│   Security   │─────►│  Encryption │ │
│  │   Email    │      │Level Selector│      │   Service   │ │
│  └────────────┘      └──────────────┘      └──────┬──────┘ │
│        │                                            │        │
│        │                                            ▼        │
│        │                                     ┌─────────────┐ │
│        │                                     │ Get Quantum │ │
│        │                                     │     Key     │ │
│        │                                     └──────┬──────┘ │
│        │                                            │        │
│        ▼                                            ▼        │
│  ┌────────────┐                              ┌─────────────┐ │
│  │ Attachments│──────────────────────────────►│   Encrypt   │ │
│  │  (separate)│     Parallel encryption      │ Attachments │ │
│  └────────────┘                              └──────┬──────┘ │
│                                                      │        │
│                                                      ▼        │
│                                              ┌───────────────┐│
│                                              │  Encrypted    ││
│                                              │    Email      ││
│                                              │ + Attachments ││
│                                              └───────┬───────┘│
│                                                      │        │
│                                                      ▼        │
│                                              ┌───────────────┐│
│                                              │  Add Metadata ││
│                                              │  Headers      ││
│                                              └───────┬───────┘│
│                                                      │        │
│                                                      ▼        │
│                                              ┌───────────────┐│
│                                              │  Send via     ││
│                                              │    SMTP       │
└───────────────┘│
│                                                               │
└─────────────────────────────────────────────────────────────┘

          └───────────────┘│
│                                                               │
└─────────────────────────────────────────────────────────────┘

### **8.2 Master Encryption Service**
```typescript
// src/services/encryption-service.ts

import { EncryptionServiceFactory, IEncryptionService } from './encryption/encryption-factory';
import { SecurityLevel, EncryptionMetadata } from '@/types/security-types';
import { KeyManagementService } from './key-management-service';
import { SecurityLevelSelector } from './security-level-selector';
import { logger } from '@/utils/logger';

export interface EmailEncryptionContext {
  recipientEmail: string;
  recipientSAEId?: string;
  securityLevel: SecurityLevel;
  messageSize: number;
}

export interface EncryptedEmail {
  encryptedBody: Buffer;
  encryptedAttachments: EncryptedAttachment[];
  metadata: EncryptionMetadata;
  headers: Record<string, string>;
}

export interface EncryptedAttachment {
  filename: string;
  encryptedData: Buffer;
  metadata: EncryptionMetadata;
  originalSize: number;
  encryptedSize: number;
}

export class EncryptionService {
  private factory: EncryptionServiceFactory;
  private levelSelector: SecurityLevelSelector;

  constructor(private kmService: KeyManagementService) {
    this.factory = new EncryptionServiceFactory(kmService);
    this.levelSelector = new SecurityLevelSelector(kmService);
  }

  /**
   * Encrypt complete email (body + attachments)
   */
  async encryptEmail(
    emailBody: string,
    attachments: Array<{ filename: string; data: Buffer }>,
    context: EmailEncryptionContext
  ): Promise<EncryptedEmail> {
    try {
      logger.info('Starting email encryption', {
        recipientEmail: context.recipientEmail,
        securityLevel: context.securityLevel,
        attachmentCount: attachments.length
      });

      // Select appropriate security level
      const selectedLevel = await this.levelSelector.selectSecurityLevel(
        context.securityLevel,
        {
          recipientHasQKD: !!context.recipientSAEId,
          messageSize: context.messageSize,
          isUrgent: false
        }
      );

      // Get encryption service for selected level
      const encryptionService = this.factory.createService(selectedLevel);

      // Encrypt email body
      const bodyBuffer = Buffer.from(emailBody, 'utf-8');
      const encryptedBodyResult = await encryptionService.encrypt(
        bodyBuffer,
        { recipientSAEId: context.recipientSAEId }
      );

      // Encrypt attachments in parallel
      const encryptedAttachments = await Promise.all(
        attachments.map(att => this.encryptAttachment(att, encryptionService, context))
      );

      // Prepare custom email headers
      const headers = this.buildEmailHeaders(encryptedBodyResult.metadata);

      logger.info('Email encryption completed successfully', {
        keyId: encryptedBodyResult.metadata.keyId,
        securityLevel: selectedLevel
      });

      return {
        encryptedBody: encryptedBodyResult.ciphertext,
        encryptedAttachments,
        metadata: encryptedBodyResult.metadata,
        headers
      };
    } catch (error) {
      logger.error('Email encryption failed', error);
      throw new Error(`Email encryption failed: ${error.message}`);
    }
  }

  /**
   * Decrypt complete email
   */
  async decryptEmail(
    encryptedBody: Buffer,
    encryptedAttachments: EncryptedAttachment[],
    metadata: EncryptionMetadata
  ): Promise<{ body: string; attachments: Array<{ filename: string; data: Buffer }> }> {
    try {
      logger.info('Starting email decryption', {
        securityLevel: metadata.securityLevel,
        keyId: metadata.keyId
      });

      // Get decryption service
      const decryptionService = this.factory.createService(metadata.securityLevel);

      // Decrypt body
      const decryptedBodyBuffer = await decryptionService.decrypt(
        encryptedBody,
        metadata
      );
      const body = decryptedBodyBuffer.toString('utf-8');

      // Decrypt attachments in parallel
      const attachments = await Promise.all(
        encryptedAttachments.map(async (att) => {
          const decryptionService = this.factory.createService(att.metadata.securityLevel);
          const decryptedData = await decryptionService.decrypt(
            att.encryptedData,
            att.metadata
          );
          
          return {
            filename: att.filename,
            data: decryptedData
          };
        })
      );

      logger.info('Email decryption completed successfully');

      return { body, attachments };
    } catch (error) {
      logger.error('Email decryption failed', error);
      throw new Error(`Email decryption failed: ${error.message}`);
    }
  }

  /**
   * Encrypt single attachment
   */
  private async encryptAttachment(
    attachment: { filename: string; data: Buffer },
    encryptionService: IEncryptionService,
    context: EmailEncryptionContext
  ): Promise<EncryptedAttachment> {
    const result = await encryptionService.encrypt(
      attachment.data,
      { recipientSAEId: context.recipientSAEId }
    );

    return {
      filename: attachment.filename,
      encryptedData: result.ciphertext,
      metadata: result.metadata,
      originalSize: attachment.data.length,
      encryptedSize: result.ciphertext.length
    };
  }

  /**
   * Build custom email headers for QuMail
   */
  private buildEmailHeaders(metadata: EncryptionMetadata): Record<string, string> {
    return {
      'X-QuMail-Version': metadata.version,
      'X-QuMail-Security-Level': metadata.securityLevel,
      'X-QuMail-Algorithm': metadata.algorithm,
      'X-QuMail-Key-ID': metadata.keyId || 'none',
      'X-QuMail-Encrypted': 'true',
      'X-QuMail-Timestamp': metadata.timestamp.toISOString()
    };
  }

  /**
   * Parse encryption metadata from email headers
   */
  parseEmailHeaders(headers: Record<string, string>): EncryptionMetadata | null {
    if (headers['X-QuMail-Encrypted'] !== 'true') {
      return null;
    }

    return {
      securityLevel: headers['X-QuMail-Security-Level'] as SecurityLevel,
      algorithm: headers['X-QuMail-Algorithm'],
      keyId: headers['X-QuMail-Key-ID'] !== 'none' ? headers['X-QuMail-Key-ID'] : undefined,
      timestamp: new Date(headers['X-QuMail-Timestamp']),
      version: headers['X-QuMail-Version']
    };
  }
}
```

---

## **9. Email Protocol Integration**

### **9.1 Email Service Architecture**
┌─────────────────────────────────────────────────────────┐
│                Email Service Layer                       │
├─────────────────────────────────────────────────────────┤
│                                                           │
│  ┌─────────────────┐          ┌─────────────────┐      │
│  │  SMTP Service   │          │  IMAP Service   │      │
│  │  (Send Email)   │          │ (Receive Email) │      │
│  └────────┬────────┘          └────────┬────────┘      │
│           │                             │               │
│           ▼                             ▼               │
│  ┌──────────────────────────────────────────────┐      │
│  │         nodemailer         │    imap-simple   │      │
│  └──────────────────────────────────────────────┘      │
│           │                             │               │
│           ▼                             ▼               │
│  ┌──────────────────────────────────────────────┐      │
│  │     Gmail / Yahoo / Outlook Servers           │      │
│  └──────────────────────────────────────────────┘      │
│                                                           │
└─────────────────────────────────────────────────────────┘

### **9.2 Email Account Configuration**
```typescript
// src/types/email-types.ts

export interface EmailAccount {
  id: string;
  email: string;
  displayName: string;
  provider: EmailProvider;
  imapConfig: IMAPConfiguration;
  smtpConfig: SMTPConfiguration;
  defaultSecurityLevel: SecurityLevel;
  saeId?: string; // For QKD-enabled accounts
  isActive: boolean;
  createdAt: Date;
  lastSyncedAt?: Date;
}

export enum EmailProvider {
  GMAIL = 'gmail',
  YAHOO = 'yahoo',
  OUTLOOK = 'outlook',
  CUSTOM = 'custom'
}

export interface IMAPConfiguration {
  host: string;
  port: number;
  secure: boolean; // Use TLS
  authMethod: 'password' | 'oauth2';
  tlsOptions?: {
    rejectUnauthorized: boolean;
    minVersion?: string;
  };
}

export interface SMTPConfiguration {
  host: string;
  port: number;
  secure: boolean;
  authMethod: 'password' | 'oauth2';
  tlsOptions?: {
    rejectUnauthorized: boolean;
    minVersion?: string;
  };
}

// Pre-configured settings for common providers
export const EMAIL_PROVIDER_CONFIGS: Record<EmailProvider, {
  imap: Partial<IMAPConfiguration>;
  smtp: Partial<SMTPConfiguration>;
}> = {
  [EmailProvider.GMAIL]: {
    imap: {
      host: 'imap.gmail.com',
      port: 993,
      secure: true
    },
    smtp: {
      host: 'smtp.gmail.com',
      port: 465,
      secure: true
    }
  },
  [EmailProvider.YAHOO]: {
    imap: {
      host: 'imap.mail.yahoo.com',
      port: 993,
      secure: true
    },
    smtp: {
      host: 'smtp.mail.yahoo.com',
      port: 465,
      secure: true
    }
  },
  [EmailProvider.OUTLOOK]: {
    imap: {
      host: 'outlook.office365.com',
      port: 993,
      secure: true
    },
    smtp: {
      host: 'smtp.office365.com',
      port: 587,
      secure: false // STARTTLS
    }
  },
  [EmailProvider.CUSTOM]: {
    imap: {},
    smtp: {}
  }
};
```

### **9.3 SMTP Service Implementation**
```typescript
// src/services/email/smtp-service.ts

import nodemailer from 'nodemailer';
import { EmailAccount, SMTPConfiguration } from '@/types/email-types';
import { CredentialStore } from '@/security/secure-storage';
import { logger } from '@/utils/logger';
import { EncryptedEmail } from '../encryption-service';

export interface SendEmailOptions {
  from: string;
  to: string[];
  cc?: string[];
  bcc?: string[];
  subject: string;
  body: string; // Plaintext or HTML
  attachments?: Array<{
    filename: string;
    path?: string;
    content?: Buffer;
  }>;
  headers?: Record<string, string>;
}

export class SMTPService {
  private credentialStore: CredentialStore;

  constructor() {
    this.credentialStore = new CredentialStore();
  }

  /**
   * Send encrypted email via SMTP
   */
  async sendEncryptedEmail(
    account: EmailAccount,
    options: SendEmailOptions,
    encryptedEmail: EncryptedEmail
  ): Promise<{ messageId: string }> {
    try {
      logger.info('Sending encrypted email via SMTP', {
        accountId: account.id,
        to: options.to,
        subject: options.subject
      });

      // Get account credentials
      const credentials = await this.credentialStore.getEmailCredentials(account.id);
      
      if (!credentials) {
        throw new Error('Email account credentials not found');
      }

      // Create transporter
      const transporter = await this.createTransporter(account, credentials.password);

      // Prepare email with encrypted content
      const mailOptions = {
        from: `"${account.displayName}" <${account.email}>`,
        to: options.to.join(', '),
        cc: options.cc?.join(', '),
        bcc: options.bcc?.join(', '),
        subject: options.subject,
        
        // Encrypted body as base64 in HTML
        html: this.buildEncryptedEmailHTML(encryptedEmail),
        
        // Add QuMail headers
        headers: {
          ...encryptedEmail.headers,
          ...options.headers
        },
        
        // Encrypted attachments
        attachments: encryptedEmail.encryptedAttachments.map(att => ({
          filename: `${att.filename}.encrypted`,
          content: att.encryptedData,
          contentType: 'application/octet-stream',
          headers: {
            'X-QuMail-Original-Filename': att.filename,
            'X-QuMail-Original-Size': att.originalSize.toString(),
            'X-QuMail-Key-ID': att.metadata.keyId || 'none'
          }
        }))
      };

      // Send email
      const info = await transporter.sendMail(mailOptions);

      logger.info('Email sent successfully', {
        messageId: info.messageId,
        response: info.response
      });

      return {
        messageId: info.messageId
      };
    } catch (error) {
      logger.error('Failed to send email via SMTP', error);
      throw new Error(`SMTP send failed: ${error.message}`);
    }
  }

  /**
   * Send regular (non-encrypted) email
   */
  async sendPlainEmail(
    account: EmailAccount,
    options: SendEmailOptions
  ): Promise<{ messageId: string }> {
    try {
      const credentials = await this.credentialStore.getEmailCredentials(account.id);
      const transporter = await this.createTransporter(account, credentials.password);

      const mailOptions = {
        from: `"${account.displayName}" <${account.email}>`,
        to: options.to.join(', '),
        cc: options.cc?.join(', '),
        bcc: options.bcc?.join(', '),
        subject: options.subject,
        text: options.body,
        html: options.body,
        attachments: options.attachments
      };

      const info = await transporter.sendMail(mailOptions);

      return {
        messageId: info.messageId
      };
    } catch (error) {
      logger.error('Failed to send plain email', error);
      throw error;
    }
  }

  /**
   * Create nodemailer transporter
   */
  private async createTransporter(
    account: EmailAccount,
    password: string
  ): Promise<nodemailer.Transporter> {
    const config = account.smtpConfig;

    const transportConfig: any = {
      host: config.host,
      port: config.port,
      secure: config.secure,
      auth: {
        user: account.email,
        pass: password
      },
      tls: config.tlsOptions || {
        rejectUnauthorized: true,
        minVersion: 'TLSv1.2'
      }
    };

    // OAuth2 support (for Gmail, etc.)
    if (config.authMethod === 'oauth2') {
      // Implement OAuth2 token refresh logic
      transportConfig.auth = {
        type: 'OAuth2',
        user: account.email,
        // Add OAuth2 tokens
      };
    }

    return nodemailer.createTransporter(transportConfig);
  }

  /**
   * Build HTML for encrypted email body
   */
  private buildEncryptedEmailHTML(encryptedEmail: EncryptedEmail): string {
    const encryptedBase64 = encryptedEmail.encryptedBody.toString('base64');
    
    return `
      <!DOCTYPE html>
      <html>
      <head>
        <meta charset="UTF-8">
        <title>Encrypted QuMail Message</title>
      </head>
      <body>
        <div style="font-family: Arial, sans-serif; padding: 20px; background: #f5f5f5;">
          <div style="background: white; padding: 30px; border-radius: 8px; max-width: 600px; margin: 0 auto;">
            <h2 style="color: #4A90E2;">🔐 Quantum-Secured Message</h2>
            <p>This message was encrypted using QuMail with <strong>${encryptedEmail.metadata.securityLevel}</strong> security.</p>
            <p>To read this message, please use QuMail application.</p>
            <div style="background: #f0f0f0; padding: 15px; border-radius: 4px; margin-top: 20px; font-family: monospace; word-break: break-all; font-size: 12px;">
              ${encryptedBase64.substring(0, 200)}...
            </div>
            <p style="margin-top: 20px; color: #666; font-size: 12px;">
              Security Level: ${encryptedEmail.metadata.securityLevel}<br>
              Algorithm: ${encryptedEmail.metadata.algorithm}<br>
              ${encryptedEmail.metadata.keyId ? `Key ID: ${encryptedEmail.metadata.keyId}` : ''}
            </p>
          </div>
        </div>
        <!-- Encrypted data for QuMail parser -->
        <div id="qumail-encrypted-data" style="display: none;">
          ${encryptedBase64}
        </div>
      </body>
      </html>
    `;
  }
}
```

### **9.4 IMAP Service Implementation**
```typescript
// src/services/email/imap-service.ts

import imapSimple from 'imap-simple';
import { simpleParser, ParsedMail } from 'mailparser';
import { EmailAccount } from '@/types/email-types';
import { CredentialStore } from '@/security/secure-storage';
import { logger } from '@/utils/logger';
import { EncryptionMetadata } from '@/types/security-types';

export interface FetchEmailsOptions {
  folder?: string;
  limit?: number;
  unreadOnly?: boolean;
  since?: Date;
}

export interface EmailMessage {
  id: string;
  uid: number;
  from: string;
  to: string[];
  cc?: string[];
  subject: string;
  date: Date;
  body: string;
  htmlBody?: string;
  attachments: EmailAttachment[];
  isEncrypted: boolean;
  encryptionMetadata?: EncryptionMetadata;
  headers: Record<string, string>;
  isRead: boolean;
  hasAttachments: boolean;
  size: number;
}

export interface EmailAttachment {
  filename: string;
  contentType: string;
  size: number;
  content: Buffer;
  isEncrypted: boolean;
}

export class IMAPService {
  private credentialStore: CredentialStore;
  private connections: Map<string, imapSimple.ImapSimple> = new Map();

  constructor() {
    this.credentialStore = new CredentialStore();
  }

  /**
   * Fetch emails from IMAP server
   */
  async fetchEmails(
    account: EmailAccount,
    options: FetchEmailsOptions = {}
  ): Promise<EmailMessage[]> {
    let connection: imapSimple.ImapSimple | null = null;

    try {
      logger.info('Fetching emails from IMAP', {
        accountId: account.id,
        folder: options.folder || 'INBOX'
      });

      // Get or create connection
      connection = await this.getConnection(account);

      // Open mailbox
      await connection.openBox(options.folder || 'INBOX');

      // Build search criteria
      const searchCriteria = this.buildSearchCriteria(options);

      // Fetch messages
      const messages = await connection.search(searchCriteria, {
        bodies: ['HEADER', 'TEXT', ''],
        markSeen: false
      });

      // Parse messages
      const parsedEmails = await Promise.all(
        messages.slice(0, options.limit || 50).map(msg => this.parseMessage(msg))
      );

      logger.info(`Fetched ${parsedEmails.length} emails`);

      return parsedEmails;
    } catch (error) {
      logger.error('Failed to fetch emails from IMAP', error);
      throw new Error(`IMAP fetch failed: ${error.message}`);
    }
  }

  /**
   * Fetch single email by UID
   */
  async fetchEmailByUID(
    account: EmailAccount,
    uid: number,
    folder: string = 'INBOX'
  ): Promise<EmailMessage | null> {
    let connection: imapSimple.ImapSimple | null = null;

    try {
      connection = await this.getConnection(account);
      await connection.openBox(folder);

      const messages = await connection.search([['UID', uid]], {
        bodies: ['HEADER', 'TEXT', ''],
        markSeen: false
      });

      if (messages.length === 0) {
        return null;
      }

      return await this.parseMessage(messages[0]);
    } catch (error) {
      logger.error('Failed to fetch email by UID', error);
      throw error;
    }
  }

  /**
   * Mark email as read
   */
  async markAsRead(
    account: EmailAccount,
    uid: number,
    folder: string = 'INBOX'
  ): Promise<void> {
    try {
      const connection = await this.getConnection(account);
      await connection.openBox(folder);
      await connection.addFlags([uid], ['\\Seen']);
      logger.info('Email marked as read', { uid });
    } catch (error) {
      logger.error('Failed to mark email as read', error);
      throw error;
    }
  }

  /**
   * Delete email
   */
  async deleteEmail(
    account: EmailAccount,
    uid: number,
    folder: string = 'INBOX'
  ): Promise<void> {
    try {
      const connection = await this.getConnection(account);
      await connection.openBox(folder);
      await connection.addFlags([uid], ['\\Deleted']);
      await connection.imap.expunge();
      logger.info('Email deleted', { uid });
    } catch (error) {
      logger.error('Failed to delete email', error);
      throw error;
    }
  }

  /**
   * Get or create IMAP connection
   */
  private async getConnection(account: EmailAccount): Promise<imapSimple.ImapSimple> {
    // Check if connection exists and is alive
    if (this.connections.has(account.id)) {
      const existing = this.connections.get(account.id)!;
      try {
        // Test connection
        await existing.getBoxes();
        return existing;
      } catch (error) {
        // Connection dead, remove it
        this.connections.delete(account.id);
      }
    }

    // Create new connection
    const credentials = await this.credentialStore.getEmailCredentials(account.id);
    
    if (!credentials) {
      throw new Error('Email credentials not found');
    }

    const config = {
      imap: {
        user: account.email,
        password: credentials.password,
        host: account.imapConfig.host,
        port: account.imapConfig.port,
        tls: account.imapConfig.secure,
        tlsOptions: account.imapConfig.tlsOptions || {
          rejectUnauthorized: true
        },
        authTimeout: 30000
      }
    };

    const connection = await imapSimple.connect(config);
    this.connections.set(account.id, connection);

    logger.info('IMAP connection established', { accountId: account.id });

    return connection;
  }

  /**
   * Parse IMAP message to EmailMessage
   */
  private async parseMessage(message: any): Promise<EmailMessage> {
    const all = message.parts.find((part: any) => part.which === '');
    const parsed: ParsedMail = await simpleParser(all.body);

    // Extract headers
    const headers: Record<string, string> = {};
    parsed.headers.forEach((value, key) => {
      headers[key] = Array.isArray(value) ? value.join(', ') : value as string;
    });

    // Check if encrypted
    const isEncrypted = headers['x-qumail-encrypted'] === 'true';
    let encryptionMetadata: EncryptionMetadata | undefined;

    if (isEncrypted) {
      encryptionMetadata = {
        securityLevel: headers['x-qumail-security-level'] as any,
        algorithm: headers['x-qumail-algorithm'],
        keyId: headers['x-qumail-key-id'] !== 'none' ? headers['x-qumail-key-id'] : undefined,
        timestamp: new Date(headers['x-qumail-timestamp']),
        version: headers['x-qumail-version']
      };
    }

    // Parse attachments
    const attachments: EmailAttachment[] = (parsed.attachments || []).map(att => ({
      filename: att.filename || 'unnamed',
      contentType: att.contentType,
      size: att.size,
      content: att.content,
      isEncrypted: att.filename?.endsWith('.encrypted') || false
    }));

    return {
      id: parsed.messageId || `uid-${message.attributes.uid}`,
      uid: message.attributes.uid,
      from: parsed.from?.text || '',
      to: (parsed.to?.value || []).map((addr: any) => addr.address),
      cc: (parsed.cc?.value || []).map((addr: any) => addr.address),
      subject: parsed.subject || '(No Subject)',
      date: parsed.date || new Date(),
      body: parsed.text || '',
      htmlBody: parsed.html || undefined,
      attachments,
      isEncrypted,
      encryptionMetadata,
      headers,
      isRead: message.attributes.flags.includes('\\Seen'),
      hasAttachments: attachments.length > 0,
      size: message.attributes.size || 0
    };
  }

  /**
   * Build IMAP search criteria
   */
  private buildSearchCriteria(options: FetchEmailsOptions): any[] {
    const criteria: any[] = ['ALL'];

    if (options.unreadOnly) {
      criteria.push(['UNSEEN']);
    }

    if (options.since) {
      criteria.push(['SINCE', options.since]);
    }

    return criteria;
  }

  /**
   * Close all IMAP connections
   */
  async closeAllConnections(): Promise<void> {
    for (const [accountId, connection] of this.connections.entries()) {
      try {
        await connection.end();
        logger.info('IMAP connection closed', { accountId });
      } catch (error) {
        logger.error('Error closing IMAP connection', error);
      }
    }
    this.connections.clear();
  }
}
```

---
10. Email Operations Architecture
    10.1 Unified Email Service
    typescript// src/services/email-service.ts

import { SMTPService, SendEmailOptions } from './email/smtp-service';
import { IMAPService, FetchEmailsOptions, EmailMessage } from './email/imap-service';
import { EncryptionService, EmailEncryptionContext } from './encryption-service';
import { EmailAccount } from '@/types/email-types';
import { SecurityLevel } from '@/types/security-types';
import { EmailRepository } from '@/repositories/email-repository';
import { logger } from '@/utils/logger';

export interface ComposeEmailRequest {
accountId: string;
to: string[];
cc?: string[];
bcc?: string[];
subject: string;
body: string;
attachments?: Array<{
filename: string;
path: string;
}>;
securityLevel: SecurityLevel;
recipientSAEId?: string;
}

export class EmailService {
private smtpService: SMTPService;
private imapService: IMAPService;
private encryptionService: EncryptionService;
private emailRepository: EmailRepository;

constructor(
smtpService: SMTPService,
imapService: IMAPService,
encryptionService: EncryptionService,
emailRepository: EmailRepository
) {
this.smtpService = smtpService;
this.imapService = imapService;
this.encryptionService = encryptionService;
this.emailRepository = emailRepository;
}

/**
* Send email with encryption
  */
  async sendEmail(request: ComposeEmailRequest): Promise<{
  messageId: string;
  encryptionUsed: boolean;
  keyId?: string;
  }> {
  try {
  logger.info('Sending email', {
  accountId: request.accountId,
  to: request.to,
  securityLevel: request.securityLevel
  });

  // Get account
  const account = await this.getAccount(request.accountId);

  // Read attachment files
  const attachments = await this.readAttachments(request.attachments || []);

  // Determine if encryption is needed
  const useEncryption = request.securityLevel !== SecurityLevel.STANDARD;

  let result;

  if (useEncryption) {
  // Encrypt email
  const encryptedEmail = await this.encryptionService.encryptEmail(
  request.body,
  attachments,
  {
  recipientEmail: request.to[0],
  recipientSAEId: request.recipientSAEId,
  securityLevel: request.securityLevel,
  messageSize: Buffer.from(request.body).length
  }
  );

  // Send encrypted email
  result = await this.smtpService.sendEncryptedEmail(
  account,
  {
  from: account.email,
  to: request.to,
  cc: request.cc,
  bcc: request.bcc,
  subject: request.subject,
  body: request.body,
  attachments: request.attachments
  },
  encryptedEmail
  );

  // Save to sent folder in database
  await this.emailRepository.saveSentEmail({
  accountId: request.accountId,
  messageId: result.messageId,
  to: request.to,
  subject: request.subject,
  body: request.body,
  encryptedBody: encryptedEmail.encryptedBody,
  encryptionMetadata: encryptedEmail.metadata,
  sentAt: new Date()
  });

  logger.info('Encrypted email sent successfully', {
  messageId: result.messageId,
  keyId: encryptedEmail.metadata.keyId
  });

  return {
  messageId: result.messageId,
  encryptionUsed: true,
  keyId: encryptedEmail.metadata.keyId
  };
  } else {
  // Send plain email
  result = await this.smtpService.sendPlainEmail(account, {
  from: account.email,
  to: request.to,
  cc: request.cc,
  bcc: request.bcc,
  subject: request.subject,
  body: request.body,
  attachments: request.attachments
  });

  await this.emailRepository.saveSentEmail({
  accountId: request.accountId,
  messageId: result.messageId,
  to: request.to,
  subject: request.subject,
  body: request.body,
  sentAt: new Date()
  });

  return {
  messageId: result.messageId,
  encryptionUsed: false
  };
  }
  } catch (error) {
  logger.error('Failed to send email', error);
  throw error;
  }
  }

/**
* Fetch emails from server and sync to local database
  */
  async syncEmails(accountId: string, options: FetchEmailsOptions = {}): Promise<{
  newCount: number;
  totalCount: number;
  }> {
  try {
  logger.info('Syncing emails', { accountId });

  const account = await this.getAccount(accountId);

  // Fetch emails from IMAP
  const emails = await this.imapService.fetchEmails(account, options);

  let newCount = 0;

  // Save to database
  for (const email of emails) {
  const exists = await this.emailRepository.emailExists(email.id);

  if (!exists) {
  // Decrypt if encrypted
  if (email.isEncrypted && email.encryptionMetadata) {
  try {
  const encryptedBody = this.extractEncryptedBody(email.htmlBody || email.body);
  const decrypted = await this.encryptionService.decryptEmail(
  encryptedBody,
  [],
  email.encryptionMetadata
  );
  email.body = decrypted.body;
  } catch (error) {
  logger.error('Failed to decrypt email', error);
  email.body = '[Decryption failed - invalid key or corrupted data]';
  }
  }

       await this.emailRepository.saveReceivedEmail({
         accountId,
         email
       });

       newCount++;
  }
  }

  // Update last synced timestamp
  await this.updateLastSynced(accountId);

  logger.info('Email sync completed', {
  accountId,
  newCount,
  totalCount: emails.length
  });

  return {
  newCount,
  totalCount: emails.length
  };
  } catch (error) {
  logger.error('Email sync failed', error);
  throw error;
  }
  }

/**
* Get email by ID from local database
  */
  async getEmailById(emailId: string): Promise<EmailMessage | null> {
  return await this.emailRepository.getEmailById(emailId);
  }

/**
* Get emails for account
  */
  async getEmailsForAccount(
  accountId: string,
  options: {
  folder?: string;
  limit?: number;
  offset?: number;
  unreadOnly?: boolean;
  } = {}
  ): Promise<{ emails: EmailMessage[]; total: number }> {
  return await this.emailRepository.getEmailsForAccount(accountId, options);
  }

/**
* Mark email as read
  */
  async markEmailAsRead(emailId: string, read: boolean = true): Promise<void> {
  const email = await this.emailRepository.getEmailById(emailId);

    if (!email) {
      throw new Error('Email not found');
    }

    // Update local database
    await this.emailRepository.updateEmailReadStatus(emailId, read);

    // Update on IMAP server
    const account = await this.getAccount(email.accountId);
    if (read) {
      await this.imapService.markAsRead(account, email.uid);
    }
}

/**
* Delete email
  */
  async deleteEmail(emailId: string): Promise<void> {
  const email = await this.emailRepository.getEmailById(emailId);

    if (!email) {
      throw new Error('Email not found');
    }

    // Delete from IMAP server
    const account = await this.getAccount(email.accountId);
    await this.imapService.deleteEmail(account, email.uid);

    // Delete from local database
    await this.emailRepository.deleteEmail(emailId);

    logger.info('Email deleted', { emailId });
}

/**
* Search emails
  */
  async searchEmails(
  accountId: string,
  query: string
  ): Promise<EmailMessage[]> {
  return await this.emailRepository.searchEmails(accountId, query);
  }

// Helper methods

private async getAccount(accountId: string): Promise<EmailAccount> {
// Implementation would fetch from AccountRepository
throw new Error('Not implemented');
}

private async readAttachments(
attachmentPaths: Array<{ filename: string; path: string }>
): Promise<Array<{ filename: string; data: Buffer }>> {
const fs = require('fs').promises;
return await Promise.all(
attachmentPaths.map(async (att) => ({
filename: att.filename,
data: await fs.readFile(att.path)
}))
);
}

private extractEncryptedBody(html: string): Buffer {
// Extract base64 encrypted data from HTML
const match = html.match(/<div id="qumail-encrypted-data"[^>]*>(.*?)<\/div>/s);
if (match && match[1]) {
return Buffer.from(match[1].trim(), 'base64');
}
throw new Error('Could not extract encrypted data from email');
}

private async updateLastSynced(accountId: string): Promise<void> {
// Update account's lastSyncedAt timestamp
// Implementation in AccountRepository
}
}
10.2 Email Draft Management
typescript// src/services/draft-service.ts

import { EmailRepository } from '@/repositories/email-repository';
import { logger } from '@/utils/logger';

export interface EmailDraft {
id: string;
accountId: string;
to: string[];
cc?: string[];
bcc?: string[];
subject: string;
body: string;
attachments: Array<{
filename: string;
path: string;
}>;
securityLevel: SecurityLevel;
createdAt: Date;
updatedAt: Date;
}

export class DraftService {
constructor(private emailRepository: EmailRepository) {}

/**
* Save email draft
  */
  async saveDraft(draft: Omit<EmailDraft, 'id' | 'createdAt' | 'updatedAt'>): Promise<EmailDraft> {
  logger.info('Saving email draft', { accountId: draft.accountId });

    const savedDraft = await this.emailRepository.saveDraft({
      ...draft,
      id: this.generateDraftId(),
      createdAt: new Date(),
      updatedAt: new Date()
    });

    return savedDraft;
}

/**
* Update existing draft
  */
  async updateDraft(draftId: string, updates: Partial<EmailDraft>): Promise<EmailDraft> {
  const existingDraft = await this.emailRepository.getDraftById(draftId);

    if (!existingDraft) {
      throw new Error('Draft not found');
    }

    const updatedDraft = {
      ...existingDraft,
      ...updates,
      updatedAt: new Date()
    };

    await this.emailRepository.updateDraft(draftId, updatedDraft);

    logger.info('Draft updated', { draftId });

    return updatedDraft;
}

/**
* Get all drafts for account
  */
  async getDraftsForAccount(accountId: string): Promise<EmailDraft[]> {
  return await this.emailRepository.getDraftsForAccount(accountId);
  }

/**
* Delete draft
  */
  async deleteDraft(draftId: string): Promise<void> {
  await this.emailRepository.deleteDraft(draftId);
  logger.info('Draft deleted', { draftId });
  }

private generateDraftId(): string {
return `draft-${Date.now()}-${Math.random().toString(36).substr(2, 9)}`;
}
}
10.3 Email Attachment Handler
typescript// src/services/attachment-handler.ts

import { app } from 'electron';
import path from 'path';
import fs from 'fs/promises';
import crypto from 'crypto';
import { logger } from '@/utils/logger';

export class AttachmentHandler {
private attachmentDir: string;

constructor() {
this.attachmentDir = path.join(app.getPath('userData'), 'attachments');
this.ensureAttachmentDir();
}

private async ensureAttachmentDir(): Promise<void> {
try {
await fs.mkdir(this.attachmentDir, { recursive: true });
} catch (error) {
logger.error('Failed to create attachment directory', error);
}
}

/**
* Save attachment to disk
  */
  async saveAttachment(
  emailId: string,
  filename: string,
  data: Buffer
  ): Promise<string> {
  try {
  // Sanitize filename
  const safeName = this.sanitizeFilename(filename);

  // Create email-specific subdirectory
  const emailDir = path.join(this.attachmentDir, emailId);
  await fs.mkdir(emailDir, { recursive: true });

  // Generate unique filename if needed
  let finalPath = path.join(emailDir, safeName);
  let counter = 1;

  while (await this.fileExists(finalPath)) {
  const ext = path.extname(safeName);
  const base = path.basename(safeName, ext);
  finalPath = path.join(emailDir, `${base}_${counter}${ext}`);
  counter++;
  }

  // Write file
  await fs.writeFile(finalPath, data);

  logger.info('Attachment saved', {
  emailId,
  filename: safeName,
  path: finalPath,
  size: data.length
  });

  return finalPath;
  } catch (error) {
  logger.error('Failed to save attachment', error);
  throw error;
  }
  }

/**
* Get attachment from disk
  */
  async getAttachment(filepath: string): Promise<Buffer> {
  try {
  // Verify path is within attachments directory
  const normalizedPath = path.normalize(filepath);
  if (!normalizedPath.startsWith(this.attachmentDir)) {
  throw new Error('Invalid attachment path');
  }

  return await fs.readFile(filepath);
  } catch (error) {
  logger.error('Failed to read attachment', error);
  throw error;
  }
  }

/**
* Delete attachment
  */
  async deleteAttachment(filepath: string): Promise<void> {
  try {
  const normalizedPath = path.normalize(filepath);
  if (!normalizedPath.startsWith(this.attachmentDir)) {
  throw new Error('Invalid attachment path');
  }

  await fs.unlink(filepath);
  logger.info('Attachment deleted', { filepath });
  } catch (error) {
  logger.error('Failed to delete attachment', error);
  throw error;
  }
  }

/**
* Delete all attachments for email
  */
  async deleteEmailAttachments(emailId: string): Promise<void> {
  try {
  const emailDir = path.join(this.attachmentDir, emailId);
  await fs.rm(emailDir, { recursive: true, force: true });
  logger.info('All attachments deleted for email', { emailId });
  } catch (error) {
  logger.error('Failed to delete email attachments', error);
  }
  }

/**
* Get attachment info
  */
  async getAttachmentInfo(filepath: string): Promise<{
  filename: string;
  size: number;
  contentType: string;
  }> {
  const stats = await fs.stat(filepath);
  const filename = path.basename(filepath);
  const contentType = this.guessContentType(filename);

    return {
      filename,
      size: stats.size,
      contentType
    };
}

private sanitizeFilename(filename: string): string {
// Remove directory traversal attempts and invalid characters
return filename
.replace(/[\/\\]/g, '')
.replace(/[<>:"|?*]/g, '_')
.substring(0, 255);
}

private async fileExists(filepath: string): Promise<boolean> {
try {
await fs.access(filepath);
return true;
} catch {
return false;
}
}

private guessContentType(filename: string): string {
const ext = path.extname(filename).toLowerCase();
const mimeTypes: Record<string, string> = {
'.pdf': 'application/pdf',
'.doc': 'application/msword',
'.docx': 'application/vnd.openxmlformats-officedocument.wordprocessingml.document',
'.xls': 'application/vnd.ms-excel',
'.xlsx': 'application/vnd.openxmlformats-officedocument.spreadsheetml.sheet',
'.png': 'image/png',
'.jpg': 'image/jpeg',
'.jpeg': 'image/jpeg',
'.gif': 'image/gif',
'.txt': 'text/plain',
'.zip': 'application/zip',
};

    return mimeTypes[ext] || 'application/octet-stream';
}

/**
* Calculate file hash (for deduplication)
  */
  async calculateHash(filepath: string): Promise<string> {
  const data = await fs.readFile(filepath);
  return crypto.createHash('sha256').update(data).digest('hex');
  }
  }

11. Local Storage & Database Schema
    11.1 Database Schema Design
    typescript// src/database/schema.ts

import Database from 'better-sqlite3';
import { app } from 'electron';
import path from 'path';
import { logger } from '@/utils/logger';

export class DatabaseSchema {
private db: Database.Database;

constructor(db: Database.Database) {
this.db = db;
}

/**
* Initialize database schema
  */
  initializeSchema(): void {
  logger.info('Initializing database schema');

    // Enable foreign keys
    this.db.pragma('foreign_keys = ON');

    // Create tables
    this.createAccountsTable();
    this.createEmailsTable();
    this.createAttachmentsTable();
    this.createDraftsTable();
    this.createQuantumKeysTable();
    this.createConfigTable();
    this.createSyncStatusTable();

    logger.info('Database schema initialized');
}

/**
* Email Accounts Table
  */
  private createAccountsTable(): void {
  this.db.exec(`
  CREATE TABLE IF NOT EXISTS accounts (
  id TEXT PRIMARY KEY,
  email TEXT NOT NULL UNIQUE,
  display_name TEXT NOT NULL,
  provider TEXT NOT NULL,
  imap_host TEXT NOT NULL,
  imap_port INTEGER NOT NULL,
  imap_secure INTEGER NOT NULL DEFAULT 1,
  smtp_host TEXT NOT NULL,
  smtp_port INTEGER NOT NULL,
  smtp_secure INTEGER NOT NULL DEFAULT 1,
  auth_method TEXT NOT NULL DEFAULT 'password',
  default_security_level TEXT NOT NULL,
  sae_id TEXT,
  is_active INTEGER NOT NULL DEFAULT 1,
  created_at TEXT NOT NULL,
  last_synced_at TEXT,
  CONSTRAINT valid_provider CHECK (provider IN ('gmail', 'yahoo', 'outlook', 'custom')),
  CONSTRAINT valid_auth CHECK (auth_method IN ('password', 'oauth2')),
  CONSTRAINT valid_security CHECK (default_security_level IN (
  'quantum-secure', 'quantum-aided', 'post-quantum', 'standard'
  ))
  );

  CREATE INDEX IF NOT EXISTS idx_accounts_email ON accounts(email);
  CREATE INDEX IF NOT EXISTS idx_accounts_active ON accounts(is_active);
  `);
  }

/**
* Emails Table
  */
  private createEmailsTable(): void {
  this.db.exec(`
  CREATE TABLE IF NOT EXISTS emails (
  id TEXT PRIMARY KEY,
  account_id TEXT NOT NULL,
  message_id TEXT NOT NULL,
  uid INTEGER,
  folder TEXT NOT NULL DEFAULT 'INBOX',
  from_address TEXT NOT NULL,
  to_addresses TEXT NOT NULL,
  cc_addresses TEXT,
  bcc_addresses TEXT,
  subject TEXT NOT NULL,
  body_plain TEXT,
  body_html TEXT,
  encrypted_body BLOB,
  is_encrypted INTEGER NOT NULL DEFAULT 0,
  security_level TEXT,
  encryption_algorithm TEXT,
  key_id TEXT,
  encryption_iv TEXT,
  encryption_auth_tag TEXT,
  encryption_version TEXT,
  is_read INTEGER NOT NULL DEFAULT 0,
  is_starred INTEGER NOT NULL DEFAULT 0,
  has_attachments INTEGER NOT NULL DEFAULT 0,
  size INTEGER NOT NULL DEFAULT 0,
  received_at TEXT NOT NULL,
  sent_at TEXT,
  created_at TEXT NOT NULL,
  FOREIGN KEY (account_id) REFERENCES accounts(id) ON DELETE CASCADE,
  FOREIGN KEY (key_id) REFERENCES quantum_keys(key_id) ON DELETE SET NULL
  );

  CREATE INDEX IF NOT EXISTS idx_emails_account ON emails(account_id);
  CREATE INDEX IF NOT EXISTS idx_emails_folder ON emails(folder);
  CREATE INDEX IF NOT EXISTS idx_emails_read ON emails(is_read);
  CREATE INDEX IF NOT EXISTS idx_emails_received ON emails(received_at DESC);
  CREATE INDEX IF NOT EXISTS idx_emails_message_id ON emails(message_id);
  CREATE INDEX IF NOT EXISTS idx_emails_search ON emails(subject, from_address);

  -- Full-text search
  CREATE VIRTUAL TABLE IF NOT EXISTS emails_fts USING fts5(
  subject, body_plain, from_address, to_addresses,
  content=emails,
  content_rowid=rowid
  );

  -- Triggers to keep FTS in sync
  CREATE TRIGGER IF NOT EXISTS emails_ai AFTER INSERT ON emails BEGIN
  INSERT INTO emails_fts(rowid, subject, body_plain, from_address, to_addresses)
  VALUES (new.rowid, new.subject, new.body_plain, new.from_address, new.to_addresses);
  END;

  CREATE TRIGGER IF NOT EXISTS emails_ad AFTER DELETE ON emails BEGIN
  DELETE FROM emails_fts WHERE rowid = old.rowid;
  END;

  CREATE TRIGGER IF NOT EXISTS emails_au AFTER UPDATE ON emails BEGIN
  DELETE FROM emails_fts WHERE rowid = old.rowid;
  INSERT INTO emails_fts(rowid, subject, body_plain, from_address, to_addresses)
  VALUES (new.rowid, new.subject, new.body_plain, new.from_address, new.to_addresses);
  END;
  `);
  }

/**
* Attachments Table
  */
  private createAttachmentsTable(): void {
  this.db.exec(`
  CREATE TABLE IF NOT EXISTS attachments (
  id TEXT PRIMARY KEY,
  email_id TEXT NOT NULL,
  filename TEXT NOT NULL,
  original_filename TEXT,
  content_type TEXT NOT NULL,
  size INTEGER NOT NULL,
  encrypted_size INTEGER,
  is_encrypted INTEGER NOT NULL DEFAULT 0,
  key_id TEXT,
  file_path TEXT NOT NULL,
  file_hash TEXT,
  created_at TEXT NOT NULL,
  FOREIGN KEY (email_id) REFERENCES emails(id) ON DELETE CASCADE,
  FOREIGN KEY (key_id) REFERENCES quantum_keys(key_id) ON DELETE SET NULL
  );

  CREATE INDEX IF NOT EXISTS idx_attachments_email ON attachments(email_id);
  CREATE INDEX IF NOT EXISTS idx_attachments_hash ON attachments(file_hash);
  `);
  }

/**
* Email Drafts Table
  */
  private createDraftsTable(): void {
  this.db.exec(`
  CREATE TABLE IF NOT EXISTS drafts (
  id TEXT PRIMARY KEY,
  account_id TEXT NOT NULL,
  to_addresses TEXT NOT NULL,
  cc_addresses TEXT,
  bcc_addresses TEXT,
  subject TEXT NOT NULL,
  body TEXT NOT NULL,
  attachments_json TEXT,
  security_level TEXT NOT NULL,
  recipient_sae_id TEXT,
  created_at TEXT NOT NULL,
  updated_at TEXT NOT NULL,
  FOREIGN KEY (account_id) REFERENCES accounts(id) ON DELETE CASCADE
  );

  CREATE INDEX IF NOT EXISTS idx_drafts_account ON drafts(account_id);
  CREATE INDEX IF NOT EXISTS idx_drafts_updated ON drafts(updated_at DESC);
  `);
  }

/**
* Quantum Keys Cache Table
  */
  private createQuantumKeysTable(): void {
  this.db.exec(`
  CREATE TABLE IF NOT EXISTS quantum_keys (
  key_id TEXT PRIMARY KEY,
  key_data BLOB NOT NULL,
  purpose TEXT NOT NULL,
  algorithm TEXT NOT NULL,
  size_bits INTEGER NOT NULL,
  usage_count INTEGER NOT NULL DEFAULT 0,
  max_usage INTEGER NOT NULL DEFAULT 1,
  retrieved_at TEXT NOT NULL,
  expires_at TEXT,
  source_sae_id TEXT,
  target_sae_id TEXT,
  CONSTRAINT valid_purpose CHECK (purpose IN ('encryption', 'decryption'))
  );

  CREATE INDEX IF NOT EXISTS idx_qkeys_expires ON quantum_keys(expires_at);
  CREATE INDEX IF NOT EXISTS idx_qkeys_usage ON quantum_keys(usage_count, max_usage);
  `);
  }

/**
* Configuration Table
  */
  private createConfigTable(): void {
  this.db.exec(`
  CREATE TABLE IF NOT EXISTS config (
  key TEXT PRIMARY KEY,
  value TEXT NOT NULL,
  type TEXT NOT NULL DEFAULT 'string',
  updated_at TEXT NOT NULL,
  CONSTRAINT valid_type CHECK (type IN ('string', 'number', 'boolean', 'json'))
  );

  -- Insert default config
  INSERT OR IGNORE INTO config (key, value, type, updated_at) VALUES
  ('app_version', '1.0.0', 'string', datetime('now')),
  ('auto_sync_enabled', 'true', 'boolean', datetime('now')),
  ('sync_interval_minutes', '5', 'number', datetime('now')),
  ('notification_enabled', 'true', 'boolean', datetime('now'));
  `);
  }

/**
* Sync Status Table
  */
  private createSyncStatusTable(): void {
  this.db.exec(`
      CREATE TABLE IF NOT EXISTS sync_status (
        account_id TEXT PRIMARY KEY,
        last_sync_started TEXT,
        last_sync_completed TEXT,
        last_sync_error TEXT,
        emails_fetched INTEGER DEFAULT 0,
        is_syncing INTEGER DEFAULT 0,
        FOREIGN KEY (account_id) REFERENCES accounts(id) ON DELETE CASCADE
      );
    `);
  }

/**
* Run database migrations
  */
  runMigrations(): void {
  const currentVersion = this.getCurrentSchemaVersion();
  logger.info(`Current database schema version: ${currentVersion}`);

    // Run migrations based on version
    if (currentVersion < 2) {
      this.migrateToV2();
    }
    // Add more migrations as needed
}

private getCurrentSchemaVersion(): number {
try {
const result = this.db.prepare('SELECT value FROM config WHERE key = ?').get('schema_version') as any;
return result ? parseInt(result.value) : 1;
} catch {
return 1;
}
}

private migrateToV2(): void {
logger.info('Migrating database to version 2');

    // Example migration
    this.db.exec(`
      ALTER TABLE emails ADD COLUMN is_starred INTEGER NOT NULL DEFAULT 0;
    `);

    this.db.prepare('INSERT OR REPLACE INTO config (key, value, type, updated_at) VALUES (?, ?, ?, datetime(\'now\'))')
      .run('schema_version', '2', 'number');

    logger.info('Migration to version 2 completed');
}
}
11.2 Database Manager
typescript// src/database/database-manager.ts

import Database from 'better-sqlite3';
import { app } from 'electron';
import path from 'path';
import { DatabaseSchema } from './schema';
import { logger } from '@/utils/logger';

export class DatabaseManager {
private static instance: DatabaseManager;
private db: Database.Database;

private constructor() {
const dbPath = path.join(app.getPath('userData'), 'qumail.db');

    logger.info('Opening database', { path: dbPath });
    
    this.db = new Database(dbPath);
    
    // Performance optimizations
    this.db.pragma('journal_mode = WAL');
    this.db.pragma('synchronous = NORMAL');
    this.db.pragma('cache_size = -64000'); // 64MB cache
    this.db.pragma('temp_store = MEMORY');
    
    // Initialize schema
    const schema = new DatabaseSchema(this.db);
    schema.initializeSchema();
    schema.runMigrations();
}

static getInstance(): DatabaseManager {
if (!DatabaseManager.instance) {
DatabaseManager.instance = new DatabaseManager();
}
return DatabaseManager.instance;
}

getDatabase(): Database.Database {
return this.db;
}

/**
* Execute query with automatic retry
  */
  executeWithRetry<T>(
  operation: () => T,
  maxRetries: number = 3
  ): T {
  let lastError: Error | undefined;

    for (let attempt = 1; attempt <= maxRetries; attempt++) {
      try {
        return operation();
      } catch (error) {
        lastError = error as Error;
        
        if (error.message.includes('SQLITE_BUSY')) {
          logger.warn(`Database busy, retrying (${attempt}/${maxRetries})`);
          // Wait before retry
          const delay = attempt * 100;
          this.sleep(delay);
          continue;
        }
        
        throw error;
      }
    }
    
    throw lastError;
}

/**
* Backup database
  */
  async backup(backupPath?: string): Promise<string> {
  const targetPath = backupPath || path.join(
  app.getPath('userData'),
  'backups',
  `qumail-backup-${Date.now()}.db`
  );

    await this.db.backup(targetPath);
    logger.info('Database backup created', { path: targetPath });
    
    return targetPath;
}

/**
* Vacuum database (cleanup and optimize)
  */
  vacuum(): void {
  logger.info('Vacuuming database');
  this.db.exec('VACUUM;');
  logger.info('Database vacuumed');
  }

/**
* Close database connection
  */
  close(): void {
  if (this.db) {
  this.db.close();
  logger.info('Database closed');
  }
  }

private sleep(ms: number): void {
const start = Date.now();
while (Date.now() - start < ms) {
// Busy wait
}
}
}

12. Service-Oriented Architecture
    12.1 Dependency Injection Container
    typescript// src/core/dependency-container.ts

import { KeyManagementService } from '@/services/key-management-service';
import { EncryptionService } from '@/services/encryption-service';
import { SMTPService } from '@/services/email/smtp-service';
import { IMAPService } from '@/services/email/imap-service';
import { EmailService } from '@/services/email-service';
import { AccountService } from '@/services/account-service';
import { ConfigurationService } from '@/services/configuration-service';
import { DatabaseManager } from '@/database/database-manager';
import { EmailRepository } from '@/repositories/email-repository';
import { AccountRepository } from '@/repositories/account-repository';
import { logger } from '@/utils/logger';
/**

Dependency Injection Container
Manages service lifecycle and dependencies
*/
export class DependencyContainer {
private static instance: DependencyContainer;
private services: Map<string, any> = new Map();

private constructor() {
this.registerServices();
}
static getInstance(): DependencyContainer {
if (!DependencyContainer.instance) {
DependencyContainer.instance = new DependencyContainer();
}
return DependencyContainer.instance;
}
/**

Register all services
*/
private registerServices(): void {
logger.info('Registering services');

// Database
const dbManager = DatabaseManager.getInstance();
this.services.set('DatabaseManager', dbManager);

// Repositories
const emailRepository = new EmailRepository(dbManager.getDatabase());
const accountRepository = new AccountRepository(dbManager.getDatabase());

this.services.set('EmailRepository', emailRepository);
this.services.set('AccountRepository', accountRepository);

// Configuration Service
const configService = new ConfigurationService();
this.services.set('ConfigurationService', configService);

// Key Management Service
const kmConfig = configService.getKMConfig();
const kmService = new KeyManagementService(kmConfig);
this.services.set('KeyManagementService', kmService);

// Encryption Service
const encryptionService = new EncryptionService(kmService);
this.services.set('EncryptionService', encryptionService);

// Email Services
const smtpService = new SMTPService();
const imapService = new IMAPService();

this.services.set('SMTPService', smtpService);
this.services.set('IMAPService', imapService);

// Unified Email Service
const emailService = new EmailService(
smtpService,
imapService,
encryptionService,
emailRepository
);
this.services.set('EmailService', emailService);

// Account Service
const accountService = new AccountService(accountRepository);
this.services.set('AccountService', accountService);

logger.info('All services registered');
}
/**

Get service by name
*/
get<T>(serviceName: string): T {
const service = this.services.get(serviceName);

if (!service) {
throw new Error(`Service not found: ${serviceName}`);
}

return service as T;
}
/**

Initialize all services
*/
async initialize(): Promise<void> {
logger.info('Initializing services');

// Initialize services that need async setup
const kmService = this.get<KeyManagementService>('KeyManagementService');

try {
await kmService.authenticate();
} catch (error) {
logger.warn('KM authentication failed during initialization', error);
// Continue - KM might not be available at startup
}

logger.info('Services initialized');
}
/**

Cleanup all services
*/
async cleanup(): Promise<void> {
logger.info('Cleaning up services');

// Close database
const dbManager = this.get<DatabaseManager>('DatabaseManager');
dbManager.close();

// Close IMAP connections
const imapService = this.get<IMAPService>('IMAPService');
await imapService.closeAllConnections();

// Clear key cache
const kmService = this.get<KeyManagementService>('KeyManagementService');
kmService.clearKeyCache();

logger.info('Services cleaned up');
}
}

### **12.2 Service Layer Diagram**
┌──────────────────────────────────────────────────────────────┐
│                    Service Architecture                       │
├──────────────────────────────────────────────────────────────┤
│                                                                │
│  ┌────────────────────────────────────────────────────────┐  │
│  │              IPC Handler Layer                         │  │
│  │  (Exposed to Renderer via Context Bridge)             │  │
│  └────────────────┬───────────────────────────────────────┘  │
│                   │                                            │
│                   ▼                                            │
│  ┌────────────────────────────────────────────────────────┐  │
│  │              Application Services                      │  │
│  ├────────────────────────────────────────────────────────┤  │
│  │  EmailService          │  Orchestrates email ops       │  │
│  │  AccountService        │  Manages email accounts       │  │
│  │  ConfigurationService  │  App configuration            │  │
│  │  SyncService           │  Background email sync        │  │
│  └────────────────┬───────────────────────────────────────┘  │
│                   │                                            │
│                   ▼                                            │
│  ┌────────────────────────────────────────────────────────┐  │
│  │              Core Services                             │  │
│  ├────────────────────────────────────────────────────────┤  │
│  │  KeyManagementService  │  QKD integration (ETSI 014)   │  │
│  │  EncryptionService     │  Multi-level encryption       │  │
│  │  SMTPService           │  Email sending (nodemailer)   │  │
│  │  IMAPService           │  Email receiving (imap)       │  │
│  │  AttachmentHandler     │  File management              │  │
│  └────────────────┬───────────────────────────────────────┘  │
│                   │                                            │
│                   ▼                                            │
│  ┌────────────────────────────────────────────────────────┐  │
│  │              Data Access Layer                         │  │
│  ├────────────────────────────────────────────────────────┤  │
│  │  EmailRepository       │  Email CRUD operations        │  │
│  │  AccountRepository     │  Account CRUD operations      │  │
│  │  KeyRepository         │  Quantum key cache            │  │
│  │  ConfigRepository      │  Configuration storage        │  │
│  └────────────────┬───────────────────────────────────────┘  │
│                   │                                            │
│                   ▼                                            │
│  ┌────────────────────────────────────────────────────────┐  │
│  │              Data Storage                              │  │
│  ├────────────────────────────────────────────────────────┤  │
│  │  SQLite Database (better-sqlite3)                      │  │
│  │  Encrypted Store (electron-store)                      │  │
│  │  System Keychain (keytar)                              │  │
│  │  File System (attachments)                             │  │
│  └────────────────────────────────────────────────────────┘  │
│                                                                │
└──────────────────────────────────────────────────────────────┘

