# المعمارية التقنية التفصيلية
## Senegal Smart Guide Hub - Technical Architecture

---

## 🏗️ نظرة عامة على المعمارية

```
┌─────────────────────────────────────────────────────────────────┐
│                        CENTRAL HUB (Dakar)                       │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │         Central Command & Control Dashboard              │   │
│  │  • Real-time Pilgrim Distribution Map                    │   │
│  │  • Missing Cases Alert System                            │   │
│  │  • Analytics & Reporting Engine                          │   │
│  │  • API Management & Integration                          │   │
│  └──────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
         ↑                    ↑                      ↑
         │                    │                      │
    (Secure APIs)        (Secure APIs)          (Secure APIs)
         │                    │                      │
         └────────────────────┼──────────────────────┘
                              │
        ┌─────────────────────┼─────────────────────┐
        │                     │                     │
   (Mecca Hub)           (Medina Hub)         (Embassy, Dakar)
        │                     │                     │
   ┌─────────────┐       ┌─────────────┐     ┌─────────────┐
   │  Operations │       │ Operations  │     │ Coordination │
   │  Center 1   │       │  Center 2   │     │   Office    │
   │  (20 staff) │       │  (15 staff) │     │  (5 staff)  │
   └─────────────┘       └─────────────┘     └─────────────┘
        ↑                     ↑                     ↑
    BLE ↓ Beacon          BLE ↓ Beacon           HTTP ↑
        ↓                     ↓                     ↓
    ┌─────────────┐       ┌─────────────┐     
    │  Volunteer  │       │  Volunteer  │     
    │   App (50)  │       │    App(30)  │     
    └─────────────┘       └─────────────┘     
        ↑                     ↑
    NFC/QR ↑ BLE          NFC/QR ↑ BLE
        │  │ GPS              │  │ GPS
   ┌──────────────────┐  ┌──────────────────┐
   │  Pilgrims (5K)   │  │  Pilgrims (3K)   │
   │  with Smart Band │  │  with Smart Band │
   └──────────────────┘  └──────────────────┘
```

---

## 🔧 المكونات الأساسية

### **1. الطبقة الجهازية (Device Layer)**

#### أ) السوار الذكي (Smart Band NFC/QR)

**المواصفات الفنية:**
```
- Processor: ARM Cortex-M0 (8-bit)
- Memory: 512KB ROM / 64KB RAM
- Battery: Lithium-ion 400mAh (60 days)
- Connectivity: BLE 5.0 (range: 100m)
- NFC: ISO14443A + ISO14443B
- QR Code: Etched on silicone surface
- Operating Temp: 0-50°C
- IP Rating: IP67 (water & dust resistant)
- Dimensions: 65mm × 160mm × 8mm
- Weight: 35g
- Colors: Available in multiple colors
```

**البيانات المخزنة:**
```json
{
  "uid": "SG2026-00001-SGD",
  "pilgrims": {
    "fullName": "Mamadou Diallo",
    "nameArabic": "ممادو ديالو",
    "nameWolof": "Mamadou Diallo",
    "passportNumber": "SN123456789",
    "dateOfBirth": "1960-03-15",
    "gender": "M",
    "bloodType": "O+",
    "medicalInfo": "Diabetes, takes insulin",
    "contactEmergency1": "+221 78 123 4567",
    "contactEmergency2": "+221 77 987 6543",
    "contactFamily": "+33 1 45 67 89 00",
    "embassyContact": "+966 11 478 5000",
    "hadjiGroup": "Group-05-Kaolack",
    "hadjiGroupLeader": "Aissatou Sarr",
    "accommodationMecca": "Al-Ajyad Hotel, Room 504",
    "accommodationMedina": "Al-Eiman Hotel, Room 201",
    "qrCodeURL": "https://smartguide.sen.gov/pilgrim/SG2026-00001-SGD",
    "encryptionKey": "AES-256-v2",
    "timestamp": "2026-06-10T08:45:00Z"
  }
}
```

**بروتوكول الاتصال:**
```
عندما يقترب الهاتف من السوار:

1. البحث (Scan):
   - الهاتف يبحث عن NFC signals
   - مدة 1-2 ثانية

2. المصادقة (Authentication):
   - تحقق من توقيع السوار الرقمي
   - فك تشفير البيانات (AES-256)

3. النقل (Transfer):
   - نقل البيانات من السوار إلى الهاتف
   - مدة 2-3 ثوان

4. الحفظ (Storage):
   - حفظ على جهاز التطبيق محلياً
   - إرسال للخادم عند توفر الإنترنت
```

#### ب) جهاز الإرسال الذكي (Beacon Device - قائد المجموعة)

**المواصفات:**
```
- Processor: STM32L476 (ARM Cortex-M4)
- Battery: 3.7V 2000mAh Li-ion battery (30 days)
- BLE: v5.0 (range: 150m with external antenna)
- GPS: u-blox NEO-M8N (accuracy: ±2.5m)
- Display: Small OLED (0.96")
- Buttons: 3 (Power, Beacon, SOS)
- Size: 80mm × 80mm × 25mm (pocket size)
- Weight: 120g
- IP Rating: IP65
- Operating Temp: -10 to +60°C
```

**الوظائف الأساسية:**
1. **Geo-Fencing:** تحديد نطاق آمن 50-100 متر
2. **Direct Messaging:** إرسال رسائل صوتية
3. **SOS Button:** تنبيه فوري
4. **Location Tracking:** تسجيل الموقع كل 30 ثانية
5. **Offline Mode:** العمل بدون إنترنت

---

### **2. الطبقة البرمجية (Software Layer)**

#### أ) تطبيق المتطوعين (Volunteer Responder App)

**البيئة التطويرية:**
```
- Framework: Flutter / React Native
- Backend: Node.js + Express
- Database: PostgreSQL (primary) + MongoDB (cache)
- Cache: Redis
- Message Queue: RabbitMQ
- Authentication: JWT + OAuth 2.0
- API: RESTful + GraphQL
```

**الميزات الأساسية:**

```javascript
// 1. تسجيل الدخول
POST /api/auth/login
{
  "username": "volunteer_id",
  "password": "encrypted",
  "location": { "latitude": 21.427324, "longitude": 39.826168 }
}

// 2. استقبال تنبيهات الحجاج التائهين
WebSocket: /api/alerts/subscribe
{
  "event": "PILGRIM_MISSING",
  "pilgrimId": "SG2026-00001-SGD",
  "lastLocation": { "lat": 21.429, "lng": 39.825 },
  "distance": "500m from your location",
  "urgency": "HIGH",
  "timestamp": 1623936000000
}

// 3. تقديم تحديث (وجدت الحاج)
POST /api/cases/{caseId}/found
{
  "volunteerId": "VOL-2026-001",
  "selfie": "base64_image",
  "location": { "latitude": 21.429, "longitude": 39.825 },
  "notes": "Found at entrance of Haram"
}
```

**واجهة المستخدم (Wolof + Arabic + English):**

```
[HOME SCREEN]
┌──────────────────────────────────┐
│  Assalamu Alaikum, Aissatou      │
│  Status: ON DUTY (5m ago)        │
├──────────────────────────────────┤
│                                  │
│   [🔴 URGENT: MISSING PILGRIM]  │
│                                  │
│   Name: Amine Diop              │
│   Age: 67 | Male               │
│   Last seen: 3 minutes ago      │
│   Distance: 800m South          │
│                                  │
│   [📍 SEE MAP] [📞 CALL]        │
│                                  │
├──────────────────────────────────┤
│ Active Cases Today: 2            │
│ Cases Resolved: 12               │
│ Success Rate: 100%               │
└──────────────────────────────────┘
```

#### ب) لوحة التحكم المركزية (Central Dashboard)

**التقنيات المستخدمة:**
```
- Frontend: React.js + D3.js (visualization)
- Real-time: WebSocket + Socket.io
- Maps: Mapbox GL + OpenStreetMap
- Charts: Chart.js + Recharts
- Authentication: Single Sign-On (SSO)
- Hosting: AWS ECS + CloudFront
```

**الميزات الرئيسية:**

```
1. LIVE TRACKING:
   - خريطة حية تعرض موقع كل حاج
   - تحديث كل 30-60 ثانية
   - تلوين حسب الحالة (أخضر=آمن، أحمر=في خطر)

2. INCIDENT MANAGEMENT:
   - عرض جميع الحالات المفتوحة
   - أولويات (Priority Levels)
   - Time to Resolution

3. STATISTICS:
   - عدد الحجاج الحاليين في كل مكان
   - الحالات المحلولة/المفتوحة
   - متوسط وقت الاستجابة
   - معدل النجاح

4. ALERTS:
   - إنذارات فورية عند زيادة الزحام
   - تنبيهات طبية عند الحاجة
   - تقارير شبه يومية
```

---

### **3. الطبقة السحابية (Cloud Layer)**

#### أ) معمارية الخادم

```
┌─────────────────────────────────────────────┐
│         AWS Infrastructure (Primary)         │
├─────────────────────────────────────────────┤
│                                             │
│  ┌──────────────────────────────────────┐  │
│  │      Load Balancer (ALB)             │  │
│  │  • SSL/TLS Termination              │  │
│  │  • DDoS Protection                  │  │
│  └──────────────────────────────────────┘  │
│              ↓                              │
│  ┌──────────────────────────────────────┐  │
│  │  Auto Scaling Group (ECS)            │  │
│  │  • 5-20 containers (dynamic)         │  │
│  │  • CPU autoscaling                   │  │
│  │  • Health checks every 30s            │  │
│  └──────────────────────────────────────┘  │
│              ↓                              │
│  ┌──────────────────────────────────────┐  │
│  │  Databases                           │  │
│  │  ├─ RDS PostgreSQL (Multi-AZ)        │  │
│  │  ├─ ElastiCache Redis                │  │
│  │  └─ DynamoDB (Backup)                │  │
│  └──────────────────────────────────────┘  │
│              ↓                              │
│  ┌──────────────────────────────────────┐  │
│  │  Storage                             │  │
│  │  ├─ S3 (Backups & Images)            │  │
│  │  └─ CloudFront (CDN)                 │  │
│  └──────────────────────────────────────┘  │
│                                             │
└─────────────────────────────────────────────┘
```

#### ب) قاعدة البيانات الأساسية

**Schema (مبسط):**

```sql
-- 1. Pilgrims Table
CREATE TABLE pilgrims (
  id VARCHAR(20) PRIMARY KEY,           -- SG2026-00001-SGD
  full_name VARCHAR(100),
  passport_number VARCHAR(20),
  date_of_birth DATE,
  gender CHAR(1),
  blood_type VARCHAR(3),
  medical_info TEXT,
  emergency_contact_1 VARCHAR(20),
  emergency_contact_2 VARCHAR(20),
  group_id VARCHAR(20),
  band_uid VARCHAR(50),
  created_at TIMESTAMP,
  updated_at TIMESTAMP
);

-- 2. Location Tracking
CREATE TABLE location_history (
  id BIGINT PRIMARY KEY AUTO_INCREMENT,
  pilgrim_id VARCHAR(20),
  latitude DECIMAL(10,8),
  longitude DECIMAL(11,8),
  accuracy INT,                         -- meters
  timestamp TIMESTAMP,
  source VARCHAR(10),                   -- 'GPS', 'BLE', 'NFC'
  FOREIGN KEY (pilgrim_id) REFERENCES pilgrims(id),
  INDEX (pilgrim_id, timestamp)
);

-- 3. Incidents
CREATE TABLE incidents (
  id VARCHAR(20) PRIMARY KEY,
  pilgrim_id VARCHAR(20),
  incident_type VARCHAR(30),            -- 'MISSING', 'MEDICAL', 'SECURITY'
  status VARCHAR(20),                   -- 'OPEN', 'IN_PROGRESS', 'RESOLVED'
  severity VARCHAR(10),                 -- 'LOW', 'MEDIUM', 'HIGH', 'CRITICAL'
  created_at TIMESTAMP,
  resolved_at TIMESTAMP,
  resolution_time_minutes INT,
  resolving_volunteer_id VARCHAR(20),
  notes TEXT,
  FOREIGN KEY (pilgrim_id) REFERENCES pilgrims(id),
  INDEX (status, severity, created_at)
);

-- 4. Volunteers
CREATE TABLE volunteers (
  id VARCHAR(20) PRIMARY KEY,
  full_name VARCHAR(100),
  phone_number VARCHAR(20),
  current_location_lat DECIMAL(10,8),
  current_location_lng DECIMAL(11,8),
  status VARCHAR(20),                   -- 'ACTIVE', 'OFF_DUTY', 'BREAK'
  cases_resolved INT DEFAULT 0,
  rating DECIMAL(3,2),                  -- 1-5 stars
  created_at TIMESTAMP
);
```

---

### **4. الطبقة الأمنية (Security Layer)**

#### أ) تشفير البيانات

```
[TRANSIT ENCRYPTION]
Client ←→ Server: TLS 1.3 + ECDHE
Client ←→ Band: AES-256-GCM

[AT-REST ENCRYPTION]
Database: AES-256 (column-level)
Backups: AES-256-CBC

[KEY MANAGEMENT]
- KMS: AWS Key Management Service
- Key Rotation: Every 90 days
- Secrets: Hashicorp Vault
```

#### ب) المصادقة والتفويض

```
[MULTI-FACTOR AUTHENTICATION]
Volunteers:
  1. Username/Password (PBKDF2)
  2. OTP via SMS (TOTP)
  3. Biometric (Fingerprint optional)

Admins:
  1. Username/Password
  2. Hardware Security Key (FIDO2)
  3. Audit logs

[ROLE-BASED ACCESS CONTROL (RBAC)]
- Volunteer: Read own data + respond to alerts
- Operation Manager: Read all data + manage volunteers
- Director: Full access + configure system
- Admin: System configuration + backup
```

#### ج) الالتزام القانوني

```
[DATA PROTECTION]
- GDPR Compliance: ✓ (Right to deletion, data export)
- Local Laws: ✓ (Senegalese data protection law)
- HIPAA-like: ✓ (Medical data encryption)

[AUDIT & LOGGING]
- All access logged: WHO, WHAT, WHEN, WHERE
- Retention: 2 years
- Review: Monthly by security team
```

---

## 🔄 سير العمل (Workflows)

### **سيناريو 1: اكتشاف حاج تائه**

```
Timeline:
├─ T+0s: Pilgrim moves >50m from beacon
├─ T+1s: Voice alert in Wolof: "You are far from the group!"
├─ T+2s: Vibration in band
├─ T+3s: Alert sent to Group Leader
├─ T+5s: If no response → Alert to nearest 50 volunteers
├─ T+10s: Volunteers see case in app + location map
├─ T+30s: First volunteers start moving toward location
├─ T+3min: Volunteer confirms visual contact
├─ T+5min: Pilgrim returned to group safely
└─ T+7min: Incident closed in system + statistics updated
```

### **سيناريو 2: حاج يمسح CoDE QR عند الضياع**

```
Timeline:
├─ T+0s: Pilgrim seeks help from security officer
├─ T+5s: Officer scans QR code with smartphone
├─ T+2s: App displays pilgrim info (name, emergency contacts)
├─ T+1s: App shows: "Embassy contact: +966 11 478 5000"
├─ T+10s: Officer calls embassy directly
├─ T+2min: Embassy staff verifies pilgrim identity
├─ T+5min: Group leader contacted + located
├─ T+15min: Pilgrim reunited with group
└─ T+20min: Incident reported to dashboard
```

---

## 📊 مؤشرات الأداء (KPIs)

```javascript
// 1. Response Time (متوسط الاستجابة)
Average: ≤ 15 minutes
Target: ≤ 10 minutes

// 2. Resolution Rate (معدل الحل)
Target: 100% same day
Backup: 98% within 24 hours

// 3. System Availability (توفر النظام)
Target: 99.9% uptime
SLA: 4.5 hours downtime/month max

// 4. User Satisfaction (رضا المستخدمين)
Expected: ≥ 90% satisfied
Measurement: Post-incident survey

// 5. False Positives (تنبيهات خاطئة)
Target: ≤ 5%
Solution: ML algorithm to filter duplicates

// 6. Data Accuracy (دقة البيانات)
Target: ≥ 98%
Verification: Match between band-data vs database
```

---

## 🛠️ خطة التطوير التقنية

### **المرحلة 1: الأساس (شهر 1-2)**
- [ ] تصميم المعمارية التفصيلية
- [ ] تطوير APIs الأساسية
- [ ] قاعدة البيانات والتشفير
- [ ] النماذج الأولية للأجهزة

### **المرحلة 2: التطبيقات (شهر 2-3)**
- [ ] تطبيق المتطوعين (MVP)
- [ ] لوحة التحكم الأساسية
- [ ] نظام الإرسالات الصوتية
- [ ] اختبارات الأمان الأولية

### **المرحلة 3: التجربة (شهر 4)**
- [ ] اختبارات ميدانية محدودة
- [ ] تحسينات بناءً على الملاحظات
- [ ] التوثيق الشامل
- [ ] تدريب الفريق

---

*هذا المستند تقني ومتخصص - يُقدّم للفريق الهندسي والتقني فقط*
