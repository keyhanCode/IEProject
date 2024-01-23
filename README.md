# **موضوع پروژه**

پیاده سازی وبلاگ (قابلیت عضویت و اشتراک‌گذاری)

برنامه‌نویس : سجادزارع‌پور

# **شرح برنامه** **:**

یک وب سایت پایه (بر پایه معماری MVC (با احراز هویت کاربر با استفاده از Google OAuth2. که اجزای زیر تشکیل شده است:

1. احراز هویت کاربر: برنامه از Google OAuth2برای احراز هویت کاربران استفاده می کند. هنگامی که کاربر روی دکمه "ورود با گوگل" کلیک می کند، برنامه کاربر را به صفحه مجوز گوگل هدایت می کند. سپس گوگل کاربر را با یک کد مجوز به برنامه باز می گرداند. برنامه کد مجوز را با یک توکن دسترسی مبادله می کند و از توکن دسترسی برای بازیابی اطلاعات نمایه کاربر استفاده می کند. اگر آدرس ایمیل کاربر تأیید شده باشد، یک رکورد کاربر جدید در پایگاه داده برنامه ایجاد می شود و کاربر وارد می شود.

**ساختار کلی برنامه و روش های پیاده سازی**  **:**

**ماژول های برنامه**  **:**

    # Python standard libraries
    
    import json
    
    # Third party libraries
    
    from flask import Flask, redirect, request, url\_for,render\_template
    
    from flask\_login import (
    
        LoginManager,
    
        current\_user,
    
        login\_required,
    
        login\_user,
    
        logout\_user
    
    )
    
    from oauthlib.oauth2 importWebApplicationClient
    
    import requests
    
    import urllib.parse
    
    # Internal imports
    
    from config import\*
    
    from database import db
    
    from models import\*

## **ماژول** **json**

توابعی را برای کار با داده های JSON فراهم می کند، که یک فرمت رایج برای ذخیره و تبادل داده های ساختاریافته است. این به برنامه اجازه می دهد تا داده های فرمت JSON را مانند پروفایل های کاربر یا موارد محتوا دستکاری و پردازش کند.

## **کتابخانه** **flask**

پایه و اساس ایجاد برنامه های وب باPython است. این توابعی را برای مسیریابی درخواست ها، رندر قالب ها، مدیریت وضعیت برنامه و اتصال به پایگاه داده ها فراهم می کند.

## **افزونه login flask**

Flaskرا با Flask-Login، یک چارچوب برای مدیریت احراز هویت کاربر و جلسات ادغام می کند. این مدیریت ورود کاربر، خروج و جلسات را در برنامه ساده می کند.

## **کلاس** **WebApplicationClient**

از کتابخانه oauthlib برای تعامل با ارائه دهندگانOAuth2 مانندGoogle استفاده می شود. این دریافت توکن های مجوز برای احراز هویت کاربر و دسترسی به منابع محافظت شده را تسهیل می کند.

## **کتابخانه** **requests**

درخواست هایHTTP را بهAPI های خارجی یا خدمات ارسال می کند. این ارسال و دریافت داده از سرورهای راه دور را ساده می کند که برای تعامل با ارائه دهندگانOAuth2 یا دسترسی به محتوا از منابع خارجی ضروری است.

## **ماژول** **urllib.parse**

توابعی را برای تجزیه و کدگذاریURL فراهم می کند که برای مدیریتURL ها و نقاط پایانیAPI ضروری است. این به برنامه اجازه می دهد تاURL ها را به درستی برای احراز هویت و بازیابی داده هاconstruct وparse کند.

## **ماژول** **models**

مدل های داده ای را برای ذخیره اطلاعات کاربر و داده های محتوا تعریف می کند.

# بخش Google Configuration

    # Google Configuration
    
    GOOGLE\_DISCOVERY\_URL= (
    
        "https://accounts.google.com/.well-known/openid-configuration"
    
    )

کد GOOGLE\_DISCOVERY\_URL یک مقدار ثابت را تعریف می‌کند که آدرس URL یک فایل JSON را مشخص می‌کند که اطلاعات مربوط به پیکربندی OpenID Connect را برای Google ارائه می‌دهد. این اطلاعات برای برنامه‌هایی که از OpenID Connect برای احراز هویت کاربران با استفاده از حساب‌های Google استفاده می‌کنند، ضروری است.

**کد** **Flask app setup:**

    # Flask app setup
    
    app = Flask(\_\_name\_\_)
    
    app.config['SECRET\_KEY'] =SECRET\_KEY
    
    app.config['SQLALCHEMY\_DATABASE\_URI'] =SQLALCHEMY\_DATABASE\_URI

این کد یک برنامه سادهFlask را راه اندازی می کند.

خط 1: کلاسFlask را وارد می کنیم.

خط 2: یک نمونه از کلاسFlask ایجاد می کنیم. اولین آرگومان نام ماژول یا بسته برنامه است. name میانبر مناسبی برای این کار است و در اکثر موارد مناسب است. این آرگومان اجباری است چون باید فلاسک بداند که کجا به دنبال منابعی مانند قالب ها و فایل های استاتیک باشد.

خط 3: مقدارSECRET\_KEY را به تنظیمات برنامه اضافه می کنیم. این کلید برای ایجاد رمزهای هش برای احراز هویت استفاده می شود.

خط 4: مقدارSQLALCHEMY\_DATABASE\_URI را به تنظیمات برنامه اضافه می کنیم. اینURI نشان دهنده محل پایگاه دادهSQLAlchemy است.

# **تنظیم مدیریت نشست کاربر**** :**

    # User session management setup
    
    # https://flask-login.readthedocs.io/en/latest
    
    login\_manager = LoginManager()
    
    login\_manager.init\_app(app)

## **توضیحات**** :**

کد بالا برای تنظیم مدیریت نشست کاربر در فلاسک با استفاده از پکیجFlask-Login استفاده می‌شود.

در خط اول کد، یک شیء از کلاسLoginManager ایجاد می‌شود. این شیء مسئول مدیریت نشست کاربران است.

در خط دوم، شیءLoginManager به برنامه فلاسک تزریق می‌شود. این کار باعث می‌شود که پکیجFlask-Login بتواند دسترسی به برنامه فلاسک را داشته باشد.

پس از اجرای این کد، پکیجFlask-Login آماده است تا برای مدیریت نشست کاربران استفاده شود

## **مدیریت ورود کاربر**** :**

    @login\_manager.unauthorized\_handler
    
    defunauthorized():
    
        return"You must be logged in to access this content.", 403
    
    # OAuth2 client setup
    
    client = WebApplicationClient(GOOGLE\_CLIENT\_ID)
    
    # Flask-Login helper to retrieve a user from our db
    
    @login\_manager.user\_loader
    
    defload\_user(_user\_id_):
    
        return User.get(_user\_id_)

این کد برای پیکربندی مدیریت ورود کاربر در یک برنامهFlask با استفاده ازOAuth2 وFlask-Login نوشته شده است.

## **تابع** **unauthorized()**

پیام خطای"You must be logged in to access this content." را با کد وضعیتHTTP 403 دسترسی ممنوع برمی‌گرداند. این تابع در صورت عدم ورود کاربر به سیستم فراخوانی می‌شود.

## **تابع** **load\_user()**

یک کاربر را از پایگاه داده بازیابی می‌کند. این تابع بهFlask-Login کمک می‌کند تا کاربر را در هنگام ورود به سیستم شناسایی کند.

مسیر های تعریف شده در برنامه routes :

    @app.route("/")
    
    defindex():
    
        if current\_user.is\_authenticated :
    
            contents = json.load(open("contents.json", "r", _encoding_="utf-8"))
    
        else :
    
            contents = {}
    
        return render\_template('index.html' , _user_= current\_user ,_contents_= contents)

## **تابع** **index()**

یک تابع مسیر(route function) درFlask است. این تابع زمانی فراخوانی می‌شود که کاربر به آدرس / درخواست دهد.

## **بررسی کد**** :**

در ابتدا، تابع بررسی می‌کند که آیا کاربر وارد سیستم شده است یا خیر. اگر کاربر وارد سیستم شده باشد، تابع محتوای فایل contents.json را بارگذاری می‌کند. در غیر این صورت، تابع یک دیکشنری خالی را به عنوان محتوا بازمی‌گرداند.

در نهایت، تابع محتویات را به همراه اطلاعات کاربر به قالب index.html ارسال می‌کند.

    @app.route("/profile")
    
    deflogin():
    
        return render\_template('profile.html' , _user_= current\_user)

این کد Flask یک صفحه پروفایل کاربر را نمایش می دهد. این صفحه شامل اطلاعات کاربر مانند نام، نام خانوادگی، آدرس ایمیل و غیره است.

    @app.route("/about")
    
    defabout():
    
        return render\_template('about.html' , _user_= current\_user)

این کد یک مسیر(route) برایURL /about در برنامهFlask تعریف می‌کند. این مسیر به تابع about() ارجاع می‌دهد.

## **تابع** **about()**

یک قالب(template) HTML با نام about.html را رندر می‌کند. همچنین، مقدار متغیر user را به قالب ارسال می‌کند.

@app.route("/contents/\<url\>")

defcontents(_url_):

    contents = json.load(open("contents.json", "r", _encoding_="utf-8"))

    request\_url = urllib.parse.quote(request.url)

    for content in contents:

        if content["title"] ==_url_:

            return render\_template("contents.html", _user_=current\_user, _contents_=content, _request\_url_=request\_url)

    return render\_template("404.html", _user_=current\_user)

این تابع با دریافت یکURL به عنوان ورودی، محتوای مربوط به آنURL را از فایل"contents.json" بارگیری می‌کند. سپس، URL درخواستی را با استفاده از تابعurllib.parse.quote() کدگذاری می‌کند. در نهایت، برای هر مورد در لیستcontents، بررسی می‌کند که آیا عنوان آن باURL ورودی مطابقت دارد. اگرURL مورد نظر یافت شود، محتوای مربوطه را به همراهURL درخواستی در قالبHTML بازنویسی می‌کند و آن را به کاربر ارائه می‌دهد. در غیر این صورت، صفحه 404 HTML را بازنویسی می‌کند و آن را به کاربر ارائه می‌دهد.

@app.route("/google-login")

defgoogle\_login():

    # Find out what URL to hit for Google login

    google\_provider\_cfg = get\_google\_provider\_cfg()

    authorization\_endpoint = google\_provider\_cfg["authorization\_endpoint"]

    # Use library to construct the request for login and provide

    # scopes that let you retrieve user's profile from Google

    request\_uri = client.prepare\_request\_uri(

        authorization\_endpoint,

        _redirect\_uri_=request.base\_url +"/callback",

        _scope_=["openid", "email", "profile"],

    )

    return redirect(request\_uri)

کد فوق یک تابعFlask است که برای شروع فرآیند ورود به سیستمGoogle طراحی شده است. این تابع ابتدا آدرسURL را برای ورود به سیستمGoogle پیدا می کند و سپس از کتابخانهOAuth2 Client برای ساخت درخواست ورود به سیستم استفاده می کند. درخواست شامل حوزه هایی است که به شما امکان می دهد نمایه کاربر را ازGoogle بازیابی کنید. در نهایت، تابع کاربر را بهURL درخواست هدایت می کند.

    @app.route("/google-login/callback")
    
    defcallback():
    
        # Get authorization code Google sent back to you
    
        code = request.args.get("code")
    
        # Find out what URL to hit to get tokens that allow you to ask for
    
        # things on behalf of a user
    
        google\_provider\_cfg = get\_google\_provider\_cfg()
    
        token\_endpoint = google\_provider\_cfg["token\_endpoint"]
    
        # Prepare and send request to get tokens! Yay tokens!
    
        token\_url, headers, body = client.prepare\_token\_request(
    
            token\_endpoint,
    
            _authorization\_response_=request.url,
    
            _redirect\_url_=request.base\_url,
    
            _code_=code,
    
        )
    
        token\_response = requests.post(
    
            token\_url,
    
            _headers_=headers,
    
            _data_=body,
    
            _auth_=(GOOGLE\_CLIENT\_ID, GOOGLE\_CLIENT\_SECRET),
    
        )
    
        # Parse the tokens!
    
        client.parse\_request\_body\_response(json.dumps(token\_response.json()))
    
        # Now that we have tokens (yay) let's find and hit URL
    
        # from Google that gives you user's profile information,
    
        # including their Google Profile Image and Email
    
        userinfo\_endpoint = google\_provider\_cfg["userinfo\_endpoint"]
    
        uri, headers, body = client.add\_token(userinfo\_endpoint)
    
        userinfo\_response = requests.get(uri, _headers_=headers, _data_=body)
    
        # We want to make sure their email is verified.
    
        # The user authenticated with Google, authorized our
    
        # app, and now we've verified their email through Google!
    
        if userinfo\_response.json().get("email\_verified"):
    
            unique\_id = userinfo\_response.json()["sub"]
    
            users\_email = userinfo\_response.json()["email"]
    
            picture = userinfo\_response.json()["picture"]
    
            users\_name = userinfo\_response.json()["given\_name"]
    
        else:
    
            return"User email not available or not verified by Google.", 400
    
        # Create a user in our db with the information provided
    
        # by Google
    
        user = User.get(unique\_id)
    
        ifnot user:
    
            user = User(_id_=unique\_id, _name_=users\_name, _email_=users\_email, _profile\_pic_=picture)
    
            db.session.add(user)
    
        db.session.commit()
    
        # Begin user session by logging the user in
    
        login\_user(user)
    
        # Send user back to homepage
    
        return redirect('/profile')

این کد یک مسیر برایCallback گوگل در برنامهFlask ایجاد می کند. این مسیر پس از موفقیت آمیز بودن احراز هویت کاربر در گوگل فراخوانی می شود. این کد مراحل زیر را دنبال می کند:

1. دریافت کد تأیید از گوگل: کد تأیید را از پارامتر code درURL درخواست دریافت می کند.
2. دریافتURL endpoint برای دریافت توکن ها: URL endpoint را برای دریافت توکن هایی که به شما امکان می دهد به نام کاربر درخواست دهید پیدا می کند.
3. درخواست توکن ها از گوگل: درخواستی را برای دریافت توکن ها با استفاده از کد تأیید و اطلاعاتGoogle Client ID وGoogle Client Secret ارسال می کند.
4. تجزیه توکن ها: توکن های دریافتی را تجزیه می کند و اطلاعات مربوط به کاربر مانندID کاربر، نام کاربری و تصویر را استخراج می کند.
5. ایجاد یا یافتن کاربر در پایگاه داده: بررسی می کند که آیا کاربر در پایگاه داده موجود است یا خیر. اگر کاربر وجود ندارد، یک کاربر جدید با اطلاعات دریافتی از گوگل ایجاد می شود.
6. شروع جلسه کاربر: کاربر را با استفاده از اطلاعات کاربر ایجاد شده یا موجود در پایگاه داده وارد سیستم می کند.
7. هدایت کاربر به صفحه پروفایل: کاربر را به صفحه پروفایل هدایت می کند.

    @app.route("/logout")
    
    @login\_required
    
    deflogout():
    
        logout\_user()
    
        return redirect(url\_for("index"))
    
    defget\_google\_provider\_cfg():
    
        return requests.get(GOOGLE\_DISCOVERY\_URL).json()
    
    logout()

این تابع با استفاده از تابع logout\_user() موجود در کتابخانهFlask، کاربر را از سیستم خارج می‌کند. سپس کاربر را به صفحه اصلی (صفحه index) هدایت می‌کند.

## **تابع** **get\_google\_provider\_cfg()**

این تابع با استفاده از کتابخانه requests، اطلاعات مربوط به ارائه‌دهنده گوگل را ازURL مشخص‌شده(GOOGLE\_DISCOVERY\_URL) دریافت می‌کند. سپس این اطلاعات را به صورت یک شیءJSON برمی‌گرداند.

## **تابع** **logout()**

تابع logout() با استفاده ازdecorator @login\_required اطمینان حاصل می‌کند که فقط کاربرانی که وارد سیستم شده‌اند می‌توانند از آن استفاده کنند.

در داخل تابع، ابتدا تابع logout\_user() را فراخوانی می‌کنیم تا کاربر را از سیستم خارج کنیم. سپس تابع redirect() را فراخوانی می‌کنیم تا کاربر را به صفحه اصلی هدایت کنیم.

## **تابع** **get\_google\_provider\_cfg()**

تابع get\_google\_provider\_cfg() با استفاده از متد get() کتابخانه requests، اطلاعات مربوط به ارائه‌دهنده گوگل را ازURL مشخص‌شده(GOOGLE\_DISCOVERY\_URL) دریافت می‌کند.

سپس این اطلاعات را به صورت یک شیءJSON برمی‌گرداند.

## **توضیح**** URL GOOGLE\_DISCOVERY\_URL**

اینURL یکURL عمومی است که اطلاعات مربوط به ارائه‌دهنده گوگل را در اختیار برنامه‌نویسان قرار می‌دهد. این اطلاعات شامل اطلاعاتی مانند آدرسURL صفحه ورود، آدرسURL صفحه تأیید، و آدرسURL صفحه خطا است.
