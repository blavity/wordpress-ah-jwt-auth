# AH JWT Auth

A WordPress plugin for SSO authentication via JSON Web Tokens (JWT) passed through HTTP headers.

## Introduction

This plugin enables Single Sign-On (SSO) for WordPress by validating JWTs in configurable HTTP headers. It's designed to work with reverse proxies and Identity-Aware Proxies that handle authentication upstream.

**Key Features:**
- 🔐 JWT-based authentication via HTTP headers
- ☁️ **Cloudflare Access integration** (recommended)
- 🔑 Support for both HS256 (shared secret) and RS256 (JWKS) algorithms
- 🔄 Automatic user provisioning and role management
- ⚙️ Configurable HTTP header support
- 📦 Available on [WordPress.org](https://wordpress.org/plugins/ah-jwt-auth/)

### Cloudflare Access Support

This plugin works seamlessly with [Cloudflare Access](https://www.cloudflare.com/products/zero-trust/access/):
- Configure the JWT header to `Cf-Access-Jwt-Assertion`
- Set your Cloudflare team's JWKS URL (e.g., `https://yourteam.cloudflareaccess.com/cdn-cgi/access/certs`)
- Automatic public key retrieval and daily refresh
- RS256 signature verification

This plugin assumes that it can retrieve a valid JWT from a configured HTTP header (the default is the `Authorization` header), so all requests should come from a reverse proxy that enforces authentication/access.

## How it works

* The plugin retrieves the user id (email) from the JWT and then checks if such a user exists. If not, the plugin creates a new user by using this email and signs him/her in.
* If a `role` claim is included in the JWT this will be assigned to the user.
* The plugin expects the JWT is passed as a HTTP header (default is `Authorization`). For example, the payload of JWT may look like:  

```json
{
  "email": "admin@example.com",
  "role": "admin"
}
```

### User Creation

Users are created with a random password (64 characters long), which effectively means access only via SSO is possible.

During the creation process the user is assigned the configured default role or the role from the "role" claim in the JWT if it was included.

In addition the SSO process will set an existing users role to match the "role" claim in the JWT if it was present.

## Supported Authentication Providers

This plugin works with any reverse proxy or Identity-Aware Proxy that can inject JWTs into HTTP headers:

- ✅ **Cloudflare Access** (recommended - full JWKS support)
- ✅ Google Cloud Identity-Aware Proxy (IAP)
- ✅ Azure AD Application Proxy
- ✅ Auth0
- ✅ Okta
- ✅ Custom JWT solutions

## Installation

1. Install via WordPress.org plugin directory or upload to `/wp-content/plugins/`
2. Activate the plugin
3. Go to Settings → AH JWT Auth
4. Configure your JWT settings:
   - **For Cloudflare Access**: See **[Cloudflare Access Setup Guide](CLOUDFLARE_ACCESS_SETUP.md)** for step-by-step instructions
   - **For shared secret**: Enter your HS256 secret key
   - Set default user role and other preferences

## Documentation

📚 **Setup and Configuration:**
- **[Cloudflare Access Setup Guide](CLOUDFLARE_ACCESS_SETUP.md)** - Complete setup instructions for Cloudflare Access with troubleshooting and security best practices

📊 **Project Research and Decisions:**
- **[Research Summary](RESEARCH_SUMMARY.md)** - TL;DR executive summary of licensing analysis, build system evaluation, and fork decision
- **[Fork Research](FORK_RESEARCH.md)** - Comprehensive analysis of fork relationship, GPL licensing compliance, and recommendations

## Credits

This plugin is based on code originally from [datawiza-inc/wordpress-proxy-auth-plugin](https://github.com/datawiza-inc/wordpress-proxy-auth-plugin), significantly refactored and enhanced with:
- Generic multi-provider support
- JWKS (JSON Web Key Set) integration
- RS256 algorithm support
- Cloudflare Access optimization
- WordPress.org deployment infrastructure
- Enhanced security and error handling

**Original Author**: [Andrew Heberle](https://github.com/andrewheberle/wordpress-ah-jwt-auth)  
**License**: GPL-3.0

## Project Status

This is a **hard fork** maintained independently from the original upstream project. The original datawiza-inc project was last updated in October 2021 and was focused on a specific vendor solution. This fork has evolved substantially with a focus on:
- Generic JWT authentication for any provider
- Enhanced Cloudflare Access support
- Active maintenance and security updates
- WordPress.org plugin directory presence
