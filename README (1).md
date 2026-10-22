# DeepSeek API

هذا الملف يشرح استخدام API الجديد فقط.

## الرابط

الرابط الحالي على Replit:

```text
https://25a2142e-23fc-4958-b134-a247420f2c28-00-txdf76800yvm.archer.replit.dev/api/max
```

لم يتغير الدومين. استخدم هذا الرابط نفسه في الطلبات.

## إرسال رسالة جديدة

استخدم `POST` وأرسل النص داخل جسم الطلب بصيغة JSON:

```bash
curl -X POST \
  "https://25a2142e-23fc-4958-b134-a247420f2c28-00-txdf76800yvm.archer.replit.dev/api/max" \
  -H "Content-Type: application/json" \
  --data-binary '{"prompt":"مرحبا، اشرح لي هذا الموضوع بالتفصيل"}'
```

استخدام `POST` مهم للنصوص الطويلة؛ لا تضع النص داخل الرابط بعد `?prompt=` لأن طول عناوين URL محدود.

## إرسال نص طويل

يمكن إرسال نص كبير بالطريقة نفسها:

```bash
curl -X POST \
  "https://25a2142e-23fc-4958-b134-a247420f2c28-00-txdf76800yvm.archer.replit.dev/api/max" \
  -H "Content-Type: application/json" \
  --data-binary @request.json
```

محتوى ملف `request.json`:

```json
{
  "prompt": "ضع النص الطويل هنا"
}
```

يدعم API جسم طلب يصل إلى 8MB. الحد النهائي للنص يعتمد على قدرة صفحة DeepSeek وسياق المحادثة الحالي.

## استكمال نفس المحادثة

بعد أول طلب، يحفظ API معرف المحادثة `chat_uid`. أرسله مع الرسالة التالية داخل JSON:

```bash
curl -X POST \
  "https://25a2142e-23fc-4958-b134-a247420f2c28-00-txdf76800yvm.archer.replit.dev/api/max" \
  -H "Content-Type: application/json" \
  --data-binary '{
    "prompt": "تابع من حيث انتهيت",
    "chat_uid": "ضع_chat_uid_هنا"
  }'
```

## الحصول على `chat_uid`

عند إنشاء محادثة جديدة، يعيد API المعرف بطريقتين:

1. في ترويسة الاستجابة:

   ```text
   X-Chat-UID
   ```

2. في بداية نص الاستجابة الجديدة:

   ```text
   chat_uid=...
   ```

احتفظ بهذا المعرف واستخدمه في الطلبات التالية للحفاظ على نفس المحادثة.

## مثال JavaScript

```javascript
const response = await fetch(
  "https://25a2142e-23fc-4958-b134-a247420f2c28-00-txdf76800yvm.archer.replit.dev/api/max",
  {
    method: "POST",
    headers: {
      "Content-Type": "application/json",
    },
    body: JSON.stringify({
      prompt: "اكتب الإجابة هنا، ويمكن أن تكون طويلة جدًا",
      // chat_uid: "للاستكمال فقط",
    }),
  },
);

const chatUid = response.headers.get("x-chat-uid");
const answer = await response.text();

console.log({ chatUid, answer });
```

## الاستجابات والأخطاء

- `200`: تم استلام رد DeepSeek.
- `400`: الطلب ناقص أو لا يحتوي على `prompt` صالح.
- `404`: قيمة `chat_uid` غير موجودة.
- `500`: حدث خطأ أثناء تشغيل المتصفح أو الأتمتة.
- `503`: طابور الأتمتة يحتاج إلى إعادة المحاولة.
- `504`: لم يصل رد مستقر من DeepSeek ضمن المهلة المحددة.

## التوافق مع GET

ما زال هذا الشكل يعمل مع الرسائل القصيرة:

```text
GET /api/max?prompt=مرحبا
```

لكن استخدم `POST` دائمًا للنصوص الطويلة أو عند استكمال محادثة باستخدام `chat_uid`.