# ⚡ **DarkForge** - بستر قدرتمند توسعه وب  
*ساخته‌شده با Django + Docker + Nginx*

---

## 🚀 **شروع سریع**

### ۱. کلون کردن پروژه
```bash
git clone https://github.com/your-username/DarkForge.git
cd DarkForge
```

### ۲. تنظیم فایل محیطی
```bash
cp .env.example .env
# حالا فایل .env را ویرایش کن و مقادیر دلخواه را وارد کن
```

### ۳. ساخت و اجرای پروژه
```bash
# ساخت و اجرای تمام سرویس‌ها
docker-compose up --build

# یا برای اجرا در پس‌زمینه:
docker-compose up -d
```

### ۴. دسترسی به سرویس‌ها
- 🌐 **وب‌سایت اصلی:** `http://localhost`
- 🔧 **پنل ادمین:** `http://localhost/admin`
- 🗄️ **مدیریت دیتابیس (pgAdmin):** `http://localhost:5050`
- 📊 **API:** `http://localhost/api/`

---

## 🛠️ **تنظیمات اولیه ضروری**

### ایجاد سوپر کاربر
```bash
# پس از اجرای کانتینرها
docker-compose exec web python manage.py createsuperuser
```

### اجرای مایگریشن‌ها
```bash
# اگر مایگریشن اجرا نشده
docker-compose exec web python manage.py migrate
```

---

## 📦 **مدیریت فایل‌های استاتیک - مهم!**

### **مرحله ۱: بررسی تنظیمات Django**
مطمئن شوید `settings.py` این تنظیمات را دارد:
```python
STATIC_URL = '/static/'
STATIC_ROOT = os.path.join(BASE_DIR, 'staticfiles')
```

### **مرحله ۲: جمع‌آوری فایل‌های استاتیک**
```bash
# این دستور تمام فایل‌های CSS، JS و تصاویر را جمع‌آوری می‌کند
docker-compose exec web python manage.py collectstatic --noinput

# اگر خطا داد، ابتدا دسترسی‌ها را تنظیم کنید:
docker-compose exec web chmod -R 755 /app/staticfiles
```

### **مرحله ۳: بررسی صحت اجرا**
1. به `http://localhost/static/admin/css/base.css` بروید
2. اگر فایل CSS نمایش داده شد، استاتیک‌ها درست تنظیم شده‌اند
3. در غیر این صورت، Nginx را ری‌استارت کنید:
```bash
docker-compose restart nginx
```

---

## 🔧 **دستورات کاربردی**

### مدیریت کانتینرها
```bash
# مشاهده لاگ‌های زنده
docker-compose logs -f

# توقف کامل پروژه
docker-compose down

# توقف + حذف volumeها (داده‌های دیتابیس پاک می‌شود!)
docker-compose down -v

# ری‌استارت سرویس خاص
docker-compose restart web
```

### دستورات Django
```bash
# ساخت اپ جدید
docker-compose exec web python manage.py startapp app_name

# ایجاد مایگریشن جدید
docker-compose exec web python manage.py makemigrations

# بررسی سلامت پروژه
docker-compose exec web python manage.py check
```

---

## 🗄️ **اتصال به دیتابیس**

### اطلاعات اتصال پیش‌فرض:
```
Host: db
Port: 5432
Database: postgres
Username: postgres
Password: (مقدار POSTGRES_PASSWORD در .env)
```

### اتصال از طریق pgAdmin:
1. به `http://localhost:5050` بروید
2. وارد شوید با:
   - Email: `admin@darkforge.local`
   - Password: `admin`
3. روی **Add New Server** کلیک کنید
4. در تب **Connection**:
   - Host: `db`
   - Port: `5432`
   - Username/Password: مقادیر `.env`

---

## 🐳 **ساختار Docker پروژه**

```
📦 DarkForge
├── 📂 web/          # سرویس Django + Gunicorn
├── 📂 db/           # PostgreSQL 15
├── 📂 nginx/        # Reverse Proxy + Static Files
└── 📂 pgadmin/      # رابط مدیریت دیتابیس
```

---

## ⚡ **بهینه‌سازی برای Production**

### ۱. تغییر تنظیمات امنیتی
```python
# در settings.py
DEBUG = False
ALLOWED_HOSTS = ['your-domain.com', 'www.your-domain.com']
CSRF_TRUSTED_ORIGINS = ['https://your-domain.com']
```

### ۲. فعال‌سازی HTTPS
فایل `nginx/default.conf` را ویرایش کنید:
```nginx
server {
    listen 80;
    server_name your-domain.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl;
    server_name your-domain.com;
    
    ssl_certificate /path/to/cert.pem;
    ssl_certificate_key /path/to/key.pem;
    
    # بقیه تنظیمات...
}
```

### ۳. تنظیم workerهای بیشتر
```yaml
# در docker-compose.yml سرویس web
command: gunicorn core.wsgi:application --bind 0.0.0.0:8000 --workers 4 --threads 2
```

---

## 🚨 **عیب‌یابی مشکلات رایج**

### مشکل ۱: فایل‌های استاتیک نمایش داده نمی‌شوند
```bash
# راه‌حل:
# ۱. بررسی mount شدن volume
docker-compose exec nginx ls -la /app/static/

# ۲. ری‌لود Nginx
docker-compose exec nginx nginx -s reload

# ۳. جمع‌آوری مجدد استاتیک
docker-compose exec web python manage.py collectstatic --clear
```

### مشکل ۲: خطای اتصال به دیتابیس
```bash
# بررسی وضعیت سرویس db
docker-compose logs db

# ری‌استارت دیتابیس
docker-compose restart db

# انتظار برای آماده‌سازی دیتابیس
docker-compose exec web python manage.py wait_for_db
```

### مشکل ۳: پورت‌ها در حال استفاده
```bash
# پیدا کردن فرآیندهای درگیر
sudo lsof -i :80
sudo lsof -i :5432

# متوقف کردن فرآیندها
sudo kill -9 <PID>
```

---

## 📈 **مانیتورینگ و لاگ**

### مشاهده لاگ‌های هر سرویس
```bash
# Django/Web
docker-compose logs web

# Nginx
docker-compose logs nginx

# Database
docker-compose logs db

# همه لاگ‌ها به صورت زنده
docker-compose logs -f --tail=100
```

### بررسی سلامت سرویس‌ها
```bash
# وضعیت کانتینرها
docker-compose ps

# مصرف منابع
docker stats

# فضای دیسک
docker system df
```

---

## 🎯 **نکات طلایی برای توسعه**

### ۱. نصب پکیج جدید
```bash
# به requirements.txt اضافه کنید، سپس:
docker-compose down
docker-compose up --build
```

### ۲. ریست دیتابیس (تست)
```bash
# حذف داده‌ها و شروع مجدد
docker-compose down -v
docker-compose up -d
docker-compose exec web python manage.py migrate
docker-compose exec web python manage.py createsuperuser
```

### ۳. توسعه Frontend
```bash
# فایل‌های استاتیک جدید را اضافه کنید
# سپس حتما collectstatic را اجرا کنید
docker-compose exec web python manage.py collectstatic
```

---

## 🤝 **مشارکت در توسعه**

1. Fork کنید
2. Branch بسازید: `git checkout -b feature/amazing-feature`
3. Commit کنید: `git commit -m 'Add amazing feature'`
4. Push کنید: `git push origin feature/amazing-feature`
5. Pull Request باز کنید

---

## 📄 **لایسنس**

این پروژه تحت لایسنس MIT منتشر شده است.

---

## ❤️ **تشکر ویژه**

از شما که DarkForge را انتخاب کردید!  
اگر سوالی داشتید یا با مشکلی مواجه شدید،  
Issue جدید باز کنید یا به من ایمیل بزنید.

---

**ساخته شده با ❤️ توسط جامعه توسعه‌دهندگان**  
*هر ستاره‌ای که می‌زنید، انگیزه‌ای برای ادامه می‌شود ⭐*

---

> 💡 **نکته آخر:** همیشه قبل از push کردن کد، تست‌ها را اجرا کنید و مطمئن شوید همه سرویس‌ها سالم هستند!

```bash
# تست نهایی
docker-compose down
docker-compose up -d
# صبر کنید تا همه سرویس‌ها بالا بیایند
docker-compose ps
# همه چیز باید Healthy یا Up باشد
```

**حالا آماده‌اید! 🚀**  
پروژه شما روی `http://localhost` در حال اجراست.
