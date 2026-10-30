<div align="center">

# واریزت · Varizet

<p dir="rtl"><b>هر واریز، خودکار تایید.</b><br>تایید خودکار پرداخت کارت‌به‌کارت برای کسب‌وکارهای آنلاین ایرانی</p>

<p><b>Every deposit, confirmed automatically.</b><br>Automated card-to-card payment confirmation for Iranian online businesses.</p>

</div>

<div dir="rtl">

### چطور کار می‌کند

**۱.** مشتری مبلغ یکتای سفارش را از اپ بانک خودش به کارت فروشنده واریز می‌کند.\
**۲.** واریزت پیامک یا اعلان بانک را تشخیص می‌دهد و واریز را با سفارش تطبیق می‌دهد.\
**۳.** سفارش در چند ثانیه تایید می‌شود و سایت شما خبرش را با وب‌هوک امضاشده می‌گیرد.

نه رسیدی لازم است، نه بررسی دستی. مبلغ‌ها در API همیشه عدد صحیح ریال هستند.

**محیط تست (Sandbox):** با کلید تست (<span dir="ltr">`vz_test_…`</span>) همه‌چیز واقعی است جز پول. سفارش بسازید و واریز را با `POST /v1/test/deposits` شبیه‌سازی کنید؛ واریز شبیه‌سازی‌شده از همان موتور تطبیق می‌گذرد و وب‌هوک واقعی می‌فرستد.

**امنیت:** هر وب‌هوک هدر `Varizet-Signature` دارد (HMAC-SHA256 با پنجره زمانی ۵ دقیقه). پیش از هر اقدام، بدنه خام درخواست را با SDK تایید کنید. آسیب‌پذیری‌ها را در issue عمومی ننویسید؛ آن‌ها را به‌صورت خصوصی به نشانی support@varizet.ir بفرستید.

**مشارکت:** گزارش خطا و Pull Request در مخزن هر SDK و افزونه پذیرفته می‌شود.

</div>

### How it works

1. The customer transfers the order's unique amount from their own bank app to the merchant's card.
2. Varizet detects the bank SMS or notification and matches the deposit to the order.
3. The order is confirmed within seconds, and your site learns about it from a signed webhook.

No receipts to upload, no manual checks. Amounts in the API are always integer rial.

**Sandbox:** with a test key (`vz_test_…`) everything is real except the money. Create orders and simulate a deposit with `POST /v1/test/deposits`; it runs through the same matching engine and sends real webhooks.

**Security:** every webhook carries a `Varizet-Signature` header (HMAC-SHA256, 5-minute window). Verify the raw request body with an SDK before acting on it. Report vulnerabilities privately to support@varizet.ir, never in public issues.

**Contributing:** issues and pull requests are welcome on every SDK and plugin repository.

## مخزن‌ها · Repositories

| Repository · Platform | توضیح · Description | Install ¹ |
| --- | --- | --- |
| [varizet-php](https://github.com/Varizet/varizet-php)<br>PHP 8.1+ | <span dir="rtl">SDK رسمی PHP: سفارش، وب‌هوک، Sandbox</span><br>Official PHP SDK: orders, webhooks, sandbox | Composer |
| [varizet-laravel](https://github.com/Varizet/varizet-laravel)<br>Laravel 10–12 | <span dir="rtl">یکپارچه‌سازی لاراول: Facade، وب‌هوک، رویدادها</span><br>Laravel integration: facade, webhooks, events | Composer |
| [varizet-python](https://github.com/Varizet/varizet-python)<br>Python 3.10+ | <span dir="rtl">SDK رسمی پایتون: sync و async، Django و FastAPI</span><br>Official Python SDK: sync/async, Django, FastAPI | pip |
| [varizet-woocommerce](https://github.com/Varizet/varizet-woocommerce)<br>WordPress 6.4+ | <span dir="rtl">افزونه وردپرس: ووکامرس و Easy Digital Downloads</span><br>WordPress: WooCommerce and Easy Digital Downloads | Release zip → Plugins › Add New › Upload |
| [varizet-whmcs](https://github.com/Varizet/varizet-whmcs)<br>WHMCS 8 | <span dir="rtl">درگاه WHMCS برای فاکتورها و تمدید سرویس‌ها</span><br>WHMCS gateway for invoices and renewals | Release zip → WHMCS root |
| [varizet-opencart](https://github.com/Varizet/varizet-opencart)<br>OpenCart 3.0.x, 4.0.2+ | <span dir="rtl">افزونه پرداخت اوپن‌کارت</span><br>OpenCart payment extension | `.ocmod.zip` → Extensions › Installer |
| [varizet-prestashop](https://github.com/Varizet/varizet-prestashop)<br>PrestaShop 8 | <span dir="rtl">ماژول پرداخت پرستاشاپ</span><br>PrestaShop payment module | Release zip → Module Manager |
| Node.js SDK<br>Node.js | <span dir="rtl">SDK رسمی Node.js با تایپ‌های TypeScript</span><br>Official Node.js SDK with TypeScript types | <span dir="rtl">به‌زودی</span> · Coming soon |

<div dir="rtl">

¹ هنوز در Packagist، PyPI و WordPress.org منتشر نشده‌ایم؛ فعلاً از Releases یا سورس همین مخزن‌ها نصب کنید. مجوز همه مخزن‌ها MIT است، جز افزونه وردپرس (GPL-2.0-or-later).

</div>

¹ Not on Packagist, PyPI or WordPress.org yet: install from Releases or source. All repositories are MIT-licensed, except the WordPress plugin (GPL-2.0-or-later).

```bash
composer config repositories.varizet-php vcs https://github.com/Varizet/varizet-php
composer require varizet/varizet-php
# Laravel: add varizet-laravel the same way, then: composer require varizet/laravel
pip install "git+https://github.com/Varizet/varizet-python"
```

## شروع سریع · Quick start

```php
use Varizet\VarizetClient;
use Varizet\Webhook\Webhook;
use Varizet\Exception\{SignatureVerificationException, InvalidPayloadException};

$varizet = new VarizetClient(getenv('VARIZET_API_KEY'));   // vz_test_… in the sandbox

// Checkout: create the order (integer rial) and redirect to the payment page
$order = $varizet->orders->create([
    'amount'            => 8_500_000,
    'merchant_order_id' => 'INV-12345',
    'return_url'        => 'https://shop.example/checkout/return',
], 'order-INV-12345-1');                      // Idempotency-Key
header('Location: ' . $order->paymentUrl);   // unique amount + destination card

// Webhook endpoint: verify the raw body (HMAC-SHA256, ±300 s) before trusting it
try {
    $event = Webhook::constructEvent(file_get_contents('php://input'),
        Webhook::headerFromServer($_SERVER), getenv('VARIZET_WEBHOOK_SECRET'));
} catch (SignatureVerificationException $e) {        // malformed | expired | mismatch
    http_response_code(401); exit;
} catch (InvalidPayloadException $e) {               // signed, but not an event
    http_response_code(400); exit;
}
if ($event->order()?->isPaid()) {
    // mark INV-12345 paid; dedupe on $event->id
}
http_response_code(200);
```

## پیوندها · Links

[varizet.ir](https://varizet.ir) · [مستندات توسعه‌دهندگان · Developer docs](https://varizet.ir/developers) · API: `https://api.varizet.ir/v1` · <span dir="rtl">پشتیبانی</span> · Support: [support@varizet.ir](mailto:support@varizet.ir)
