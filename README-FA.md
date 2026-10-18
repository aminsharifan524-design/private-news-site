# راه‌انداز اولیه سایت خبری خصوصی

این یک نسخه آغازین از سایت خبری با Next.js و Supabase است:
- ورود با نام کاربری و رمز عبور؛ کاربران به ایمیل واقعی نیاز ندارند.
- حساب کاربران فقط از پنل مدیر ساخته می‌شود.
- خبرها بعد از ورود نمایش داده می‌شوند.
- پنل مدیر برای انتشار خبر و ساخت حساب دارد.

## مهم
این نسخه هنوز روی اینترنت منتشر نشده و قبل از استفاده واقعی باید مراحل تنظیم امنیت انجام شود. کلید `service_role` را هرگز در کد مرورگر، پیام یا مخزن عمومی قرار ندهید.

## متغیرهای Vercel
در Project Settings → Environment Variables این موارد را اضافه کنید:
- `NEXT_PUBLIC_SUPABASE_URL`
- `NEXT_PUBLIC_SUPABASE_ANON_KEY`
- `SUPABASE_SERVICE_ROLE_KEY` (محرمانه و فقط سمت سرور)
- `BOOTSTRAP_SECRET` (یک عبارت تصادفی طولانی و محرمانه)

## ساخت مدیر اولیه
پس از Deploy، با یک ابزار ارسال درخواست HTTP، درخواست POST به `/api/bootstrap` بفرستید و JSON زیر را با مقادیر خودتان جایگزین کنید:
`{"secret":"BOOTSTRAP_SECRET","username":"admin","password":"یک-رمز-قوی-حداقل-۱۰-نویسه"}`

پس از موفقیت، متغیر `BOOTSTRAP_SECRET` را فوراً از Vercel حذف کنید و دوباره Deploy بگیرید. این مسیر راه‌اندازی نباید برای همیشه فعال بماند.

## سیاست‌های دیتابیس
در Supabase → SQL Editor اجرا کنید. نام جدول باید دقیقاً `"News"` باشد:

```sql
alter table public."News" enable row level security;

create policy "Signed-in users read published news"
on public."News" for select to authenticated
using (published = true);

create policy "Admins insert news"
on public."News" for insert to authenticated
with check ((auth.jwt() -> 'app_metadata' ->> 'role') = 'admin');

create policy "Admins update news"
on public."News" for update to authenticated
using ((auth.jwt() -> 'app_metadata' ->> 'role') = 'admin')
with check ((auth.jwt() -> 'app_metadata' ->> 'role') = 'admin');

create policy "Admins delete news"
on public."News" for delete to authenticated
using ((auth.jwt() -> 'app_metadata' ->> 'role') = 'admin');
```

## ورود بدون ایمیل واقعی
Supabase همچنان از سازوکار Email/Password استفاده می‌کند، اما برنامه نام کاربری را به یک نشانی داخلی در دامنه `.invalid` تبدیل می‌کند. کاربران ایمیل واقعی وارد نمی‌کنند.
