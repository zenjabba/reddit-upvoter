# Getting Your TOTP Secret for 2FA

If you use two-factor authentication (2FA) on Reddit, you'll need to provide your **TOTP secret** to allow the bot to automatically generate 2FA codes.

## What is a TOTP Secret?

When you set up 2FA on Reddit, you scan a QR code with your authenticator app. That QR code contains a secret key (called the TOTP secret) that your app uses to generate the rotating 6-digit codes. We need that same secret so the bot can generate codes automatically.

The TOTP secret is a **base32-encoded string** that looks like:
- `JBSWY3DPEHPK3PXP`
- `NB2HI4DTHIXS6Y3PNRQXQZJAMFZXM3DPEB3W64TMMV2GS3LFHU======`

---

## Method 1: Get Secret from Your Password Manager

If you store your Reddit 2FA in a password manager, you can usually view the secret:

### 1Password

1. Open your Reddit login in 1Password
2. Look for the "One-Time Password" field
3. Click the field or right-click and select "Copy Secret Key"
4. The secret will look like `otpauth://totp/...?secret=YOURSECRETHERE`
5. Extract just the secret part after `secret=`

### Bitwarden

1. Open your Reddit login in Bitwarden
2. Look for "Authenticator Key (TOTP)"
3. Click the eye icon to reveal the secret
4. Copy the secret key (not the 6-digit code)

### LastPass

1. Open your Reddit login in LastPass
2. Edit the entry
3. Look for "Multifactor Password" or "TOTP Seed"
4. Copy the seed/secret value

### Keeper

1. Open your Reddit entry
2. Go to "Two-Factor Code"
3. Click to view the secret key
4. Copy the raw secret (not the QR code)

---

## Method 2: Get Secret from Your Authenticator App

### Authy (Desktop/Mobile)

Authy encrypts secrets and doesn't easily allow exporting. You'll need to use Method 3 (reset 2FA).

### Google Authenticator

**On Android:**
1. Open Google Authenticator
2. Tap the three dots (⋮) on the Reddit entry
3. Select "Export accounts"
4. Export with or without a QR code
5. Use a QR code reader app to decode it
6. Extract the secret from the `otpauth://` URL

**On iOS:**
Unfortunately, Google Authenticator doesn't show the secret. You'll need to use Method 3.

### Microsoft Authenticator

Microsoft Authenticator doesn't easily show secrets. You'll need to use Method 3 (reset 2FA).

### 2FAS Auth

1. Open 2FAS Auth
2. Long-press on the Reddit entry
3. Tap "Reveal secret key" or "Show seed"
4. Copy the secret

### Aegis Authenticator (Android)

1. Open Aegis
2. Tap and hold the Reddit entry
3. Select "Advanced" or tap the pencil icon
4. View the "Secret" field
5. Copy the secret

### Raivo OTP (iOS)

1. Open Raivo OTP
2. Tap on the Reddit entry
3. Tap "Edit"
4. View the "Secret" field
5. Copy the secret

---

## Method 3: Reset Reddit 2FA to Get the Secret

If you can't get your existing secret, you'll need to reset your 2FA:

### ⚠️ Important: Before You Start

- Make sure you can access your email (for recovery codes)
- Keep your current 2FA app running until you confirm the new setup works
- Have a backup authentication method ready

### Steps:

1. **Disable Current 2FA**
   - Go to https://www.reddit.com/prefs/update/
   - Scroll to "Two-Factor Authentication"
   - Click "Click to disable"
   - Enter your current 2FA code
   - Confirm disabling

2. **Re-enable 2FA and Save the Secret**
   - On the same page, click "Click to enable"
   - Reddit will show a QR code
   - **Below the QR code**, you'll see a text string (this is your TOTP secret!)
   - **Copy this secret and save it securely** - this is what you need

3. **Add to Your Authenticator App**
   - Scan the QR code with your authenticator app as usual
   - Verify it works by generating a code

4. **Save Backup Codes**
   - Reddit will show backup codes
   - Save these in a secure location

5. **Add Secret to Bot Config**
   - Add the TOTP secret to your `config.yaml`:
   ```yaml
   reddit:
     password: "YOUR_PASSWORD"  # Just your password, no :123456
     totp_secret: "YOUR_TOTP_SECRET_HERE"
   ```

---

## Method 4: Use a Browser Extension to Extract

### Using a QR Code Scanner Extension

If you're re-enabling 2FA on Reddit:

1. Install a QR code reader browser extension (like "QR Code Reader" for Chrome)
2. When Reddit shows the 2FA QR code, use the extension to scan it
3. The extension will show the `otpauth://` URL
4. Extract the secret from the URL:
   ```
   otpauth://totp/Reddit:username?secret=YOURSECRETHERE&issuer=Reddit
   ```
5. Copy the part after `secret=` and before `&` (or end of URL)

---

## Verifying Your TOTP Secret

Once you have your secret, you can verify it works:

### Using Python (Quick Test)

```bash
pip install pyotp
python3 -c "import pyotp; print(pyotp.TOTP('YOUR_SECRET_HERE').now())"
```

This should output a 6-digit code that matches your authenticator app.

### Using the Bot's Test Mode

```bash
# Add totp_secret to config.yaml first
python reddit_upvoter.py --test
```

If authentication succeeds, your TOTP secret is correct!

---

## Security Best Practices

1. **Keep Your Secret Safe**
   - The TOTP secret is as sensitive as your password
   - Store it in a password manager
   - Never share it or commit it to git

2. **Use Strong Encryption**
   - If storing in a file, encrypt it
   - Consider using environment variables or a secrets manager

3. **Backup Your Secret**
   - If you lose the secret, you'll need to reset 2FA
   - Store a backup in a secure, separate location

4. **Rotate Regularly**
   - Consider resetting your 2FA periodically
   - Update the secret in your config when you do

---

## Troubleshooting

### "Authentication failed" Error

- **Time Sync Issue**: Make sure your server's clock is accurate
  ```bash
  # On Linux
  sudo timedatectl set-ntp true
  ntpdate -q pool.ntp.org
  ```

- **Wrong Secret**: Double-check you copied the entire secret
- **Spaces in Secret**: Remove any spaces from the secret
- **Wrong Format**: Make sure it's the base32 secret, not the `otpauth://` URL

### "Invalid 2FA code" Error

- Your server's time might be off by more than 30 seconds
- The secret might be incorrect
- You might be using the wrong encoding

### Still Having Issues?

1. Reset your 2FA completely and get a fresh secret (Method 3)
2. Use the test mode to verify: `python reddit_upvoter.py --test`
3. Check the logs for specific error messages
4. Make sure you're using the password WITHOUT the `:123456` suffix when using `totp_secret`

---

## Example Configuration

Once you have your TOTP secret, your config should look like:

```yaml
reddit:
  client_id: "abc123def456"
  client_secret: "xyz789uvw012"
  username: "your_reddit_username"
  password: "your_actual_password"  # NO :123456 here!
  totp_secret: "JBSWY3DPEHPK3PXP"   # Your TOTP secret
  user_agent: "RedditUpvoter/1.0 by u/your_reddit_username"
```

The bot will automatically generate fresh 2FA codes every 30 seconds!
