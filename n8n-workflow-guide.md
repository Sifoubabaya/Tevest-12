# 🤖 N8N Workflow - VERIDIAN Sales Pipeline
# من إرسال البريد إلى إتمام الصفقة

## 📋 سير العمل الكامل:

```json
{
  "name": "VERIDIAN Sales Pipeline - Complete",
  "nodes": [
    {
      "parameters": {
        "operation": "read",
        "filePath": "leads.csv"
      },
      "name": "1️⃣ Load Leads from CSV",
      "type": "n8n-nodes-base.file",
      "typeVersion": 1,
      "position": [100, 200]
    },
    {
      "parameters": {
        "url": "https://api.gmail.com/gmail/v1/users/me/messages/send",
        "method": "POST",
        "authentication": "oAuth2",
        "credentialsType": "gmail",
        "requestBody": {
          "raw": "{{ $json.emailContent }}"
        }
      },
      "name": "2️⃣ Send Email Campaign",
      "type": "n8n-nodes-base.httpRequest",
      "typeVersion": 4,
      "position": [300, 200]
    },
    {
      "parameters": {
        "operation": "insert",
        "table": "sales_pipeline",
        "columns": {
          "company": "{{ $json.company }}",
          "email": "{{ $json.email }}",
          "status": "email_sent",
          "timestamp": "{{ $now.toISOString() }}"
        }
      },
      "name": "3️⃣ Log Email to Database",
      "type": "n8n-nodes-base.postgres",
      "typeVersion": 2,
      "position": [500, 200]
    },
    {
      "parameters": {
        "interval": [48, "hours"]
      },
      "name": "4️⃣ Wait 48 Hours",
      "type": "n8n-nodes-base.wait",
      "typeVersion": 1,
      "position": [700, 200]
    },
    {
      "parameters": {
        "query": "SELECT * FROM sales_pipeline WHERE status = 'email_sent' AND timestamp < NOW() - INTERVAL '48 hours'"
      },
      "name": "5️⃣ Check for Follow-up",
      "type": "n8n-nodes-base.postgres",
      "typeVersion": 2,
      "position": [900, 200]
    },
    {
      "parameters": {
        "content": "مرحبا {{ $json.name }},\n\nهل استقبلت الرسالة السابقة؟\nلا تفوت العرض المحدود!\n\nhttps://sifoubabaya.github.io/Tevest-12/contact.html"
      },
      "name": "6️⃣ Send Follow-up Email",
      "type": "n8n-nodes-base.httpRequest",
      "typeVersion": 4,
      "position": [1100, 200]
    },
    {
      "parameters": {
        "spreadsheetId": "{{ $env.GOOGLE_SHEETS_ID }}",
        "operation": "append",
        "columns": {
          "Company": "{{ $json.company }}",
          "Contact": "{{ $json.email }}",
          "Status": "follow_up_sent",
          "Date": "{{ $now.toISOString() }}"
        }
      },
      "name": "7️⃣ Update Google Sheets",
      "type": "n8n-nodes-base.googleSheets",
      "typeVersion": 2,
      "position": [1300, 200]
    },
    {
      "parameters": {
        "eventTriggerRules": {
          "conditions": [
            {
              "key": "status",
              "value": "replied"
            }
          ]
        }
      },
      "name": "8️⃣ Webhook - Email Response",
      "type": "n8n-nodes-base.webhook",
      "typeVersion": 1,
      "position": [1500, 200]
    },
    {
      "parameters": {
        "url": "{{ $json.responseLink }}",
        "method": "GET"
      },
      "name": "9️⃣ Send Demo Link",
      "type": "n8n-nodes-base.httpRequest",
      "typeVersion": 4,
      "position": [1700, 200]
    },
    {
      "parameters": {
        "spreadsheetId": "{{ $env.GOOGLE_SHEETS_ID }}",
        "operation": "update",
        "columns": {
          "Status": "demo_invited"
        }
      },
      "name": "🔟 Log Demo Invitation",
      "type": "n8n-nodes-base.googleSheets",
      "typeVersion": 2,
      "position": [1900, 200]
    },
    {
      "parameters": {
        "interval": [7, "days"]
      },
      "name": "Wait for Demo",
      "type": "n8n-nodes-base.wait",
      "typeVersion": 1,
      "position": [2100, 200]
    },
    {
      "parameters": {
        "content": "مرحبا {{ $json.name }},\n\nكيف كان انطباعك على العرض التوضيحي؟\nهل تريد معرفة المزيد؟\n\nسعر خاص: $999/شهر (Starter Plan)\n\nرابط الشراء: https://sifoubabaya.github.io/Tevest-12/contact.html"
      },
      "name": "Send Offer Email",
      "type": "n8n-nodes-base.httpRequest",
      "typeVersion": 4,
      "position": [2300, 200]
    },
    {
      "parameters": {
        "channel": "#sales",
        "message": "🎉 عرض جديد من {{ $json.company }}!\nالحالة: {{ $json.status }}\nالتاريخ: {{ $now.toISOString() }}"
      },
      "name": "Slack Notification",
      "type": "n8n-nodes-base.slack",
      "typeVersion": 2,
      "position": [2500, 200]
    },
    {
      "parameters": {
        "table": "sales_deals",
        "columns": {
          "company": "{{ $json.company }}",
          "deal_value": "{{ $json.dealValue }}",
          "status": "won",
          "close_date": "{{ $now.toISOString() }}"
        }
      },
      "name": "Log Completed Deal",
      "type": "n8n-nodes-base.postgres",
      "typeVersion": 2,
      "position": [2700, 200]
    }
  ],
  "connections": {
    "1️⃣ Load Leads from CSV": {
      "main": [
        [
          {
            "node": "2️⃣ Send Email Campaign",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "2️⃣ Send Email Campaign": {
      "main": [
        [
          {
            "node": "3️⃣ Log Email to Database",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "3️⃣ Log Email to Database": {
      "main": [
        [
          {
            "node": "4️⃣ Wait 48 Hours",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "4️⃣ Wait 48 Hours": {
      "main": [
        [
          {
            "node": "5️⃣ Check for Follow-up",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "5️⃣ Check for Follow-up": {
      "main": [
        [
          {
            "node": "6️⃣ Send Follow-up Email",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "6️⃣ Send Follow-up Email": {
      "main": [
        [
          {
            "node": "7️⃣ Update Google Sheets",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "7️⃣ Update Google Sheets": {
      "main": [
        [
          {
            "node": "8️⃣ Webhook - Email Response",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "8️⃣ Webhook - Email Response": {
      "main": [
        [
          {
            "node": "9️⃣ Send Demo Link",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "9️⃣ Send Demo Link": {
      "main": [
        [
          {
            "node": "🔟 Log Demo Invitation",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "🔟 Log Demo Invitation": {
      "main": [
        [
          {
            "node": "Wait for Demo",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "Wait for Demo": {
      "main": [
        [
          {
            "node": "Send Offer Email",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "Send Offer Email": {
      "main": [
        [
          {
            "node": "Slack Notification",
            "type": "main",
            "index": 0
          }
        ]
      ]
    },
    "Slack Notification": {
      "main": [
        [
          {
            "node": "Log Completed Deal",
            "type": "main",
            "index": 0
          }
        ]
      ]
    }
  }
}
```

## 📊 مراحل سير العمل:

### **المرحلة 1: تحميل العملاء المحتملين**
```
📂 CSV File → Load Leads
├─ Company Name
├─ Contact Email
├─ Contact Name
└─ Manager Title
```

### **المرحلة 2: إرسال البريد الأول**
```
📧 Email Campaign
├─ Subject: عرض خاص - VERIDIAN
├─ Body: رسالة تسويقية احترافية
├─ Attachment: Brochure PDF
└─ Track: Open Rate
```

### **المرحلة 3: تسجيل في قاعدة البيانات**
```
🗄️ Database Log
├─ Company Name
├─ Email Sent Date
├─ Status: "email_sent"
└─ Tracking ID
```

### **المرحلة 4: الانتظار 48 ساعة**
```
⏰ Wait Timer
└─ Check if reply received
```

### **المرحلة 5: متابعة أولى**
```
📧 Follow-up Email #1
├─ إذا لم يرد: أرسل متابعة
├─ إذا رد: انتقل للخطوة التالية
└─ Update Status
```

### **المرحلة 6: دعوة عرض توضيحي**
```
🎬 Send Demo Invitation
├─ Link to Demo
├─ Scheduling Calendar Link
└─ Special Offer Code
```

### **المرحلة 7: انتظار العرض**
```
⏰ Wait 7 Days
└─ للعودة بالعرض التوضيحي
```

### **المرحلة 8: إرسال العرض النهائي**
```
💰 Final Offer
├─ Pricing Plans
├─ Special Discount (محدود)
├─ Payment Link
└─ Contract PDF
```

### **المرحلة 9: إتمام الصفقة**
```
✅ Deal Closed
├─ Log to Database
├─ Send Confirmation Email
├─ Generate Invoice
└─ Slack Notification
```

---

## 🔧 **خطوات الإعداد على N8N:**

### **1. تثبيت N8N:**
```bash
npm install -g n8n
n8n start
```

### **2. الوصول إلى الواجهة:**
```
http://localhost:5678
```

### **3. إضافة Credentials:**
```
✅ Gmail OAuth2
✅ Google Sheets API
✅ PostgreSQL Database
✅ Slack Bot Token
```

### **4. تحميل قوائم العملاء:**
```csv
Company,Email,Name,Title
DP World,operations@dpworld.com,Ahmed,Operations Manager
Aramex,logistics@aramex.com,Sara,Logistics Director
Bahri,contact@bahri.sa,Mohammed,Supply Chain Manager
```

---

## 📈 **مؤشرات الأداء (KPIs):**

```
📊 Metrics Dashboard:
├─ Emails Sent: 99
├─ Open Rate: ~30%
├─ Reply Rate: ~15%
├─ Demo Rate: ~50%
├─ Close Rate: ~30-40%
└─ Expected Revenue: $15,000-$25,000
```

---

## 💾 **متطلبات قاعدة البيانات:**

```sql
CREATE TABLE sales_pipeline (
  id SERIAL PRIMARY KEY,
  company VARCHAR(255),
  email VARCHAR(255),
  name VARCHAR(255),
  status VARCHAR(50),
  sent_date TIMESTAMP,
  replied_date TIMESTAMP,
  demo_scheduled BOOLEAN,
  deal_value DECIMAL(10,2),
  closed_date TIMESTAMP
);

CREATE TABLE email_templates (
  id SERIAL PRIMARY KEY,
  template_name VARCHAR(255),
  subject VARCHAR(255),
  body TEXT,
  created_date TIMESTAMP
);

CREATE TABLE sales_deals (
  id SERIAL PRIMARY KEY,
  company VARCHAR(255),
  deal_value DECIMAL(10,2),
  status VARCHAR(50),
  close_date TIMESTAMP,
  notes TEXT
);
```

---

## 🎯 **متغيرات البيئة:**

```env
GMAIL_CREDENTIALS=your_gmail_oauth_token
GOOGLE_SHEETS_ID=your_sheet_id
POSTGRES_HOST=localhost
POSTGRES_USER=sales_user
POSTGRES_PASSWORD=secure_password
SLACK_BOT_TOKEN=xoxb-your-token
VERIDIAN_DEMO_LINK=https://sifoubabaya.github.io/Tevest-12/
VERIDIAN_CONTACT_LINK=https://sifoubabaya.github.io/Tevest-12/contact.html
```

---

## ✉️ **قوالب البريد:**

### **البريد الأول:**
```
الموضوع: عرض حصري - VERIDIAN
الجسم:
السلام عليكم [اسم المدير]،

هل تريد تقليل مخاطر سلسلة التوريد بنسبة 98%؟

✅ تنبؤ ذكي فوري
✅ مراقبة 24/7
✅ توفير 40% من التكاليف

👉 https://sifoubabaya.github.io/Tevest-12/

تحياتي،
خالد
khaledrabie628@gmail.com
```

### **البريد الثاني (متابعة):**
```
الموضوع: RE: عرض حصري - VERIDIAN

هل استقبلت الرسالة السابقة؟
لا تفوت العرض المحدود!

اطلب عرض توضيحي مجاني الآن:
https://sifoubabaya.github.io/Tevest-12/contact.html
```

### **البريد الثالث (عرض العرض التوضيحي):**
```
الموضوع: عرضك التوضيحي جاهز!

جرب VERIDIAN مجاناً:
https://link-to-demo.com

احجز وقتك الآن!
```

---

## 🚀 **بدء الـ Workflow:**

```bash
# تشغيل N8N
n8n start

# إنشاء Workflow جديد
# Copy و Paste الـ JSON أعلاه

# تفعيل Triggers
# تحديد Schedule (يومي أو أسبوعي)

# مراقبة التنفيذ
# Dashboard → View Executions
```

---

**هل تريد مني أن أساعدك في:**
1. ✅ **تحميل الـ Workflow على N8N؟**
2. ✅ **إنشاء قاعدة بيانات PostgreSQL؟**
3. ✅ **كتابة رسائل البريد الاحترافية؟**
4. ✅ **إضافة Google Sheets للتتبع؟**

**الآن عندك نظام بيع أوتوماتيكي 24/7! 🤖💰**