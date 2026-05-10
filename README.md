# 📦 Chillers Survey — El Wagd

تطبيق ويب لتسجيل زيارات الثلاجات للعملاء، مع تخزين البيانات على Firebase ونشره على GitHub Pages.

## 📁 محتويات الريبو

```
your-repo/
├── index.html          ← التطبيق الرئيسي
├── chillers.json       ← قائمة الـ 1,434 ثلاجة (Master)
└── README.md
```

---

## 🚀 خطوات النشر (مرة واحدة بس)

### 1) إنشاء مشروع Firebase

1. ادخل على https://console.firebase.google.com
2. **Add Project** → اختار اسم (مثلاً: `chillers-survey-elwagd`) → **Continue** → **Create Project**
3. من القائمة الجانبية: **Build → Realtime Database → Create Database**
   - اختار **Singapore (asia-southeast1)** أو أقرب location
   - ابدأ في **Test mode** (هتعدل القواعد بعدين)
4. من **Project Settings (⚙️) → General → Your apps → Web (`</>`)**:
   - اكتب nickname (مثلاً: `survey-web`) → **Register app**
   - **انسخ الـ `firebaseConfig`** (هتلصقه في `index.html`)

### 2) تعديل `index.html`

افتح `index.html` وابحث عن السطر ده:

```js
const firebaseConfig = {
  apiKey: "REPLACE_API_KEY",
  authDomain: "REPLACE.firebaseapp.com",
  databaseURL: "https://REPLACE-default-rtdb.firebaseio.com",
  ...
};
```

استبدله بالـ config اللي نسخته من Firebase.

### 3) قواعد الأمان للـ Realtime Database

من Firebase Console → **Realtime Database → Rules**، استخدم القواعد دي:

```json
{
  "rules": {
    "surveys": {
      ".read": true,
      ".write": true,
      ".indexOn": ["truck", "status", "ts"]
    }
  }
}
```

> ⚠️ القواعد دي تسمح لأي حد يقرا ويكتب — مناسبة للاستخدام الداخلي. لو محتاج أمان أعلى لاحقاً، نضيف Authentication.

### 4) إنشاء ريبو على GitHub

1. https://github.com/new → اختار اسم (مثلاً: `chillers-survey`) → **Public**
2. ارفع الملفات الـ 3 (`index.html` + `chillers.json` + `README.md`)
3. **Settings → Pages → Source: Deploy from branch → main → / (root) → Save**
4. بعد دقيقتين هيكون الرابط:
   ```
   https://YOUR_USERNAME.github.io/chillers-survey/
   ```

---

## 📱 طريقة الاستخدام

### للمندوب
1. يفتح الرابط من الموبايل
2. يفلتر بالعربية بتاعته (Truck) من أعلى الصفحة
3. يدوس على أي ثلاجة في القائمة → تفتح شاشة السيرفي
4. يضغط **📡 تحديد موقعي** عشان يأكد إنه فعلاً في المكان (بيظهر المسافة بين موقعه والثلاجة)
5. يختار **حالة الزيارة** (تم / لم يتم) → يختار **السبب** → يكتب **الملاحظات**
6. **💾 حفظ** → البيانات بتترفع على Firebase فوراً

### للمشرف
- **📋 القائمة**: كل الثلاجات + فلترة حسب العربية/الموزع/الحالة + بحث
- **🗺️ الخريطة**: عرض كل الثلاجات على الخريطة بألوان حسب الحالة (أخضر=تم، أصفر=لم يتم، رمادي=معلق)
- **📊 التقدم**: نسبة إنجاز كل عربية + Export Excel

---

## 🔄 تحديث قائمة الثلاجات

لو فيه تعديلات على الـ Master File:

1. شغل سكريبت تحويل Excel إلى JSON (موجود تحت)
2. ارفع `chillers.json` الجديد على نفس الريبو
3. التطبيق هيقرا التحديث تلقائي (`?v=` cache buster موجود في الكود)

### سكريبت تحويل Excel → JSON

```python
import pandas as pd
import json

df = pd.read_excel('Cairo_El_Wagdi___Haram_Chillers_Survey.xlsx', sheet_name='Sheet1')

chillers = []
for _, row in df.iterrows():
    chillers.append({
        "id": int(row['#']),
        "distributor": str(row['Distributor']).strip(),
        "cust_code": str(row['Cust_Code']).strip(),
        "truck": str(row['Truck']).strip().upper() if pd.notna(row['Truck']) else "UNASSIGNED",
        "customer_name": str(row['Customer Name']).strip(),
        "chiller_code": str(row['Chiller Code']).strip(),
        "address": str(row['Address']).strip(),
        "lng": float(row['Longitude']),
        "lat": float(row['Latitude']),
        "maps_url": str(row['Location']).strip()
    })

with open('chillers.json', 'w', encoding='utf-8') as f:
    json.dump(chillers, f, ensure_ascii=False, indent=2)

print(f"Generated {len(chillers)} chillers")
```

---

## 🛠️ التقنيات المستخدمة

- **HTML/CSS/JavaScript** (vanilla — بدون build tools)
- **Firebase Realtime Database** — تخزين بيانات الزيارات
- **Leaflet.js** — عرض الخريطة
- **SheetJS (xlsx)** — تصدير Excel
- **GitHub Pages** — استضافة مجانية

---

## ⚠️ ملاحظات مهمة

### حدود Firebase Realtime Database (الخطة المجانية Spark)
- **مساحة:** 1 GB
- **bandwidth شهري:** 10 GB
- **اتصالات متزامنة:** 100

البيانات صغيرة جداً (نص فقط) فالخطة المجانية كافية جداً.

### النسخ الاحتياطي
البيانات على Firebase آمنة، لكن دايماً اعمل **Export Excel** أسبوعياً واحفظه في مكان منفصل.

---

## 🐛 مشاكل شائعة

| المشكلة | الحل |
|---|---|
| التطبيق بيقول "فشل تحميل قائمة الثلاجات" | تأكد إن `chillers.json` في نفس مجلد `index.html` على GitHub |
| البيانات مش بتتحفظ | افتح Console (F12) → شوف الـ error. غالباً قواعد Firebase مش مظبوطة |
| GPS مش شغال | لازم الموقع يكون **HTTPS** (GitHub Pages بـ HTTPS تلقائي ✓) + إعطاء إذن للموقع من المتصفح |

---

محتاج أي تعديل؟ قولي وهعدل.
