# AZ Iranian Bank Gateway Framework config

<p dir="rtl">
 کدهای آزاد و متن باز به زبان پایتون (python) که برای ارتباط با درگاه های بانکهای ایرانی در جنگو (Django) توسعه داده شده است.
 
 
[[_TOC_]]

<h1 dir="rtl">نصب</h1>

<p dir="rtl"> نصب از طریق پکیج منیجر </p>

``pip install az-iranian-bank-gateways``


<h1 dir="rtl">تنظیمات</h1>
 
### settings.py

<p dir="rtl"> در فایل `settings.py` تنظیمات زیر را انجام میدهیم. </p>

 
 ``` python

 AZ_IRANIAN_BANK_GATEWAYS = {
    'CHANNELS': {
        'BMI': {
            'PATH': 'azbankgateways.banks.BMI',
            'MERCHANT_CODE': '<YOUR MERCHANT CODE>',
            'TERMINAL_CODE': '<YOUR TERMINAL CODE>',
            'SECRET_KEY': '<YOUR SECRET KEY>',
        },
    },
    'DEFAULT': 'BMI',
    'CURRENCY': 'IRR', 
    'TRACKING_CODE_QUERY_PARAM': 'tc',
    'TRACKING_CODE_LENGTH': 20,
}
 ```

1. <p dir="rtl">  `CHANNELS` :  تنظیمات مربوط به هر بانک به صورت دیکشنری های جدا در این قسمت وجود دارد. تنظیماتی مانند کلاس اجرا کننده، کلیدهای امنیتی که توسط بانک در اختیار شما قرار می گیرد. </p>

1. <p dir="rtl">  `DEFAULT`: در زمانی که به سازنده فکتوری پارامتری ارسال نشود از این تنظیم به عنوان بانک پیش فرض استفاده خواهد شد و ارتباطات با این بانک برقرار می شود. </p>
 
1. <p dir="rtl"> `CURRENCY`: واحد پولی که نرم افزار با آن کار می کند. این واحد پولی فارغ از واحد پولی درگاه خواهد بود. واحد های پولی مجاز `IRR` و `IRT` هستند که در صورتی که واحد پولی نرم افزار با واحد پولی درگاه بانک متفاوت باشد تبدیل ریال به تومان یا بالعکس انجام خواهد شد.  </p>
 
1. <p dir="rtl"> `TRACKING_CODE_QUERY_PARAM `: پارامتری که در هنگام بازگشت از درگاه به کال بک یو آر تعیین شده تنظیم و ارسال می گردد. به عنوان مثال زمانی که از کاربر از درگاه بانک باز می گردد چه پرداخت موفق داشته باشد و چه نا موفق کاربر به لینکی که در هنگام استفاده از درگاه تنظیم می شود ارجاع داده می شود و در انتهای آن این رشته + کد پیگیری بازگردانده می شود تا بتوان داده ها را از این طریق بازیابی کرد.  </p>
  
1. <p dir="rtl"> `TRACKING_CODE_LENGTH`: طول کد پیگیری تولید شده توسط سیستم است. دقت شود که در برخی درگاه ها مانند درگاه بانک ملی ایران طول ۲۰ کاراکتر خطای شماره سفارش ارسال نشده است را می دهد. </p>
 

### urls.py

<p dir="rtl">
 در فایل `urls.py`
</p>

```python
from django.contrib import admin
from django.urls import path

from azbankgateways.urls import az_bank_gateways_urls

admin.autodiscover()

urlpatterns = [
    path('admin/', admin.site.urls),
    path('bankgateways/', az_bank_gateways_urls()),
]
```
<p dir="rtl">
با اضافه کردن `path('bankgateways/', az_bank_gateways_urls()),` به لیست یو آر ال ها، پرداخت ها پس از درگاه به این مسیر هدایت و اعتبار سنجی می شوند و سپس مجدد به سمت کال بکی که به ازای هر درخواست تنظیم می شود، مسیر یابی خواهد شد.
</p>


<h1 dir="rtl">نحوه استفاده</h1>
<h2 dir="rtl">ارسال به بانک</h2>


<p dir="rtl">
 برای استفاده و اتصال به درگاه بانک کافی است یک `BankFactory` ایجاد کنیم و پارامترهای اجباری را تنظیم کنیم. سپس کاربر را می توانیم به درگاه بانک هدایت  کنیم.
</p>

  
```python
"""
BankFactory()  می توانید به صورت دیفالت نیز استفاده کنید که از بانک پیش فرض استفاده خواهد کرد.
یا اینکه بانک مورد نظر را در هنگام ساخت فکتوری به آن ارسال کنید.
"""
from django.urls import reverse
from azbankgateways import bankfactories, models as bank_models, default_settings as settings



def go_to_gateway_view(request):
    # خواندن مبلغ از هر جایی که مد نظر است
    amount = 1000
    # تنظیم شماره موبایل کاربر از هر جایی که مد نظر است
    user_mobile_number = '+989112221234'  # اختیاری

    factory = bankfactories.BankFactory()  # or bankfactories.BankFactory(bank_models.BankType.BMI)
    bank = factory.create()
    bank.set_request(request)
    bank.set_amount(amount)
    # یو آر ال بازگشت به نرم افزار برای ادامه فرآیند
    bank.set_client_callback_url(reverse('callback_gateway_view'))
    bank.set_mobile_number(user_mobile_number)  # اختیاری

    # در صورت تمایل اتصال این رکورد به رکورد فاکتور یا هر چیزی که بعدا بتوانید ارتباط بین محصول یا خدمات را با این
    # پرداخت برقرار کنید. 
    bank_record = bank.ready()
    
    # هدایت کاربر به درگاه بانک
    return bank.redirect_gateway()

```

<p dir="rtl">
`set_mobile_number` پارامتری است که شماره موبایل کاربری که قصد خرید دارد را با آن تنظیم کرده و این شماره موبایل جهت پرداخت و پیگیری آسان تر به درگاه ارسال می شود
</p>

<h2 dir="rtl">بازگشت از بانک</h2>

<p dir="rtl"> 
وضعیت رکورد بانک به شرح ذیل می باشد.
 </p>

<p dir="rtl"> 
`WAITING`: در انتظار برای انتقال کاربر به درگاه بانک
 </p>


<p dir="rtl"> 
`REDIRECT_TO_BANK`: کاربر به درگاه بانک منتقل شده است ولی هنوز از درگاه باز نگشته است.
 </p>


<p dir="rtl"> 
`RETURN_FROM_BANK`: کاربر از درگاه برگشته ولی عملیات صحت سنجی یا تکمیل نشده است یا با خطا درهنگام تایید از سوی بانک مواجه شده است. در این شرایط می توان با فراخوانی مجدد در بازه زمانی کمتر از ۱۵ دقیقه که کاربر بازگشته عملیات تایید را مجدد درخواست داد. شرح تایید مجدد در پایین تر آورده شده است. 
 </p>
 

<p dir="rtl"> 
`CANCEL_BY_USER`: پرداخت توسط کاربر کنسل شده است. </p>


<p dir="rtl"> 
`EXPIRE_GATEWAY_TOKEN`: ارتباط با درگاه بانک برقرار شده ولی کاربر به درگاه هدایت نشده است. </p>


<p dir="rtl"> 
`EXPIRE_VERIFY_PAYMENT`: در بازه زمانی ۱۵ دقیقه پس از بازگشت موفق به تایید اطلاعات پرداخت نشده ایم. </p>


<p dir="rtl"> 
`COMPLETE`: وضعیت پرداخت موفق است.</p>


```python
import logging

from django.http import HttpResponse, Http404
from django.urls import reverse

from azbankgateways import bankfactories, models as bank_models, default_settings as settings


def callback_gateway_view(request):
    tracking_code = request.GET.get(settings.TRACKING_CODE_QUERY_PARAM, None)
    if not tracking_code:
        logging.debug("این لینک معتبر نیست.")
        raise Http404

    try:
        bank_record = bank_models.Bank.objects.get(tracking_code=tracking_code)
    except bank_models.Bank.DoesNotExist:
        logging.debug("این لینک معتبر نیست.")
        raise Http404

    # در این قسمت باید از طریق داده هایی که در بانک رکورد وجود دارد، رکورد متناظر یا هر اقدام مقتضی دیگر را انجام دهیم
    if bank_record.is_success:
        # پرداخت با موفقیت انجام پذیرفته است و بانک تایید کرده است.
        # می توانید کاربر را به صفحه نتیجه هدایت کنید یا نتیجه را نمایش دهید.
        return HttpResponse("پرداخت با موفقیت انجام شد.")

    # پرداخت موفق نبوده است. اگر پول کم شده است ظرف مدت ۴۸ ساعت پول به حساب شما بازخواهد گشت.
    return HttpResponse("پرداخت با شکست مواجه شده است. اگر پول کم شده است ظرف مدت ۴۸ ساعت پول به حساب شما بازخواهد گشت.")
```


<h2 dir="rtl">درخواست تایید مجدد از بانک</h2>

```python
import logging
from azbankgateways import bankfactories, models as bank_models, default_settings as settings

factory = bankfactories.BankFactory()  # or bankfactories.BankFactory(bank_models.BankType.BMI)

# غیر فعال کردن رکورد های قدیمی
bank_models.Bank.objects.update_expire_records()

# مشخص کردن رکوردهایی که باید تعیین وضعیت شوند
for item in bank_models.Bank.objects.filter_return_from_bank():
	bank = factory.create()
	bank.verify(item.tracking_code)		
	bank_record = bank_models.Bank.objects.get(tracking_code=item.tracking_code)
	if bank_record.is_success:
		logging.debug("This record is verify now.", extra={'pk': bank_record.pk})
    
```


# TODO

- [X] Bank model structure

- [X] BMI gateway support

- [X] Documentation

- [ ] Zarinpal gateway support

- [ ] Saman gateway support

