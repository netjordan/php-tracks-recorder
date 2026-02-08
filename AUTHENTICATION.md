# HTTP Authentication Setup

This guide explains how to set up HTTP Basic Authentication for your Tracks Recorder installation on cPanel with LiteSpeed.

## Files Created

- `.htaccess` - Authentication configuration
- `.htpasswd.sample` - Password file template (rename to `.htpasswd`)

## Setup Instructions

### Step 1: Update .htaccess

Edit the `.htaccess` file and update the `AuthUserFile` path:

```apache
AuthUserFile /home/YOUR_CPANEL_USERNAME/public_html/YOUR_PATH/.htpasswd
```

Replace:
- `YOUR_CPANEL_USERNAME` with your cPanel username
- `YOUR_PATH` with the path to your installation (e.g., `tracks` or leave empty if in root)

**Example:**
```apache
AuthUserFile /home/john123/public_html/tracks/.htpasswd
```

To find your full path, you can create a temporary PHP file with:
```php
<?php echo __DIR__; ?>
```

### Step 2: Create .htpasswd File

You have several options to create the password file:

#### Option A: Using cPanel (Recommended for shared hosting)

1. Log into your cPanel
2. Go to "Directory Privacy" (or "Password Protect Directories")
3. Navigate to your tracks recorder directory
4. Click "Edit" and enable password protection
5. Add users and passwords through the interface
6. cPanel will automatically create the `.htpasswd` file

#### Option B: Using Command Line (if you have SSH access)

```bash
# Navigate to your directory
cd /home/YOUR_CPANEL_USERNAME/public_html/YOUR_PATH/

# Create password for user
htpasswd -c .htpasswd username

# Add additional users (without -c flag)
htpasswd .htpasswd another_username
```

#### Option C: Using Online Generator

1. Visit a htpasswd generator (search "htpasswd generator")
   - Example: https://hostingcanada.org/htpasswd-generator/
2. Enter your desired username and password
3. Copy the generated line
4. Create `.htpasswd` file with the content:
   ```
   username:$apr1$xyz12345$abcdefghijklmnopqrstuvwxyz
   ```

#### Option D: Using PHP Script (temporary)

Create a temporary file `generate_htpasswd.php`:

```php
<?php
$username = 'admin';
$password = 'your_password_here';
$encrypted = crypt($password, base64_encode($password));
echo "$username:$encrypted\n";

// Alternative using password_hash (PHP 5.5+)
// Note: Some servers may not accept this format
// echo "$username:" . password_hash($password, PASSWORD_BCRYPT) . "\n";
?>
```

Run it, copy the output to `.htpasswd`, then **DELETE the PHP file immediately**.

### Step 3: Set Correct Permissions

Set the correct file permissions:

```bash
chmod 644 .htaccess
chmod 644 .htpasswd
```

Or through cPanel File Manager: Right-click → Change Permissions → 644

### Step 4: Test Authentication

1. Visit your tracks recorder URL in a browser
2. You should see a login prompt
3. Enter your username and password
4. The interface should load after successful authentication

### Important Notes

#### Record Endpoint (Mobile Apps)

The `.htaccess` file is configured to **allow `record.php` without authentication**. This means:
- ✅ Mobile apps (Owntracks/Overland) can submit data without login
- ✅ The web interface (index.php) requires authentication
- ✅ RPC calls (rpc.php) require authentication

If you want to also protect `record.php`, remove these lines from `.htaccess`:
```apache
<Files "record.php">
    Satisfy Any
    Allow from all
</Files>
```

Then configure HTTP Basic Auth in your mobile app settings.

#### Multiple Users

To add multiple users to `.htpasswd`:
```
admin:$apr1$xyz12345$abcdefghijklmnopqrstuv
user2:$apr1$abc67890$zyxwvutsrqponmlkjihgfe
user3:$apr1$def24680$mnbvcxzasdfghjklpoiuy
```

Each line is one user. All users will have full access to the application.

#### Security Considerations

1. **HTTPS Recommended**: HTTP Basic Auth sends credentials in base64 (easily decoded). Always use HTTPS in production.
2. **Strong Passwords**: Use strong, unique passwords for each user.
3. **Keep .htpasswd Secret**: Never commit `.htpasswd` to version control (already in `.gitignore`).
4. **Regular Updates**: Change passwords periodically.

#### Troubleshooting

**500 Internal Server Error:**
- Check the `AuthUserFile` path is correct (absolute path, not relative)
- Verify file permissions (644 for both files)
- Check LiteSpeed error logs in cPanel

**Authentication not working:**
- Clear browser cache and cookies
- Try incognito/private browsing mode
- Verify `.htpasswd` format (username:encrypted_password)

**Mobile app can't submit data:**
- If you protected `record.php`, configure HTTP auth in your app
- For Owntracks: Settings → Connection → Authentication
- For Overland: Add username:password@ to URL: `https://user:pass@yourdomain.com/record.php`

## Removing Authentication

To remove authentication:
1. Delete or rename `.htaccess`
2. Delete `.htpasswd`

Or comment out the auth lines in `.htaccess`:
```apache
# AuthType Basic
# AuthName "Tracks Recorder - Login Required"
# AuthUserFile /home/.../.htpasswd
# Require valid-user
```
