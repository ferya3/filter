# filter

اسکریپت‌های مدیریت تانل: کاربرها به سرور ایران وصل می‌شوند و HAProxy ترافیک را به سرورهای خارج
(Xray با VLESS + Reality) می‌فرستد.

| پوشه | چه کاری می‌کند | کجا اجرا می‌شود |
|---|---|---|
| [`panel/`](panel/) | پنل وب: کاربرها، سقف حجم، تاریخ انقضا، لینک و QR، محدودیت دسترسی هر کاربر | خارج ۱ |
| [`peer/`](peer/) | آماده کردن یک سرور خارج تازه با همان کلیدها | سرور خارج جدید، از طریق خارج ۱ |

هیچ رمز یا کلیدی در این ریپو نیست؛ کلیدها فقط روی خود سرورها ساخته و نگه داشته می‌شوند
(`/root/tunnel.env` روی خارج ۱).

## نصب یا به‌روزرسانی پنل (روی خارج ۱)

```bash
curl -fsSL https://raw.githubusercontent.com/ferya3/filter/main/panel/install.sh -o /root/tunpanel-install.sh
bash /root/tunpanel-install.sh
```

بار اول آدرس و رمز پنل را چاپ می‌کند. اجرای دوباره فقط کد را به‌روز می‌کند؛ رمز، کاربرها و مصرف دست نمی‌خورند.
جزئیات بیشتر: [`panel/README.md`](panel/README.md).

## اضافه کردن یک سرور خارج

روی **خارج ۱** (IP سرور جدید را در خط اول بگذارید):

```bash
T=x.x.x.x
ssh-copy-id root@$T
scp /root/tunnel.env root@$T:/root/tunnel.env
curl -fsSL https://raw.githubusercontent.com/ferya3/filter/main/peer/setup.sh -o /root/peer-setup.sh
ssh root@$T 'bash -s' < /root/peer-setup.sh
grep -q "^$T " /etc/tun-peers || echo "$T 22" >> /etc/tun-peers
```

پنل ظرف یک دقیقه کانفیگ کامل (همه‌ی کاربرها) را روی سرور جدید کپی می‌کند.

بعد روی **سرور ایران**:

```bash
tun-addserver f3 x.x.x.x          # همیشه فعال
tun-addserver f3 x.x.x.x backup   # فقط وقتی بقیه قطع باشند
```
