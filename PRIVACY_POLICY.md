# Privacy Policy

**Last Updated:** April 15, 2026

Atlas Ledger is committed to protecting your privacy and securing your financial data. This policy explains how we collect, use, and protect your information.

## Data Collection

### Information You Provide

Atlas Ledger is a **local-first** application. All data is stored on your device by default. The only exceptions are optional iCloud sync (encrypted, user-controlled) and optional accountant share links (zero-knowledge encrypted; see Accountant Sharing below). Data that you have not explicitly chosen to share is NOT transmitted to external servers:

- **Expense Records:** Amount, merchant, category, date, notes, payment method
- **Receipt Images:** Photos captured or selected from your photo library
- **Mileage Tracking:** Trip start/end locations, distance, date, purpose
- **Income Records:** Amount, source, category, date, recurring frequency
- **Bill Records:** Bill name, amount, due date, category, payment history
- **Account Information:** Account names, balances (all stored locally)
- **Savings Goals:** Goal names, target amounts, contributions
- **Voice Memos:** Audio recordings attached to expenses, stored as `.m4a` files in the app's local `/voice/` directory
- **Speech Transcripts:** Text transcribed from voice dictation, stored with the associated expense record
- **Pay Schedule:** Pay frequency and anchor date used for budget planning
- **Bill Allocation Preferences:** Which bills are mapped to which paychecks (no deposit amounts are recorded)
- **AI Correction Feedback:** When you correct an on-device AI suggestion, the correction (field name, model output, your correction, merchant context) is stored locally to improve future suggestions
- **Attached Documents:** Files you attach to expense records (PDFs, images, or other documents) are stored in the app's local documents directory and encrypted using iOS Data Protection; document text extracted via on-device OCR is stored locally with the document record

### Information We DO NOT Collect

- We do not collect personally identifiable information
- We do not track your usage or analytics
- We do not have access to your financial data
- We do not share data with third parties
- We do not use advertising or tracking SDKs

## Data Protection & Security

### Encryption at Rest

**All financial data in Atlas Ledger is encrypted at rest using iOS Data Protection.**

- **Protection Level:** `NSFileProtectionCompleteUntilFirstUserAuthentication` — iOS hardware encryption applied to all app files
- **What This Means:**
  - All app data is encrypted when your device is powered off or locked before first unlock
  - Data is protected even if your device is stolen while powered off
  - After the device is unlocked once per boot, data remains accessible for background operations (e.g., background sync, notifications)
  - Encryption keys are derived from your device passcode and the device's hardware security module (Secure Enclave)
  - This protection level is enforced at the iOS kernel level — Atlas Ledger cannot weaken or bypass it

### What Gets Protected

iOS Data Protection encrypts:
- SQLite database containing all expense records
- Receipt images stored locally
- Mileage location history
- Income records and recurring schedules
- Bill records and payment history
- Account balances and settings
- Savings goal information
- Voice memo recordings (`.m4a` files)
- Speech transcripts
- Pay schedule and bill allocation preferences
- User preferences
- Attached documents (PDFs and images stored in the app's local documents directory)
- OCR text extracted from attached documents (stored locally with the document record)

### Subscription Status (iOS Keychain)

Your Pro subscription status (tier and expiration date) is cached in the iOS Keychain. The Keychain is a secure, encrypted storage managed by iOS — the data is protected by your device passcode and is not accessible to other apps. Atlas Ledger stores only the resulting tier status; your payment card details are handled entirely by Apple and never reach the app.

### Device Security Requirements

To ensure data protection works:
- Your device **must have a passcode** enabled
- We recommend using Face ID or Touch ID for additional security
- Keep your iOS version up to date for latest security patches

## Data Storage

### Local Storage Only

All data is stored exclusively on your device using:
- **SQLite Database:** Encrypted expense, mileage, account, income, bill, pay schedule, and transcript data
- **File System:** Encrypted receipt images, voice memo recordings (`.m4a`), and attached documents (PDFs and images stored in the app's documents directory)
- **iOS Keychain:** Pro subscription tier and expiration date (encrypted by iOS)
- **AsyncStorage:** On-device AI correction feedback (model name, field name, model output, user correction, merchant context, timestamp) — stored locally only, never synced
- **App Cache Directory:** Core ML model files downloaded for on-device AI features — deletable by clearing app cache
- **Secure Storage:** Settings and preferences

### iCloud Sync (Optional)

Atlas Ledger supports optional iCloud sync, available to all users at no cost. When enabled in Settings → iCloud Sync:
- Your expense data, receipts, and mileage records are synced across your Apple devices via iCloud (CloudKit)
- Data is encrypted in transit and at rest by Apple's CloudKit infrastructure
- Sync is **opt-in** and can be disabled at any time in Settings
- Apple handles the encrypted data as a conduit; they cannot read the content of your records
- Disabling sync stops new data from being uploaded; existing iCloud data can be deleted from Settings

**iCloud Backup:** When you back up your device to iCloud, iOS encrypts your Atlas Ledger data end-to-end. Apple cannot access this data.

### Accountant Sharing (Optional)

When you use the Share feature to send a report to your accountant:
- A snapshot of the selected expense data and receipt images is **encrypted on your device** using AES-256-GCM before any data leaves the app
- The encrypted snapshot is uploaded to **atlasledger.live** solely to host it for the recipient
- The decryption key is embedded in the share link and is **never transmitted to our server** (zero-knowledge architecture)
- Share links expire after the duration you choose (default: 7 days) and can be revoked at any time from Settings → Active Shares
- We do not process, read, or retain the contents of shared snapshots; we store only the encrypted ciphertext until expiry or revocation
- Receipt images included in a share are encrypted with the same key and stored as ciphertext alongside the snapshot

**If you do not use the Share feature, no data is ever uploaded to atlasledger.live.**

## Permissions

### Camera Access (Optional)

**Why We Need It:** To capture receipt photos for expense tracking

**What We Do:**
- Photos are stored locally on your device
- Images are encrypted using iOS Data Protection
- We never upload images to external servers

**You Can:**
- Deny camera access and manually add expenses
- Revoke permission anytime in iOS Settings

### Photo Library Access (Optional)

**Why We Need It:** To select existing receipt photos

**What We Do:**
- You control which photos to share
- Images are copied to app storage and encrypted
- Original photos remain in your library

### Microphone Access (Optional)

**Why We Need It:** To record voice memos attached to expenses

**What We Do:**
- Record audio locally and save it as an encrypted `.m4a` file in the app's `/voice/` directory
- Recordings are capped at 2 minutes each
- Audio is never transmitted outside your device

**You Can:**
- Deny microphone access and add text notes instead
- Delete individual voice memos at any time
- Revoke permission anytime in iOS Settings

### Speech Recognition (Optional)

**Why We Need It:** To transcribe voice dictation into expense notes and descriptions; also used for voice-activated trip tracking commands ("Start trip", "End trip") in Atlas Pro

**What We Do:**
- Use iOS's built-in on-device speech recognizer (`SFSpeechRecognizer`) when available — no audio is sent to Apple's servers in on-device mode
- Transcripts are stored locally with the expense record and are searchable within the app
- Voice commands for trip tracking are processed on-device; text-to-speech feedback uses on-device synthesis (no network required)

**You Can:**
- Skip voice dictation and type notes manually
- Delete transcripts by editing or deleting the associated expense
- Revoke permission anytime in iOS Settings

### Location Access (Optional - Atlas Pro Only)

**Why We Need It:** To automatically track mileage for trip logging; trip start/stop can also be triggered via voice command

**What We Do:**
- Record start/end location coordinates for trips
- Location data stored locally and encrypted
- Background location used only when actively tracking a trip
- Trip state is persisted locally so tracking survives an app restart

**You Can:**
- Use Atlas without enabling location tracking
- Manually enter mileage instead
- Revoke permission anytime

## Data Retention

### How Long We Keep Data

You control your data retention:
- Expenses, receipts, and mileage are kept until you delete them
- Deleted items are permanently removed (no recovery)
- Uninstalling the app deletes all local data

### Data Export

You can export your data anytime:
- Export to CSV for spreadsheets
- Export to PDF for tax filing
- All exports are generated locally on your device

## Future Features

We may add additional optional cloud features in future releases. Any new data transmission will be disclosed in an updated privacy policy before release, and all such features will be opt-in.

## Third-Party Services

### AI-Powered Category Suggestions

Atlas Ledger uses on-device machine learning to suggest expense categories:
- **All processing happens locally** on your device (no cloud AI)
- Category suggestions based on merchant name and your past categorizations
- Your categorization history is stored locally and encrypted
- We never send your expense data to external AI services
- Suggestions are always optional and can be overridden

### Cash Flow Projections

30, 60, and 90-day cash flow projections are calculated entirely on-device using your local expense, bill, and income data. No data is sent to external servers for this calculation.

### On-Device AI Features

Atlas Ledger uses three on-device Core ML models — Receipt Understanding, Voice Intent, and NL Query — to help you capture and query expenses faster. **No receipt images, voice audio, query text, or expense data is transmitted to any server for ML processing.** All inference runs entirely on your device.

**These AI features are optional.** You can enable or disable each model individually in Settings → Intelligence. The app functions fully without them: receipt scanning falls back to built-in regex-based processing, voice commands fall back to the built-in natural language parser, and the query engine uses built-in pattern matching. No functionality is lost when models are disabled or not downloaded.

#### Receipt Field Extraction (`ReceiptUnderstandingService`)

When you scan a receipt, the app processes the image on-device using the Receipt Understanding Core ML model to extract merchant name, amount, date, category, tax, and tip:

- Processing happens entirely on your device — no receipt image is transmitted to any server for ML processing
- Receipt images are stored in the app's local document directory and encrypted using iOS Data Protection (see Data Protection above)
- If iCloud receipt sync is enabled (opt-in), images may be uploaded to **your own iCloud account** via Apple's CloudKit infrastructure — Atlas Ledger does not host or have access to these images
- Extracted field suggestions are labeled "suggested" and always require your confirmation before saving
- No receipt image or extracted data is sent to any Atlas Ledger server or third-party AI service

#### Document Wallet (`DocumentService`)

When you attach a document to an expense and request text extraction:

- Document files (PDFs, images) are stored in the app's local documents directory and encrypted using iOS Data Protection
- Text extraction is performed on-device using the same ML Kit text recognition framework used for receipt scanning — no document image or extracted text is transmitted to any server
- Extracted text is stored locally in the SQLite database alongside the document record and is searchable within the app
- If iCloud receipt sync is enabled (opt-in), attached document files may be synced to your own iCloud account via Apple's CloudKit infrastructure — Atlas Ledger does not host or have access to these files
- Documents attached to expenses are not included in accountant share links unless you explicitly include them

#### Voice Command Processing (`VoiceIntentService`)

When you use voice commands to log expenses or control the app:

- Voice input is processed by **Apple's on-device Speech Recognition framework** (`SFSpeechRecognizer`) in on-device mode — audio is not sent to Apple's servers
- Intent classification and entity extraction run on-device using the Voice Intent Core ML model — no audio or text is sent to any third-party service
- **Transcript retention:** The speech-to-text transcript is held in memory only for the duration of the command. If the command results in an expense the user saves, only the final saved expense fields are persisted — the raw transcript is discarded. If the command is cancelled or not saved, the transcript is discarded immediately.
- Atlas Ledger does not retain audio recordings of voice commands at any point
- Nothing about the voice session (audio, transcript, or intent) is transmitted to Anthropic or any other third party

#### Natural Language Queries (`NLQueryService`)

When you ask questions about your expenses in plain language (for example, "How much did I spend on food last month?"):

- Your query text is processed entirely on-device using template matching — no query text is sent to any external server or cloud API
- The app queries your local SQLite database to answer the question
- Query text and results never leave your device
- No cloud API is called for natural language understanding

#### AI Correction Feedback (`CorrectionFeedbackService`)

When you correct an AI-suggested field (for example, changing a suggested category), Atlas Ledger stores that correction locally to improve future suggestions:

- **What is stored:** Model name, field name, the model's original output, your correction, merchant context, and timestamp
- **Where it is stored:** AsyncStorage on your device only
- **What is NOT stored:** The original receipt image, full expense record, or any identifying information beyond merchant context
- This data is never synced to iCloud, shared with other apps, or transmitted to any server
- You can delete correction feedback data by uninstalling the app or clearing app data

#### Core ML Model Storage

On-device AI models (Core ML format) are stored in the app's cache directory on your device:

- Models are downloaded to your device and stored locally — they are never uploaded or shared
- You can delete individual models directly in Settings → Intelligence (tap the delete button next to any model)
- You can also delete all cached models by clearing the app's cache in iOS Settings → General → iPhone Storage → Atlas Ledger
- Deleting a model does not affect your expense data; the corresponding AI feature simply reverts to built-in processing

### Atlas Pro Subscription (Apple StoreKit 2)

Subscription purchases are processed entirely by Apple:
- Atlas Ledger never receives or stores your payment card details
- Apple processes and validates subscription receipts; the app receives only the resulting tier status (free or Pro) and expiration date
- That tier status is cached locally in the iOS Keychain (see Data Protection above)
- Subscription status is validated against Apple's servers on app launch to confirm it remains active

### FinanceKit (Apple)

When you enable FinanceKit integration:
- Data is requested directly from Apple's FinanceKit API
- Imported transactions are stored locally and encrypted
- We do not transmit your financial data to external servers
- You control which transactions to import

### TestFlight Beta Testing

If you participate in TestFlight beta:
- Crash logs may be sent to Apple for debugging
- Crash reports do not contain your expense data
- Reports include device info and app version only

## Children's Privacy

Atlas Ledger is not intended for users under 13. We do not knowingly collect data from children.

## Changes to This Policy

We will update this privacy policy as needed:
- Material changes will be communicated via the App Store release notes
- "Last Updated" date will be updated at the top of this document
- Continued use of the app after changes constitutes acceptance

## Your Rights

You have the right to:
- **Access:** View all data stored in the app
- **Export:** Export data to CSV or PDF
- **Delete:** Permanently delete expenses, receipts, or all data
- **Control:** Manage permissions in iOS Settings

## Data Security Best Practices

To protect your financial data:
1. **Enable Device Passcode:** Required for iOS Data Protection
2. **Use Face ID/Touch ID:** Adds biometric authentication layer
3. **Enable iCloud Backup:** Encrypted backups in case of device loss
4. **Keep iOS Updated:** Latest security patches
5. **Review Permissions:** Regularly check app permissions in Settings

## Contact Us

For privacy questions or concerns:
- **GitHub Issues:** https://github.com/neptune/atlas-ledger/issues
- **Email:** privacy@atlasledger.app

## Transparency Commitment

Atlas Ledger is built on transparency:
- Local-first architecture means your data stays with you
- No hidden data collection or analytics
- No third-party tracking SDKs
- Bank-level encryption for all financial data

---

**Summary:**
- All data stored locally on your device by default
- Optional iCloud sync (free for all users): encrypted in transit and at rest via Apple CloudKit
- Optional accountant sharing: data is AES-256-GCM encrypted on-device before upload; server never holds your decryption key
- Bank-level encryption using iOS Data Protection; subscription status encrypted in iOS Keychain
- You control all permissions and data retention
- No tracking, no analytics, no third-party data sharing
- Voice memos and speech transcripts stay on your device
- Attached documents (PDFs and images) are stored locally and encrypted; OCR text extraction runs on-device — no document content is transmitted to any server
- Subscription purchases handled entirely by Apple — Atlas Ledger never sees payment details
- All AI features (receipt extraction, document OCR, voice commands, natural language queries, correction feedback) run entirely on-device — no expense data, document files, voice recordings, or query text is transmitted to any third-party service for AI processing
- AI models are optional and can be disabled per-feature in Settings → Intelligence; the app works fully without them using built-in processing
- When you correct an AI suggestion, the correction is stored locally only and never leaves your device
