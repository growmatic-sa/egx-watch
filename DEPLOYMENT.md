# إصلاح نشر مؤشر البورصة

## GitHub Pages

الرابط أعاد HTTP 404 أثناء الفحص. ملف index.html موجود في جذر فرع main، لكن لم يتم التحقق من إعدادات Pages الخاصة بالحساب.

1. افتح https://github.com/growmatic-sa/egx-watch/settings/pages
2. اختر Deploy from a branch ثم main ثم /(root) واضغط Save.
3. ارفع محتويات مجلد egx-watch المرفق إلى جذر المستودع مع الحفاظ على مجلد icons. لا ترفع ملف ZIP نفسه أو مجلد egx-watch داخل المستودع.
4. انتظر نجاح النشر في Actions ثم افتح https://growmatic-sa.github.io/egx-watch/

النسخة الحالية على GitHub وضعت ملفات الأيقونات في الجذر، بينما HTML وmanifest وsw.js تطلبها داخل icons. الملفات المرفقة تحافظ على المسار الصحيح.

## خادم البيانات

الخادم https://egx-data.tahamahm3.workers.dev يعيد JSON بنجاح لكنه لا يرسل Access-Control-Allow-Origin عند طلبه من https://growmatic-sa.github.io. تعديل HTML وحده لا يصلح ذلك.

أضف الترويسات التالية على جميع استجابات Cloudflare Worker، بما فيها استجابات الأخطاء والكاش:

```javascript
const headers = new Headers(response.headers);
headers.set('Access-Control-Allow-Origin', 'https://growmatic-sa.github.io');
headers.set('Access-Control-Allow-Methods', 'GET, OPTIONS');
return new Response(response.body, {
  status: response.status,
  statusText: response.statusText,
  headers
});
```

response هنا هي الاستجابة النهائية في كود Worker الحالي. يجب دمج التعديل داخل معالج الطلب الأصلي؛ هذا المقتطف ليس بديلاً كاملاً عن Worker. كود Worker غير موجود في الأرشيف، ولذلك لم يتم تعديله أو نشره.

لا تستخدم mode: 'no-cors'؛ سيجعل JSON غير قابل للقراءة في المتصفح.

## ما تم إصلاحه محلياً

- توقف حالات التحميل بعد فشل الطلبات مع إظهار رسالة لكل سهم.
- مهلة 15 ثانية لكل طلب ومنع عمليات التحديث المتداخلة.
- التحقق من قائمة المتابعة المخزنة محلياً.
- إزالة محارف \\n الظاهرة خارج HTML.
- إصدار جديد للكاش مع عدم حذف كاش التطبيقات الأخرى على نفس النطاق.

لا تحتوي النسخة على إصلاح منشور لخادم الأسعار؛ اكتمال ظهور البيانات يعتمد على تعديل CORS في Worker.
