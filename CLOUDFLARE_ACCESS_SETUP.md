# Cloudflare Access Setup Guide

This guide walks you through setting up WordPress SSO with Cloudflare Access and the AH JWT Auth plugin.

## Overview

[Cloudflare Access](https://www.cloudflare.com/products/zero-trust/access/) is a Zero Trust security solution that provides identity-aware access control for your applications. When integrated with this plugin, it enables:

- Single Sign-On (SSO) with multiple identity providers
- Zero Trust security model
- No VPN required
- Centralized access policies
- Automatic JWT validation

## Prerequisites

- WordPress site on Cloudflare (orange-clouded DNS)
- Cloudflare Access subscription (Free plan available)
- AH JWT Auth plugin installed and activated
- Admin access to both WordPress and Cloudflare dashboard

## Step 1: Configure Cloudflare Access Application

### 1.1 Create Access Application

1. Log into [Cloudflare Dashboard](https://dash.cloudflare.com/)
2. Navigate to **Zero Trust** → **Access** → **Applications**
3. Click **Add an application**
4. Select **Self-hosted**

### 1.2 Configure Application Details

**Application Configuration:**
```
Application name: WordPress SSO
Session Duration: 24 hours (or as needed)
Application domain: yourdomain.com
Path: Include all paths (/)
```

**Identity Providers:**
- Select your preferred IdP (Azure AD, Okta, Google, etc.)
- Or use Cloudflare's built-in One-time PIN

### 1.3 Create Access Policy

**Create a policy to allow access:**
```
Policy name: WordPress Users
Action: Allow
```

**Include rules** (example):
- Emails ending in: @yourcompany.com
- Or: Specific email addresses
- Or: Groups from your IdP

**Example Policy:**
```
Include:
  - Emails ending in: @blavity.com
  
Exclude:
  - Email: blocked-user@blavity.com
```

### 1.4 Save Application

After saving, Cloudflare will automatically:
- Issue JWTs for authenticated users
- Add the `Cf-Access-Jwt-Assertion` header to requests
- Enforce your access policies

## Step 2: Configure WordPress Plugin

### 2.1 Plugin Settings

Navigate to WordPress Admin → **Settings** → **AH JWT Auth**

### 2.2 Basic Configuration

**JWT Header Name:**
```
Cf-Access-Jwt-Assertion
```
⚠️ **Important**: Use exact capitalization as shown above. HTTP headers in Cloudflare Access are case-sensitive for security reasons. Using incorrect capitalization (e.g., `cf-access-jwt-assertion` or `CF-ACCESS-JWT-ASSERTION`) will cause authentication to fail silently, as the plugin won't find the JWT in the expected header.

**Algorithm:**
```
RS256
```

### 2.3 JWKS Configuration

**JWKS URL:**
```
https://yourteam.cloudflareaccess.com/cdn-cgi/access/certs
```

Replace `yourteam` with your Cloudflare team name from the Zero Trust dashboard.

**To find your team name:**
1. Go to Zero Trust dashboard
2. Look at the URL: `https://dash.teams.cloudflare.com/YOUR_TEAM_NAME/`
3. Or check Settings → General → Team domain

### 2.4 User Settings

**Email Claim:**
```
email
```
(Default - Cloudflare includes this in the JWT)

**Role Claim:**
```
role
```
(Optional - see Advanced Configuration below)

**Default Role:**
```
subscriber
```
(Or your preferred default: editor, author, administrator)

### 2.5 Save Settings

Click **Save Changes**

## Step 3: Verify Configuration

### 3.1 Test Access

1. **Log out** of WordPress admin
2. Visit your WordPress site
3. You should be redirected to Cloudflare Access login
4. Authenticate with your IdP
5. You'll be redirected back and automatically logged into WordPress

### 3.2 Check User Creation

1. Log into WordPress admin (as a separate admin user or via direct database access)
2. Go to **Users**
3. Verify that new users are created automatically
4. Check that email addresses match your authenticated users

### 3.3 Troubleshooting

**Common Issues:**

1. **Not redirected to login:**
   - Ensure Cloudflare DNS is proxied (orange cloud)
   - Verify Application path covers your WordPress URL
   - Check that Access policy is active

2. **Login fails:**
   - Check plugin settings → Enable debug logging
   - Verify JWKS URL is correct
   - Ensure header name is exact: `Cf-Access-Jwt-Assertion`

3. **User not created:**
   - Verify JWT contains `email` claim
   - Check WordPress admin notices for errors
   - Review PHP error logs

## Step 4: Advanced Configuration

### 4.1 Role Mapping from Cloudflare Groups

Cloudflare Access can include group information in the JWT. To map groups to WordPress roles:

**Option A: Using Cloudflare Access Custom Claims**

1. In Cloudflare Access application settings:
   - Go to **JWT configuration**
   - Add custom claim:
     ```
     Name: role
     Value: (based on groups or identity provider)
     ```

2. Create transformation rules to map groups to roles:
   ```
   If group = "wordpress-admins" → role = "administrator"
   If group = "wordpress-editors" → role = "editor"
   Default → role = "subscriber"
   ```

**Option B: Using Cloudflare Workers**

Deploy a Cloudflare Worker to transform the JWT:

```javascript
addEventListener('fetch', event => {
  event.respondWith(handleRequest(event.request))
})

async function handleRequest(request) {
  const jwt = request.headers.get('Cf-Access-Jwt-Assertion')
  
  if (jwt) {
    // Validate JWT structure
    const parts = jwt.split('.')
    if (parts.length !== 3) {
      return new Response('Invalid JWT format', { status: 401 })
    }
    
    try {
      // Decode JWT payload (base64url decode)
      const payload = JSON.parse(atob(parts[1].replace(/-/g, '+').replace(/_/g, '/')))
      
      // Map groups to WordPress roles
      let role = 'subscriber' // default
      
      if (payload.groups && Array.isArray(payload.groups)) {
        if (payload.groups.includes('wordpress-admins')) {
          role = 'administrator'
        } else if (payload.groups.includes('wordpress-editors')) {
          role = 'editor'
        }
      }
      
      // Note: This example shows concept only
      // In production, you would need to:
      // 1. Properly verify the JWT signature
      // 2. Re-sign the JWT with the modified payload
      // 3. Handle errors appropriately
      // 4. Use a proper JWT library like jose
      
      // For production, consider using Cloudflare Access custom claims instead
      
    } catch (e) {
      console.error('JWT parsing error:', e)
      return new Response('Invalid JWT', { status: 401 })
    }
  }
  
  return fetch(request)
}
```

**⚠️ Important**: This is a simplified example for illustration. In production:
- Use a proper JWT library (e.g., `jose` npm package)
- Properly verify JWT signatures before modification
- Handle all error cases
- Consider using Cloudflare Access custom claims instead of modifying JWTs

### 4.2 Service Tokens

For API access or service-to-service authentication:

1. Create Service Token in Cloudflare Access
2. Update Access Policy to include service tokens
3. Use the service token in API requests:
   ```bash
   curl -H "CF-Access-Client-Id: <client-id>" \
        -H "CF-Access-Client-Secret: <client-secret>" \
        https://yourdomain.com/wp-json/
   ```

### 4.3 Multiple Applications

If you have multiple WordPress sites:

1. Create separate Access Applications for each
2. Use the same JWKS URL for all
3. Configure each WordPress instance independently
4. Consider using the same team domain for easier management

## Security Best Practices

### 1. Enforce Cloudflare Proxy
Ensure your WordPress origin server:
- Only accepts connections from Cloudflare IPs
- Blocks direct access attempts
- Uses firewall rules to enforce this

**Example nginx configuration:**
```nginx
# Allow Cloudflare IPs only
include cloudflare-ips.conf;
deny all;
```

### 2. Disable WordPress Login
Since users authenticate via Cloudflare Access:

**Option A: Redirect wp-login.php**
```php
// In wp-config.php or functions.php
add_action('login_init', function() {
    if (!is_user_logged_in()) {
        wp_redirect(home_url());
        exit;
    }
});
```

**Option B: Use plugin settings**
- Some security plugins allow disabling wp-login.php
- Configure to allow SSO-only authentication

### 3. Regular JWKS Refresh
The plugin automatically refreshes JWKS daily, but you can:
- Force refresh via plugin settings
- Monitor refresh errors in admin notices
- Set up alerts for failed JWKS updates

### 4. Session Management
- Set appropriate session duration in Cloudflare Access
- Match WordPress session settings if needed
- Consider requiring re-authentication for admin actions

### 5. Audit Logging
- Enable Cloudflare Access audit logs
- Monitor WordPress user creation events
- Track authentication failures

## Monitoring and Maintenance

### Health Checks

**Check JWKS Status:**
1. Go to Settings → AH JWT Auth
2. View "Last JWKS Refresh" timestamp
3. Click "Refresh JWKS Now" to force update

**Monitor Logs:**
- WordPress Debug Log (if WP_DEBUG enabled)
- Cloudflare Access Audit Logs
- Server access logs for authentication patterns

### Common Maintenance Tasks

**Daily:**
- Automatic JWKS refresh (via WP-Cron)

**Weekly:**
- Review user creation patterns
- Check for authentication errors in logs

**Monthly:**
- Review Cloudflare Access policies
- Audit user roles and permissions
- Update plugin if new version available

## Troubleshooting Guide

### Issue: "Invalid JWT signature"

**Causes:**
- JWKS not refreshed
- Wrong JWKS URL
- Clock skew between servers

**Solutions:**
1. Click "Refresh JWKS Now" in plugin settings
2. Verify JWKS URL: `https://yourteam.cloudflareaccess.com/cdn-cgi/access/certs`
3. Check server time synchronization (NTP)

### Issue: "Email claim not found"

**Causes:**
- JWT doesn't contain email claim
- Email claim name mismatch

**Solutions:**
1. Check JWT contents in Cloudflare Access → Application → JWT preview
2. Verify "Email Claim" setting in plugin matches JWT claim name
3. Ensure your IdP provides email in the authentication flow

### Issue: Users created with wrong role

**Causes:**
- Role claim not in JWT
- Role claim name mismatch
- Invalid role value

**Solutions:**
1. Check "Default Role" setting in plugin
2. Verify role claim exists in JWT
3. Ensure role values match WordPress roles: administrator, editor, author, contributor, subscriber

### Issue: Redirected to Cloudflare but not logged into WordPress

**Causes:**
- Plugin not receiving JWT header
- WordPress caching interfering
- JWT validation failing

**Solutions:**
1. Check WordPress debug log for JWT validation errors
2. Disable caching for logged-in users
3. Verify Cloudflare Access application covers the correct paths
4. Check that origin server receives `Cf-Access-Jwt-Assertion` header

## Example Configurations

### Configuration 1: Basic Setup
```
Header: Cf-Access-Jwt-Assertion
Algorithm: RS256
JWKS URL: https://myteam.cloudflareaccess.com/cdn-cgi/access/certs
Email Claim: email
Role Claim: (empty)
Default Role: subscriber
```

### Configuration 2: Role-Based Setup
```
Header: Cf-Access-Jwt-Assertion
Algorithm: RS256
JWKS URL: https://myteam.cloudflareaccess.com/cdn-cgi/access/certs
Email Claim: email
Role Claim: role
Default Role: subscriber
```
(Requires Cloudflare custom claims or Worker transformation)

### Configuration 3: Multi-Domain Setup
```
Domain 1 (example.com):
  Header: Cf-Access-Jwt-Assertion
  JWKS URL: https://myteam.cloudflareaccess.com/cdn-cgi/access/certs
  Default Role: editor

Domain 2 (blog.example.com):
  Header: Cf-Access-Jwt-Assertion
  JWKS URL: https://myteam.cloudflareaccess.com/cdn-cgi/access/certs
  Default Role: author
```

## Performance Considerations

### JWKS Caching
- JWKS is cached in WordPress transients
- Refresh occurs daily via WP-Cron
- No external API calls on each request (after initial cache)

### Page Caching
When using page caching plugins:
- Exclude logged-in user pages from cache
- Or use cache variations based on authentication
- Consider using Cloudflare's cache with bypass for authenticated users

### Load Balancing
If using multiple WordPress servers:
- JWKS cache is per-server (WordPress transients)
- Each server refreshes independently
- Consider using persistent object cache (Redis/Memcached) for shared cache

## Additional Resources

- [Cloudflare Access Documentation](https://developers.cloudflare.com/cloudflare-one/applications/configure-apps/)
- [JWT.io Debugger](https://jwt.io/) - Debug and inspect JWTs
- [Plugin Support Forum](https://wordpress.org/support/plugin/ah-jwt-auth/)
- [Cloudflare Community](https://community.cloudflare.com/)

## Support

For issues specific to:
- **Plugin functionality**: [WordPress.org support forum](https://wordpress.org/support/plugin/ah-jwt-auth/)
- **Cloudflare Access**: [Cloudflare support](https://support.cloudflare.com/)
- **Integration questions**: [GitHub Issues](https://github.com/blavity/wordpress-ah-jwt-auth/issues)

---

**Last Updated**: 2025-12-13  
**Plugin Version**: 1.5.4+  
**Tested with**: Cloudflare Access (current version)
