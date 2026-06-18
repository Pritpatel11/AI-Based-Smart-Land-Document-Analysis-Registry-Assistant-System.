# AI-Based Smart Land Document Analysis & Registry Assistant System
## Comprehensive Test Case Documentation (Part 1/2)

This document contains detailed test cases for the project, organized by module. Each test case follows the structure of ID, Module, Scenario, Expected Result, Actual Result, Status, and Resolution.

---

### Module 1: Authentication & Access Control
This module handles user registration, login, role-based access control, and password recovery.

| TC ID | Module | Test Description/Scenario | Desired (Expected) Result | Actual Result | Status | How it was Solved (Resolution) |
|---|---|---|---|---|---|---|
| TC_AUTH_01 | Auth | User Registration (Valid data) | Account created; email verification sent. | Account saved; email received | PASS | Implemented using Bcrypt hashing for password & Nodemailer for email. |
| TC_AUTH_02 | Auth | Login with incorrect password | System shows "Invalid credentials". | Error message displayed correctly. | PASS | Solved using Express error-handling middleware. |
| TC_AUTH_03 | Auth | Password Reset (Invalid Email) | System shows "Email not found". | Correct error message shown. | PASS | Implemented lookup check in MongoDB User model. |
| TC_AUTH_04 | Auth | Role-based Access (User to Admin) | User should not see "Audit Vault". | Navigation restricted to User features. | PASS | Solved using React Context `user.role` check in App.jsx. |
| TC_AUTH_05 | Auth | Session Timeout (Expired Token) | User should be logged out after 24h. | Token expired; redirect to Login. | PASS | Configured JWT `expiresIn: '24h'` in backend. |
| TC_AUTH_06 | Auth | Empty Field Registration | Show "All fields are required" error. | Fields highlighted red. | PASS | Added Yup/Zod schema validation on Frontend and Joi on Backend. |
| TC_AUTH_07 | Auth | Duplicate Email Registration | Show "Email already exists" error. | Error handled without crashing. | PASS | Handled via MongoDB `unique: true` index and try-catch block. |
| TC_AUTH_08 | Auth | Password Strength Check | Reject passwords < 8 characters. | Rejected weak password. | PASS | Added regex validation in Signup frontend component. |
| TC_AUTH_09 | Auth | Social Login (Google) | User logged in via Google OAuth. | Profile data synced successfully. | PASS | Implemented using Passport.js / Firebase Auth. |
| TC_AUTH_10 | Auth | Logout Functionality | Clear cookies/localStorage and redirect. | Successfully redirected to Home. | PASS | Cleared `authToken` from Cookies and Context state. |
| TC_AUTH_11 | Auth | Login Rate Limiting | Block user after 5 failed attempts. | System blocked for 15 mins. | PASS | Implemented using `express-rate-limit`. |
| TC_AUTH_12 | Auth | Update Profile Details | Changes reflected in Database. | DB updated correctly. | PASS | Solved using Mongoose `findByIdAndUpdate`. |
| TC_AUTH_13 | Auth | Change Password (Old vs New) | Old password must match. | Validation successful. | PASS | Used Bcrypt `compare` method to verify old password. |
| TC_AUTH_14 | Auth | JWT Token Forgery | System should reject modified token. | Access Denied. | PASS | JWT Secret key verification fails on tampering. |
| TC_AUTH_15 | Auth | Verify Email Link | Activate user status on click. | Status updated to 'Active'. | PASS | Handled via unique verification token in URL params. |
| TC_AUTH_16 | Auth | Login with special emojis in Name | System should encode chars correctly. | Database showed "?" markers. | FAIL | Solved by changing DB collation to `utf8mb4_unicode_ci`. |
| TC_AUTH_17 | Auth | Remember Me check | Persistent login after browser close. | User was logged out on restart. | FAIL | Solved by implementing Refresh Tokens stored in HTTP-only cookies. |

---

### Module 2: Document Upload & Storage
Covers the process of uploading land deeds, validating file formats, and storing them securely.

| TC ID | Module | Test Description/Scenario | Desired (Expected) Result | Actual Result | Status | How it was Solved (Resolution) |
|---|---|---|---|---|---|---|
| TC_UPL_01 | Upload | Upload valid Land Deed (PDF) | File uploaded; processing starts. | Upload successful; preview visible. | PASS | Solved using Multer (storage) and Cloudinary (hosting). |
| TC_UPL_02 | Upload | Upload very large file (>100MB) | Show "File too large" error. | Server crashed/timed out. | FAIL | Solved by increasing Node.js client_max_body_size and adding Multer limits. |
| TC_UPL_03 | Upload | Invalid File Type (SVG/EXE) | Reject upload with error message. | Error message: "Only PDF/JPG/PNG". | PASS | Added file filter in Multer configuration. |
| TC_UPL_04 | Upload | Simultaneous Multiple Uploads | Handle multiple files in queues. | All files processed sequentially. | PASS | Implemented `Promise.all` for parallel Cloudinary uploads. |
| TC_UPL_05 | Upload | Upload with Slow Internet | Handle timeouts gracefully. | Show progress bar / Retry option. | PASS | Added Axios progress event listener on Frontend. |
| TC_UPL_06 | Upload | File Rename on Server | Sanitize file names (remove spaces). | File saved as `user_id_doc.pdf`. | PASS | Customized Multer `destination` and `filename` functions. |
| TC_UPL_07 | Upload | Delete Uploaded Document | Permanent removal from Cloudinary. | File removed and DB entry deleted. | PASS | Used `cloudinary.uploader.destroy` and Mongoose `findOneAndDelete`. |
| TC_UPL_08 | Upload | Checksum Verification | Ensure file integrity after upload. | Hashes match after transfer. | PASS | Added SHA-256 hashing pre- and post-upload for critical docs. |
| TC_UPL_09 | Upload | Metadata Extraction (Client side) | Extract file size/name before upload. | Metadata displayed in preview. | PASS | Used browser File API to show local details. |
| TC_UPL_10 | Upload | Server Storage Full | Graceful error handling for disk space. | "Upload failed: Server error". | PASS | Added disk space check middleware in backend. |
| TC_UPL_11 | Upload | Corrupted PDF File | System should reject corrupt binary. | Processing failed but gave no error. | FAIL | Solved by adding `pdf-parse` integrity check post-upload. |
| TC_UPL_12 | Upload | Filename too long (>255 chars) | System should truncate/reject. | File system error (ENOENT). | FAIL | Solved by adding path length validation in Multer logic. |

---

### Module 3: AI-Powered OCR & Data Extraction
Tests the Gemini Pro AI model's ability to extract structured data from diverse document types.

| TC ID | Module | Test Description/Scenario | Desired (Expected) Result | Actual Result | Status | How it was Solved (Resolution) |
|---|---|---|---|---|---|---|
| TC_OCR_01 | AI OCR | Process clear land deed doc | Data (Owner/Date) extracted. | Details extracted correctly. | PASS | Solved using Gemini Pro Vision API with custom prompts. |
| TC_OCR_02 | AI OCR | Process handwritten old deed | Extract handwritten text accurately. | Garbage values/Special chars | FAIL | Solved by adding a "Pre-processing" layer using Sharp.js to enhance contrast. |
| TC_OCR_03 | AI OCR | Low Resolution Document | Show warning "Image quality low". | Analysis failed with obscure error. | FAIL | Implemented image resolution check before sending to Gemini. |
| TC_OCR_04 | AI OCR | Extraction of Owner Name | Extract full names (First/Last). | Name mapped to UserProfile. | PASS | Refined prompt to return JSON with specific key mapping. |
| TC_OCR_05 | AI OCR | Date of Registration Extraction | Extract DD/MM/YYYY format. | Standardized date format. | PASS | Used Regex parsing on Gemini output to normalize dates. |
| TC_OCR_06 | AI OCR | Survey Number Detection | Identify survey/plot numbers. | Extracted accurately. | PASS | Integrated custom "Search Pattern" for regional document styles. |
| TC_OCR_07 | AI OCR | Multilingual Support (Hindi/Gujarati) | Extract text in non-English scripts. | Accurate translation/extraction. | PASS | Leveraged Gemini's native multilingual OCR capabilities. |
| TC_OCR_08 | AI OCR | Multi-page PDF Processing | Analyze all pages of the document. | Data aggregated from 10 pages. | PASS | Implemented PDF splitting and sequential API calls for large docs. |
| TC_OCR_09 | AI OCR | Error Handling (Garbled Text) | Identify if doc is fake or unreadable. | Flag "Unreadable document". | PASS | Added heuristic check for high confidence scores from AI. |
| TC_OCR_10 | AI OCR | Extraction Latency | Response should be within 10 seconds. | Sometimes took 30+ seconds. | FAIL | Solved by implementing Redis caching for frequently analyzed docs. |
| TC_OCR_11 | AI OCR | Missing Signature detection | Identify if signature field is empty. | AI reported "No data found". | FAIL | Solved by adding a 'Critical Field' validation layer in extraction logic. |
| TC_OCR_12 | AI OCR | Page Orientation (Upside down) | Auto-rotate for better OCR. | Text extraction was gibberish. | FAIL | Solved by using `Tesseract` deskewing or AI auto-orientation prompts. |

---

### Module 4: Risk Analysis & Fraud Detection
Evaluation of the heuristic and AI-driven logic that identifies suspicious records.

| TC ID | Module | Test Description/Scenario | Desired (Expected) Result | Actual Result | Status | How it was Solved (Resolution) |
|---|---|---|---|---|---|---|
| TC_RSK_01 | Risk | Signature Mismatch Detection | Flag if signatures look inconsistent. | Flagged as "Suspicious". | PASS | Used OpenCV/AI to compare signature contours across pages. |
| TC_RSK_02 | Risk | Altered Date Detection | Detect white-out or digital edits. | Flagged "Altered Document". | PASS | Gemini vision model trained to detect pixel-level inconsistencies. |
| TC_RSK_03 | Risk | Blacklisted Owner Check | Compare owner name against govt lists. | Alert: "High Risk Owner". | PASS | Integrated with external Land Records CSV/API. |
| TC_RSK_04 | Risk | Duplicate Document detection | Detect if same deed is uploaded twice. | Show "Already Analyzed". | PASS | Used MD5 hash of file content as a unique identifier in DB. |
| TC_RSK_05 | Risk | Stamps Verification | Verify if legal stamps are present. | Verification successful. | PASS | AI-model detects Presence of Stamp paper watermark. |
| TC_RSK_06 | Risk | Valuation Logic Check | Flag if price is 50% below market. | "Under-valued" Warning. | PASS | Implemented calculation using Area * Regional Govt Rates. |
| TC_RSK_07 | Risk | Fraud Trend Dashboard | Display monthly fraud attempts. | Graphs updated in Real-time. | PASS | Aggregated AuditLog data into Recharts/Chart.js. |
| TC_RSK_08 | Risk | Status Badge Update (UI) | Badge should turn RED for Fraud. | UI updated instantly. | PASS | Used Socket.io for live status updates to Admin panel. |
| TC_RSK_09 | Risk | Unit Conversion Confusion | Handle Square Yards vs Square Feet. | Area was 9x smaller in reports. | FAIL | Solved by implementing a "Standard Unit Normalizer" middleware. |
| TC_RSK_10 | Risk | Expired Deed Identification | Flag if document date is > 30 years. | No flag raised. | FAIL | Added property age calculation logic in Analysis module. |

---

### Module 5: Admin Panel & Audit Vault
Administrative tools for overseeing system activity and manually verifying flagged documents.

| TC ID | Module | Test Description/Scenario | Desired (Expected) Result | Actual Result | Status | How it was Solved (Resolution) |
|---|---|---|---|---|---|---|
| TC_ADM_01 | Admin | View All Uploaded Docs | List all docs in Audit Vault. | List populated from DB. | PASS | Used `paginate` plugin in Mongoose for efficient loading. |
| TC_ADM_02 | Admin | Manual Verification Toggle | Admin can override AI status. | Status changed to "Verified". | PASS | Backend endpoint `PATCH /verify` updates document status. |
| TC_ADM_03 | Admin | Search User Activity | Filter logs by User ID / Email. | Result shown correctly. | PASS | Implemented MongoDB `$or` search query in admin routes. |
| TC_ADM_04 | Admin | System Analytics (Doc Counts) | Show total Genuine vs Fraud count. | Stats displayed on Top Bar. | PASS | Used MongoDB `aggregation` pipeline for real-time counts. |
| TC_ADM_05 | Admin | Download Audit Report | Export CSV/PDF of suspicious cases. | File downloaded successfully. | PASS | Integrated `json2csv` and `jspdf` libraries. |
| TC_ADM_06 | Admin | Delete Malicious User | Permanent ban and doc removal. | User data wiped. | PASS | Cascade delete implemented for associated documents. |
| TC_ADM_07 | Admin | Real-time Audit Alerts | Admin notified of new high-risk doc. | Browser notification popped. | PASS | Used Web Notification API paired with Socket.io. |
| TC_ADM_08 | Admin | Admin Logging (audit trail) | Log every action taken by Admin. | Logs saved in `AuditLog` model. | PASS | Middleware records `req.user` and `action` on every sensitive route. |
| TC_ADM_09 | Admin | Bulk Export (10k+ records) | Download full audit as CSV. | Server timed out / Memory leak. | FAIL | Solved by switching from `.JSON()` to `Streams` for data export. |
| TC_ADM_10 | Admin | Dashboard Load (Real-time) | Stats should refresh without F5. | Admin had to manually refresh. | FAIL | Solved by adding a `useEffect` interval and Socket listener. |
