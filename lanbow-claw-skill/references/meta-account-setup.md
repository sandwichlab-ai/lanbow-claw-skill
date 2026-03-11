# Meta Ad Account Setup Guide

From scratch: Register a Meta developer account → Create an app → Get App ID & App Secret → Get an Access Token → Complete configuration in the lanbow-ads CLI.

## Agent Behavior

**When the user needs to set up their Meta ad account, guide them through Steps 1-2 (done in the browser by the user), then ask them to provide their credentials. Once they give you the App ID, App Secret, Access Token, or Ad Account ID, run the corresponding CLI commands on their behalf:**

```bash
# User provides App ID and App Secret → you run:
lanbow-ads config set --app-id <APP_ID> --app-secret <APP_SECRET>

# User provides Access Token → you run:
lanbow-ads auth set-token <ACCESS_TOKEN>

# User has App Secret configured and wants to extend token → you run:
lanbow-ads auth exchange --token <SHORT_LIVED_TOKEN>

# User provides Ad Account ID → you run:
lanbow-ads config set --account <AD_ACCOUNT_ID>
```

**Do not ask the user to run these commands themselves. You have the CLI available — execute the commands directly once the user provides the values.**

If the user prefers to use the OAuth browser flow instead of providing a token directly, run `lanbow-ads auth login` for them and guide them through the browser authorization steps.

---

## Prerequisites

- A Facebook personal account (to log in to the developer platform)
- A Meta ad account (either Business Manager or a personal ad account)

> If you don't have an ad account yet, log in to [Meta Business Suite](https://business.facebook.com/) and follow the prompts to create one.

---

## Step 1: Register a Meta Developer Account

1. Go to [developers.facebook.com](https://developers.facebook.com/)
2. Click **Get Started** in the top right corner
3. Log in with your Facebook account
4. Agree to the developer platform terms
5. Verify your account (phone number verification may be required)

Once complete, you will be taken to the developer Dashboard.

---

## Step 2: Create an App and Get App ID & App Secret

### 2.1 Create an App

1. In the developer Dashboard, click **Create App**
2. Select use case: **Other**, then click Next
3. Select app type: **Business**, then click Next
4. Fill in the app details:
   - **App Name**: Choose any name, e.g. `My Ads Tool`
   - **App Contact Email**: Your email address
   - **Business Portfolio**: Select your Business Manager if you have one, otherwise skip
5. Click **Create App**

### 2.2 Get the App ID

After creation, you will be redirected to the app Dashboard. Your **App ID** is displayed at the top of the page — it's a numeric string like `1234567890123456`.

### 2.3 Get the App Secret

1. In the left menu, click **App Settings** → **Basic**
2. Find the **App Secret** field and click **Show** (you'll need to enter your Facebook password to confirm)
3. Copy the secret key

> **The App Secret is sensitive information. Do not share it with anyone or commit it to a code repository.**

### 2.4 Enable the Marketing API

1. In the left menu, click **Add Product** (or scroll down on the app Dashboard to find the product list)
2. Find **Marketing API** and click **Set Up**
3. Enable it — no additional configuration is needed

### 2.5 Configure the OAuth Redirect URI

1. In the left menu, click **Facebook Login** → **Settings**
   - If you don't see Facebook Login, add it first via **Add Product**
2. In the **Valid OAuth Redirect URIs** field, add:
   ```
   http://localhost:8080/callback
   ```
3. Click **Save Changes**

---

## Step 3: Configure the lanbow-ads CLI

### 3.1 Save the App ID and App Secret

```bash
lanbow-ads config set --app-id YOUR_APP_ID --app-secret YOUR_APP_SECRET
```

For example:
```bash
lanbow-ads config set --app-id 1234567890123456 --app-secret a1b2c3d4e5f6a1b2c3d4e5f6
```

### 3.2 Verify the Configuration

```bash
lanbow-ads config list
```

The output should display your app_id and app_secret (the secret will be partially masked).

---

## Step 4: Get an Access Token (OAuth Login)

### 4.1 Log In

```bash
lanbow-ads auth login
```

This command will:
1. Start a temporary local HTTP server (port 8080)
2. Automatically open your browser to the Facebook OAuth authorization page
3. You confirm the authorization in the browser
4. The browser redirects back to the local server, and the CLI receives and saves the Access Token

### 4.2 Authorization Page

In the browser, you will see the Facebook authorization page:

1. Confirm your identity (the logged-in Facebook account)
2. Select the ad accounts and Facebook Pages to authorize
3. Check the required permissions (typically select all)
4. Click **Continue**

After authorization is complete, the browser will show a success page. You can close the browser.

### 4.3 Extend Token Validity (Recommended)

The OAuth flow provides a short-lived token by default (valid for about 1-2 hours). Use the following command to exchange it for a long-lived token (valid for about 60 days):

```bash
lanbow-ads auth exchange --token YOUR_SHORT_LIVED_TOKEN
```

> If you have already configured the App Secret before running `auth login`, the CLI will automatically attempt to exchange for a long-lived token.

### 4.4 Check Login Status

```bash
lanbow-ads auth status
```

Example output:
```
Status: Authenticated
Token Source: OAuth (cached)
User ID: 123456789
Expires: 2026-05-09 (in 59 days)
```

---

## Step 5: Set the Default Ad Account

### 5.1 View Your Accessible Ad Accounts

```bash
lanbow-ads accounts list
```

Example output:
```
┌──────────────────┬─────────────────────┬──────────┐
│ id               │ name                │ status   │
├──────────────────┼─────────────────────┼──────────┤
│ act_123456789    │ My Business Account │ ACTIVE   │
│ act_987654321    │ Test Account        │ ACTIVE   │
└──────────────────┴─────────────────────┴──────────┘
```

### 5.2 Set the Default Account

```bash
lanbow-ads config set --account act_123456789
```

All subsequent commands will automatically use this account — no need to pass `--account` every time.

### 5.3 Set Account Aliases (Optional)

If you manage multiple accounts, you can assign aliases:

```bash
lanbow-ads config accounts add main act_123456789 --label "Primary Account"
lanbow-ads config accounts add test act_987654321 --label "Test Account"
```

Switch between accounts using aliases:

```bash
lanbow-ads campaigns list --account main
lanbow-ads campaigns list --account test
```

---

## Step 6: Verify Everything Works

Run the following commands. If they all return results without errors, your setup is complete:

```bash
# Check authentication status
lanbow-ads auth status

# View account info
lanbow-ads accounts list

# View campaign list (may be empty, but should not error)
lanbow-ads campaigns list

# View available Facebook Pages
lanbow-ads pages list
```

---

## Quick Reference

```bash
# 1. Save App credentials
lanbow-ads config set --app-id YOUR_APP_ID --app-secret YOUR_APP_SECRET

# 2. Log in to get a token
lanbow-ads auth login

# 3. Set the default ad account
lanbow-ads config set --account act_XXXXXXXXX

# 4. Verify
lanbow-ads auth status
lanbow-ads accounts list
```

---

## FAQ

### "App Not Set Up" or "Invalid App ID"

Confirm your App ID is correct:

```bash
lanbow-ads config get meta_app_id
```

If it's wrong, reset it:

```bash
lanbow-ads config set --app-id CORRECT_APP_ID
```

### CLI Did Not Receive the Callback After Browser Authorization

1. Check if port 8080 is in use:
   ```bash
   lsof -i :8080
   ```
2. If it's occupied, terminate the process using that port and retry `lanbow-ads auth login`

### "Error validating access token: Session has expired"

The token has expired. Log in again:

```bash
lanbow-ads auth login --force
```

### Cannot Find Ad Accounts

Confirm that your Facebook account has access to ad accounts. Check in [Meta Business Suite](https://business.facebook.com/settings/) that your account is linked to an ad account.

### Insufficient Permissions ("Permissions error")

Log out and log in again, making sure to check all permissions during authorization:

```bash
lanbow-ads auth logout
lanbow-ads auth login
```

On the browser authorization page, expand the permissions list and ensure `ads_management`, `ads_read`, `pages_show_list`, and other required permissions are all checked.
