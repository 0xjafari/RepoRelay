# 📡 RepoRelay — Fetch Any URL via GitHub Actions + Pages / دریافت هر آدرس با اکشن‌ها و پیجز گیت‌هاب

[![GitHub Actions](https://img.shields.io/badge/GitHub%20Actions-Workflow-blue.svg)](https://github.com/features/actions)
[![GitHub Pages](https://img.shields.io/badge/GitHub%20Pages-Live-brightgreen.svg)](https://pages.github.com/)
[![License: GPL v3](https://img.shields.io/badge/License-GPLv3-blue.svg)](https://www.gnu.org/licenses/gpl-3.0)

> **English** – Use GitHub’s infrastructure to fetch any public URL and instantly host the result on GitHub Pages. No local server, no installation.  
> **فارسی** – با استفاده از زیرساخت گیت‌هاب، هر آدرسی را دریافت کرده و نتیجه را روی GitHub Pages میزبانی کنید. بدون نیاز به نصب یا سرور محلی.

- 🚀 **Fully automated** – One click to fetch & publish.
- 🌍 **Live webpage** – Served instantly via GitHub Pages.
- 🧩 **Simple & minimal** – Only a workflow file + Pages activation.

---

## 📖 Table of Contents / فهرست مطالب

1. [How It Works](#-1-how-it-works--روش-کار)
2. [Architecture Overview](#-2-architecture-overview--معماری-کلی)
3. [Repository Structure](#-3-repository-structure--ساختار-مخزن)
4. [One‑Time Setup](#-4-one-time-setup--تنظیمات-اولیه)
5. [Daily Usage](#-5-daily-usage--استفاده-روزانه)
6. [Limitations & Known Issues](#-6-limitations--known-issues--محدودیت‌ها-و-مشکلات-شناخته‌شده)
7. [Troubleshooting](#-7-troubleshooting--عیب‌یابی)
8. [Security Recommendations](#-8-security-recommendations--توصیه‌های-امنیتی)
9. [Intended Use](#-9-intended-use--کاربرد-مجاز)
10. [Disclaimer](#-10-disclaimer--سلب-مسئولیت)
11. [License (GNU GPLv3)](#-11-license-gnu-gplv3--مجوز-gnu-gplv3)

---

# ⚙️ 1. How It Works / روش کار

**English**  

RepoRelay uses **GitHub Actions** (runners on GitHub’s infrastructure) to fetch any HTTP/HTTPS URL you provide. The fetched HTML is saved to `fetched_content.html` in your repository and then automatically published via **GitHub Pages**.  

You never run any code locally – everything happens inside GitHub.

**فارسی**  

خب RepoRelay از **GitHub Actions** (سرورهای اجراکننده در زیرساخت گیت‌هاب) استفاده می‌کند تا هر آدرسی را که وارد کنید دریافت کند. HTML گرفته شده در فایل `fetched_content.html` در مخزن شما ذخیره و سپس توسط **GitHub Pages** منتشر می‌شود.

هیچ کدی روی سیستم شما اجرا نمی‌شود – همه چیز درون گیت‌هاب انجام می‌گردد.

---

# 🧠 2. Architecture Overview / معماری کلی

**English**  

```
You → github.com → GitHub Runner (cloud)
                         │
                         ├── curl $URL
                         │
                         ▼
                   fetched_content.html
                         │
                         ├── commit + push
                         │
                         ▼
                   GitHub Pages (live)
                         │
                         ▼
You ← https://username.github.io/repo/fetched_content.html
```

| Component | Role | Technology |
|-----------|------|-------------|
| **GitHub Actions** | Executes `curl` on a cloud runner | YAML workflow, `ubuntu-latest` |
| **GitHub Pages** | Serves the fetched HTML as a website | Static site hosting |
| **Your browser** | Views the live page via `github.io` | Any modern browser |

**فارسی**  

```
شما → github.com → اجراکننده گیت‌هاب (ابر)
                         │
                         ├── curl $URL
                         │
                         ▼
                   fetched_content.html
                         │
                         ├── commit + push
                         │
                         ▼
                   GitHub Pages (زنده)
                         │
                         ▼
شما ← https://username.github.io/repo/fetched_content.html
```

| جزء | نقش | فناوری |
|-----|------|--------|
| **GitHub Actions** | اجرای `curl` روی یک اجراکننده ابری | YAML workflow، `ubuntu-latest` |
| **GitHub Pages** | ارائه HTML گرفته شده به عنوان یک وب‌سایت | میزبانی سایت ایستا |
| **مرورگر شما** | مشاهده صفحه زنده از طریق `github.io` | هر مرورگر مدرن |

---

# 📂 3. Repository Structure / ساختار مخزن

```
RepoRelay/
├── .github/
│   └── workflows/
│       └── fetch.yml          # GitHub Action workflow
├── fetched_content.html       # Auto‑generated (starts empty)
├── README.md                  # This file
└── .gitignore                 # Optional
```

Only two files are required: the workflow and this README. Everything else is created automatically.

---

# 🛠️ 4. One‑Time Setup / تنظیمات اولیه

**English**  

### Prerequisites
- A GitHub account.

### Step‑by‑Step

1. **Create a new public repository** on GitHub (e.g., `RepoRelay`).  
   *Do NOT add a README or .gitignore – we will add them manually.*

2. **Create the workflow file**  
   - In your repo, click **Add file** → **Create new file**.
   - Path: `.github/workflows/fetch.yml`
   - Paste the content below.

3. **Enable GitHub Pages**  
   - Go to **Settings** → **Pages**.
   - Under **Branch**, select `main` and `/ (root)`.
   - Click **Save**. Wait a minute for the green message: *"Your site is published at https://...".*

4. **Create this README** (optional but recommended)  
   - Add a file `README.md` with the content you are reading now.

#### `fetch.yml` (copy exactly)

```yaml
name: Fetch URL

on:
  workflow_dispatch:
    inputs:
      url:
        description: 'URL (e.g., https://example.com)'
        required: true
        default: 'https://example.com'

env:
  FORCE_JAVASCRIPT_ACTIONS_TO_NODE24: true

jobs:
  fetch:
    runs-on: ubuntu-latest
    permissions:
      contents: write
    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Fetch content from URL (overwrite file)
        run: |
          curl -s -L "${{ github.event.inputs.url }}" > fetched_content.html
          echo "" >> fetched_content.html
          echo "<!-- Fetched at $(date) from ${{ github.event.inputs.url }} -->" >> fetched_content.html

      - name: Commit and push changes (safe rebase)
        run: |
          git config user.name "github-actions"
          git config user.email "actions@github.com"
          git add fetched_content.html
          if git diff --cached --quiet; then
            echo "No changes detected. Nothing to commit."
            exit 0
          fi
          git commit -m "Update fetched content from ${{ github.event.inputs.url }}"
          git pull --rebase origin main
          git push origin main
```

**فارسی**  

### پیش‌نیازها
- یک حساب گیت‌هاب.

### گام به گام

۱. **یک مخزن عمومی جدید** در گیت‌هاب بسازید (مثلاً `RepoRelay`).  
   *فایل README یا .gitignore اضافه نکنید – بعداً دستی اضافه می‌کنیم.*

۲. **فایل workflow را ایجاد کنید**  
   - در مخزن خود، روی **Add file** → **Create new file** کلیک کنید.
   - مسیر: `.github/workflows/fetch.yml`
   - محتوای زیر را کپی کنید.

۳. **GitHub Pages را فعال کنید**  
   - به **Settings** → **Pages** بروید.
   - در قسمت **Branch**، `main` و `/ (root)` را انتخاب کنید.
   - روی **Save** کلیک کنید. یک دقیقه صبر کنید تا پیغام سبز *"Your site is published..."* ظاهر شود.

۴. **این README را ایجاد کنید** (اختیاری اما توصیه می‌شود)  
   - فایل `README.md` را با محتوایی که هم‌اکنون می‌خوانید اضافه کنید.

#### محتوای `fetch.yml` (دقیقاً کپی کنید)

```yaml
name: Fetch URL

on:
  workflow_dispatch:
    inputs:
      url:
        description: 'آدرس (مثلاً https://example.com)'
        required: true
        default: 'https://example.com'

env:
  FORCE_JAVASCRIPT_ACTIONS_TO_NODE24: true

jobs:
  fetch:
    runs-on: ubuntu-latest
    permissions:
      contents: write
    steps:
      - name: دریافت مخزن
        uses: actions/checkout@v4

      - name: دریافت محتوا از آدرس (رونویسی فایل)
        run: |
          curl -s -L "${{ github.event.inputs.url }}" > fetched_content.html
          echo "" >> fetched_content.html
          echo "<!-- دریافت شده در $(date) از ${{ github.event.inputs.url }} -->" >> fetched_content.html

      - name: commit و push تغییرات (rebase امن)
        run: |
          git config user.name "github-actions"
          git config user.email "actions@github.com"
          git add fetched_content.html
          if git diff --cached --quiet; then
            echo "تغییری وجود ندارد. اقدامی نیاز نیست."
            exit 0
          fi
          git commit -m "به‌روزرسانی محتوا از ${{ github.event.inputs.url }}"
          git pull --rebase origin main
          git push origin main
```

---

# 🚀 5. Daily Usage / استفاده روزانه

**English**  

Every time you want to fetch a URL:

1. Go to your repository on GitHub.
2. Click the **Actions** tab.
3. On the left sidebar, click **Fetch URL**.
4. Click **Run workflow** → a small form appears.
5. Enter the **full URL** (including `https://`).
6. Click **Run workflow**.
7. Wait about 30 seconds until the yellow dot turns green.
8. Open your GitHub Pages link:  
   `https://YOUR_USERNAME.github.io/REPO_NAME/fetched_content.html`  
   (Replace `YOUR_USERNAME` and `REPO_NAME` with your actual names.)

The page will now show the fetched content. Press **Ctrl+F5** to hard‑refresh if you see an old version.

**فارسی**  

هر بار که می‌خواهید یک آدرس را دریافت کنید:

۱. به مخزن خود در گیت‌هاب بروید.  
۲. روی برگه **Actions** کلیک کنید.  
۳. در نوار کناری سمت چپ، روی **Fetch URL** کلیک کنید.  
۴. روی دکمه **Run workflow** کلیک کنید → یک فرم کوچک باز می‌شود.  
۵. **آدرس کامل** (شامل `https://`) را وارد کنید.  
۶. روی **Run workflow** کلیک کنید.  
۷. حدود ۳۰ ثانیه صبر کنید تا دایره زرد رنگ سبز شود.  
۸. لینک GitHub Pages خود را باز کنید:  
   `https://USERNAME.github.io/REPONAME/fetched_content.html`  
   (به جای USERNAME و REPONAME نام واقعی خود را بنویسید.)

صفحه اکنون محتوای گرفته شده را نشان می‌دهد. اگر نسخه قدیمی می‌بینید با **Ctrl+F5** صفحه را به‌طور کامل تازه کنید.

---

# ⚠️ 6. Limitations & Known Issues / محدودیت‌ها و مشکلات شناخته‌شده

| Limitation (English) / محدودیت (فارسی) | Explanation / توضیح |
|----------------------------------------|---------------------|
| **Only simple GET requests** | Cannot submit forms, log in, or handle JavaScript‑rendered content. |
| **Relative URLs break styling** | CSS, images, and scripts using relative paths won’t load. The page may look ugly, but text and links remain readable. |
| **Delay** | Fetch + Pages rebuild takes 1‑2 minutes. |
| **Monthly action minutes** | Free tier gives 2000 minutes. Each fetch uses ~1 minute. |
| **No interactive browsing** | Each URL must be entered manually. Links inside the fetched page will point to the original site (which may or may not be accessible). |

---

# 🔧 7. Troubleshooting / عیب‌یابی

| Error (English) / خطا (فارسی) | Likely Cause / علت محتمل | Solution / راه‌حل |
|------------------------------|--------------------------|--------------------|
| `403 Permission denied` | Workflow lacks write access | **Settings → Actions → Workflow permissions → Read and write permissions** |
| `cannot pull with rebase: You have unstaged changes` | Old workflow version | The YAML above already fixes this. Update your `fetch.yml`. |
| GitHub Pages shows old content | Browser cache or Pages rebuild delay | Hard refresh (Ctrl+F5) or wait 2 minutes and retry. |
| `curl: (28) Connection timed out` | Target website is slow or blocking GitHub runners | No fix – some sites block `curl` from cloud IPs. Try a different URL. |
| Empty `fetched_content.html` | URL missing `https://` or site requires JavaScript | Add `https://`. For JS‑heavy sites, this method will not work. |

---

# 🧾 8. Security Recommendations / توصیه‌های امنیتی

**English**  
- **Use a dedicated GitHub account** for sensitive fetching if needed.
- **Do not fetch illegal content** – see section 9 for intended use.
- **Regularly delete old versions** of `fetched_content.html` if you no longer need them (they stay in Git history unless you force‑push).

**فارسی**  
- **از یک حساب گیت‌هاب جداگانه** برای دریافت‌های حساس استفاده کنید.
- **محتوای غیرقانونی دریافت نکنید** – بخش ۹ را ببینید.
- **نسخه‌های قدیمی** `fetched_content.html` را حذف کنید اگر دیگر نیاز ندارید (آنها در تاریخچه گیت باقی می‌مانند مگر اینکه force-push کنید).

---

# 🎯 9. Intended Use / کاربرد مجاز

**English**  

RepoRelay is designed for legitimate purposes such as:

- **Archiving web pages** – saving a static snapshot of any public URL.
- **Debugging and testing** – checking how a remote page behaves when fetched from a cloud IP.
- **Educational demonstrations** – showing how GitHub Actions and Pages can be combined.
- **Personal automation** – fetching content for further processing (e.g., with other GitHub Actions).

You are responsible for complying with all applicable laws and the terms of service of any website you fetch.

**فارسی**  

RepoRelay برای کاربردهای قانونی مانند زیر طراحی شده است:

- **بایگانی صفحات وب** – ذخیره یک نسخه ایستا از هر آدرس عمومی.
- **اشکال‌زدایی و تست** – بررسی رفتار یک صفحه راه‌دور هنگام دریافت از یک IP ابری.
- **نمایش‌های آموزشی** – نشان دادن ترکیب قابلیت‌های GitHub Actions و Pages.
- **اتوماسیون شخصی** – دریافت محتوا برای پردازش بیشتر (مثلاً با سایر GitHub Actions).

شما مسئول رعایت تمام قوانین قابل اعمال و شرایط خدمات هر وب‌سایتی که دریافت می‌کنید هستید.

---

# ⚖️ 10. Disclaimer / سلب مسئولیت

**English**  

> **THE SOFTWARE (WORKFLOW AND DOCUMENTATION) IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.**

You alone bear responsibility for any use of this tool that violates laws or third‑party rights.

**فارسی**  

> **نرم‌افزار (workflow و مستندات) «همان‌طور که هست» و بدون هرگونه ضمانت، صریح یا ضمنی، از جمله اما نه محدود به ضمانت‌های قابلیت فروش، تناسب برای یک هدف خاص و عدم نقض حقوق دیگران ارائه می‌شود. در هیچ حالتی نویسندگان یا دارندگان کپی‌رایت در قبال هیچ ادعا، خسارت یا مسئولیت دیگری، خواه در یک اقدام قراردادی یا تخلفی، ناشی از یا در ارتباط با نرم‌افزار یا استفاده یا معاملات دیگر در نرم‌افزار مسئول نخواهند بود.**

تنها خودتان مسئول هرگونه استفاده از این ابزار که قوانین یا حقوق اشخاص ثالث را نقض کند، هستید.

---

# 📜 11. License (GNU GPLv3) / مجوز GNU GPLv3

**English**  
RepoRelay is free software: you can redistribute it and/or modify it under the terms of the **GNU General Public License as published by the Free Software Foundation, either version 3 of the License, or (at your option) any later version.**

This program is distributed in the hope that it will be useful, but WITHOUT ANY WARRANTY; without even the implied warranty of MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE. See the GNU General Public License for more details.

You should have received a copy of the GNU General Public License along with this program. If not, see <https://www.gnu.org/licenses/>.

**فارسی**  
RepoRelay نرم‌افزاری آزاد است: می‌توانید آن را تحت شرایط **مجوز عمومی همگانی گنو (GNU GPL) نسخه ۳** یا هر نسخه بعدتر (به انتخاب خود) توزیع و تغییر دهید.

این برنامه به این امید توزیع می‌شود که مفید باشد، اما **هیچ ضمانتی** ندارد؛ حتی ضمانت ضمنی قابلیت فروش یا مناسب بودن برای یک هدف خاص. برای جزئیات بیشتر، متن کامل مجوز GPL را مطالعه کنید.

نسخه‌ای از مجوز GPL باید به همراه این برنامه دریافت کرده باشید. در غیر این صورت، به <https://www.gnu.org/licenses/> مراجعه کنید.

---

<div align="center">

## 📡 Fetch Any URL – Host on GitHub Pages / دریافت هر آدرس – میزبانی روی GitHub Pages

**No installation. No local server. Just a few clicks.**  
**نیاز به نصب ندارد. بدون سرور محلی. فقط چند کلیک.**

**Use responsibly. Respect others’ rights.**  
**مسئولانه استفاده کنید. به حقوق دیگران احترام بگذارید.**

Version 1.0  
Architecture: GitHub Actions → Pages relay  
License: GNU General Public License v3.0

</div>
