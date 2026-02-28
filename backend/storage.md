# Laravel Storage Link Fix Guide

## 📌 Problem

When images or files uploaded in Laravel are not visible in production,
usually the **storage link** is missing.

Example issue: - Images upload successfully - Files exist inside
`storage/app/public` - But URLs return **404 Not Found**

------------------------------------------------------------------------

## ✅ Solution (Step‑by‑Step)

### 1️⃣ Run Storage Link Command

Execute this command inside your Laravel project root:

``` bash
php artisan storage:link
```

This creates a symbolic link:

    public/storage → storage/app/public

------------------------------------------------------------------------

### 2️⃣ Verify Folder Permissions (Linux Server)

``` bash
chmod -R 775 storage
chmod -R 775 bootstrap/cache
```

Or:

``` bash
sudo chown -R www-data:www-data storage bootstrap/cache
```

------------------------------------------------------------------------

### 3️⃣ Check `.env` Configuration

Make sure:

    FILESYSTEM_DISK=public
    APP_URL=https://yourdomain.com

------------------------------------------------------------------------

### 4️⃣ Clear Laravel Cache

``` bash
php artisan config:clear
php artisan cache:clear
php artisan route:clear
php artisan view:clear
```

------------------------------------------------------------------------

### 5️⃣ Verify Image URL

Correct usage:

``` php
asset('storage/image.jpg')
```

Example output:

    https://yourdomain.com/storage/image.jpg

------------------------------------------------------------------------

## 🚀 Deployment Notes (Docker / Dokploy)

If using Docker or Dokploy:

-   Run `php artisan storage:link` **inside container**
-   Ensure volume persistence for `storage` folder
-   Restart container after changes

------------------------------------------------------------------------

## 🔍 Troubleshooting

  Problem                            Solution
  ---------------------------------- ------------------------
  404 error                          Storage link missing
  Permission denied                  Fix folder permissions
  Works locally but not production   Run command on server
  Images load slowly                 Check CDN / caching

------------------------------------------------------------------------

## ✅ Final Checklist

-   [ ] Storage link created
-   [ ] Permissions fixed
-   [ ] Cache cleared
-   [ ] Correct APP_URL set
-   [ ] Public disk configured

------------------------------------------------------------------------

## 🎯 Result

After completing these steps, uploaded files should be publicly
accessible via:

    /storage/filename.ext

------------------------------------------------------------------------

**Author Note:**\
This guide helps fix the most common Laravel production upload
visibility issues.
