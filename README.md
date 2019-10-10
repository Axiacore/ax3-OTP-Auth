# AX3 OTP Auth

AX3 OTP Auth is a very simple Django library for generating and verifying one-time passwords using HTOP guidelines.

## Installation

Axes is easy to install from the PyPI package:

    $ pip install django-axes

After installing the package, the project settings need to be configured.

**1.** Add ``ax3_OTP`` to your ``INSTALLED_APPS``::

    INSTALLED_APPS = [
        'django.contrib.admin',
        'django.contrib.auth',
        'django.contrib.contenttypes',
        'django.contrib.sessions',
        'django.contrib.messages',
        'django.contrib.staticfiles',

        # Axes app can be in any position in the INSTALLED_APPS list.
        'ax3_OTP_Auth',
    ]

**2.** Add ``ax3_OTP_Auth.backends.OTPAuthBackend`` to the top of ``AUTHENTICATION_BACKENDS``:

    AUTHENTICATION_BACKENDS = [
        'ax3_OTP_Auth.backends.OTPAuthBackend',

        # Django ModelBackend is the default authentication backend.
        'django.contrib.auth.backends.ModelBackend',
    ]

**3.** Add the following to your urls.py:

    urlpatterns = [
        path('OTP-Auth/', include('ax3_OTP_Auth.urls', namespace='otp_auth')),
    ]

**4.** Create html button to your template:

    <button class="js-otp-auth" type="button" otp-login="{% url 'otp_auth:start' %}" otp-redirect="{% url 'login' %}">
        Login
    </button>

**5.** Create Javascript for open OTP window:

    $(() => {
        $('.js-otp-auth').on('click', function() {
            let redirect = $(this).attr('otp-redirect');
            let OTPLoginUrl = $(this).attr('otp-login');

            window.open(`${window.origin}${OTPLoginUrl}?redirect=${redirect}`, '_blank', 'location=yes,height=470,width=420,scrollbars=yes,status=yes');
        });
    });

## Configuration

If your need pass any param for whole pipeline you can use `OTP_AUTH_PARAMS`:

    `OTP_AUTH_PARAMS = ['param']`

If your need change life time cache value you can use `OTP_AUTH_TTL`:

    `OTP_AUTH_TTL = 60 * 60 * 5  # 5 minutes`

If your need change sms message:

    `OTP_AUTH_MESSAGE = 'Utiliza {} como código de inicio de sesión.`

Configure countries allowed list:
    COLOMBIA = 57
    ARGENTINA = 54
    BOLIVIA = 591
    CHILE = 56
    COSTA_RICA = 506
    CUBA = 53
    DOMINICAN_REPUBLIC = 809
    ECUADOR = 593
    GUATEMALA = 502
    MEXICO = 52
    PERU = 51

    `OTP_AUTH_COUNTRIES_CODES = [57, 54]

## NSN Configuration

AX3 OTP use NSN AWS service for sending messages, please create a group and AIM user with the following policy:

    {
        "Version": "2012-10-17",
        "Statement": [
            {
                "Effect": "Allow",
                "Action": [
                    "sns:Publish",
                    "sns:SetSMSAttributes",
                    "sns:GetSMSAttributes"
                ],
                "Resource": "*"
            }
        ]
    }

Set AIM user credentials to your settings:

    OTP_AUTH_AWS_ACCESS_KEY_ID = ''
    OTP_AUTH_AWS_SECRET_ACCESS_KEY = ''
    OTP_AUTH_AWS_DEFAULT_REGION = 'us-west-2'

## Authentication and Authorization

Authenticated user requires an OTP, this OTP was sent by AWS SNS service, once the code is valid, the system returns a token that must then be used to obtain the phone number which was requested. for this purpose you can use 'get_phone_number':

    hotp = HOTP(session_key=request.session.session_key)
    phone_number = htop.get_phone_number(code='123')