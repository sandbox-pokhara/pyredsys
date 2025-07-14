# pyredsys

Typed RedSys integration in python

## Installation

You can install the package via pip:

```
pip install pyredsys
```

## Request Payment

```python
import secrets

from pydantic import HttpUrl

from pyredsys.form import render_form
from pyredsys.payment import request_payment
from pyredsys.types import MerchantParameters

secret_key = "sq7HjrUOBfKmC576ILgskD5srU870gJ7"


signed_params = request_payment(
    secret_key,
    MerchantParameters(
        DS_MERCHANT_ORDER=secrets.token_hex(6),
        DS_MERCHANT_MERCHANTCODE=263100000,
        DS_MERCHANT_TERMINAL=38,
        DS_MERCHANT_CURRENCY=978,
        DS_MERCHANT_TRANSACTIONTYPE=0,
        DS_MERCHANT_AMOUNT=249,
        DS_MERCHANT_MERCHANTURL=HttpUrl(
            "https://webhook.site/3b23296d-ec3e-4438-a173-57ef09342352"
        ),
        DS_MERCHANT_URLOK=HttpUrl("https://example.com/success"),
        DS_MERCHANT_URLKO=HttpUrl("https://example.com/failure"),
        DS_MERCHANT_CONSUMERLANGUAGE=2,  # English
    ),
)

# NOTE: This is only for testing purposes, this html will contain a form
# with submit button that redirects to RedSys checkout
with open("form.html", "w") as fp:
    fp.write(render_form(signed_params))

# <form name="from"
#       action="https://sis-t.redsys.es:25443/sis/realizarPago"
#       method="post">
#        <input type="hidden" name="Ds_SignatureVersion" value="HMAC_SHA512_V1" />
#        <input type="hidden"
#               name="Ds_MerchantParameters"
#               value="eyJEU19NRVJDSEFOVF9PUkRFUiI6ImRjZmQ0YzhlNDBhYyIsIkRTX01FUkNIQU5UX01FUkNIQU5UQ09ERSI6IjI2MzEwMDAwMCIsIkRTX01FUkNIQU5UX1RFUk1JTkFMIjoiMzgiLCJEU19NRVJDSEFOVF9DVVJSRU5DWSI6Ijk3OCIsIkRTX01FUkNIQU5UX1RSQU5TQUNUSU9OVFlQRSI6IjAiLCJEU19NRVJDSEFOVF9BTU9VTlQiOiIyNDkiLCJEU19NRVJDSEFOVF9NRVJDSEFOVFVSTCI6Imh0dHBzOi8vd2ViaG9vay5zaXRlLzNiMjMyOTZkLWVjM2UtNDQzOC1hMTczLTU3ZWYwOTM0MjM1MiIsIkRTX01FUkNIQU5UX1VSTE9LIjoiaHR0cHM6Ly9leGFtcGxlLmNvbS9zdWNjZXNzIiwiRFNfTUVSQ0hBTlRfVVJMS08iOiJodHRwczovL2V4YW1wbGUuY29tL2ZhaWx1cmUiLCJEU19NRVJDSEFOVF9DT05TVU1FUkxBTkdVQUdFIjoyfQ==" />
#        <input type="hidden"
#               name="Ds_Signature"
#               value="azUk_BaFWkeeYsxEZOtBT20Y6wAnaY6iIvbNwCoMXI827bGZ2eH9VtqY1XaxLc1TVVijlV0PB0yqXvbaxunlAg==" />
#        <input type="submit" />
# </form>
```

## Validating a Notification

After the customer makes completes the checkout, a notification is sent to your DS_MERCHANT_MERCHANTURL which contains form data `Ds_SignatureVersion`, `Ds_MerchantParameters`, `Ds_Signature`. It is important to verify the signature to make sure that the data is not tampered with. `validate_notification` validates the notification from RedSys and also returns a typed pydantic object.

```py
from pyredsys.notification import validate_notification

secret = "sq7HjrUOBfKmC576ILgskD5srU870gJ7"


notif = validate_notification(
    "sq7HjrUOBfKmC576ILgskD5srU870gJ7",
    "eyJEc19EYXRlIjoiMTQlMkYwNyUyRjIwMjUiLCJEc19Ib3VyIjoiMDclM0E1NCIsIkRzX1NlY3VyZVBheW1lbnQiOiIxIiwiRHNfQ2FyZF9Db3VudHJ5IjoiNzI0IiwiRHNfQW1vdW50IjoiMjQ5IiwiRHNfQ3VycmVuY3kiOiI5NzgiLCJEc19PcmRlciI6ImViODU0OWRmNzhlYiIsIkRzX01lcmNoYW50Q29kZSI6IjI2MzEwMDAwMCIsIkRzX1Rlcm1pbmFsIjoiMDM4IiwiRHNfUmVzcG9uc2UiOiIwMDAwIiwiRHNfTWVyY2hhbnREYXRhIjoiIiwiRHNfVHJhbnNhY3Rpb25UeXBlIjoiMCIsIkRzX0NvbnN1bWVyTGFuZ3VhZ2UiOiIyIiwiRHNfQXV0aG9yaXNhdGlvbkNvZGUiOiI1MzU1NTkiLCJEc19DYXJkX0JyYW5kIjoiMSIsIkRzX0NhcmRfVHlwb2xvZ3kiOiJDT05TVU1PIiwiRHNfTWVyY2hhbnRfQ29mX1R4bmlkIjoiMjUwNzE0MDc1NDM1MCIsIkRzX1Byb2Nlc3NlZFBheU1ldGhvZCI6Ijc4IiwiRHNfQ29udHJvbF8xNzUyNDcyNDc1Mzk3IjoiMTc1MjQ3MjQ3NTM5NyJ9",
    "ZtGJrxieB-55ac4yso-WiRDvvS2yApveuh0u4O-AUYMiImw73N8iQkcGSpPIgaG3lSVMn5gMlFMBKomNf9GoLQ==",
    "HMAC_SHA512_V1",
)
print(notif.Ds_Date)  # 14/07/2025
print(notif.Ds_Hour)  # 07:54
print(notif.Ds_SecurePayment)  # 1
print(notif.Ds_Card_Country)  # 724
print(notif.Ds_Amount)  # 249
print(notif.Ds_Currency)  # 978
print(notif.Ds_Order)  # eb8549df78eb
print(notif.Ds_MerchantCode)  # 263100000
print(notif.Ds_Terminal)  # 38
print(notif.Ds_Response)  # 0
print(notif.Ds_MerchantData)
print(notif.Ds_TransactionType)  # 0
print(notif.Ds_ConsumerLanguage)  # 2
print(notif.Ds_AuthorisationCode)  # 535559
print(notif.Ds_Card_Brand)  # 1
print(notif.Ds_Card_Typology)  # CONSUMO
print(notif.Ds_Merchant_Cof_Txnid)  # 2507140754350
print(notif.Ds_ProcessedPayMethod)  # 78
print(notif.Ds_Control_1752472475397)  # 1752472475397
```

## License

This project is licensed under the terms of the MIT license.
