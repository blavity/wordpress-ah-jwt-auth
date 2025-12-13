# Fork Research: Licensing and Build System Analysis

## Executive Summary

This document provides a comprehensive analysis of the wordpress-ah-jwt-auth project, examining its licensing, build system, and relationship to the upstream project. Based on this research, **a recommendation is made to maintain the hard fork** with potential upstream contributions on a case-by-case basis.

## 1. License Analysis

### 1.1 Current License
- **License Type**: GNU General Public License v3.0 (GPL-3.0)
- **Copyright**: Copyright (C) 2021 Andrew Heberle
- **License URI**: https://www.gnu.org/licenses/gpl-3.0.html

### 1.2 Upstream License
- **Project**: datawiza-inc/wordpress-proxy-auth-plugin
- **License Type**: GNU General Public License v2.0 (GPL-2.0)
- **Note**: The upstream project header indicates "MPL-2.0 License" in code comments but the LICENSE file contains GPL-2.0

### 1.3 License Compatibility Assessment

#### GPL-2.0 to GPL-3.0 Upgrade
✅ **Compatible and Compliant**

The current fork's GPL-3.0 license is fully compatible with the upstream GPL-2.0:
- GPL-2.0 allows "or any later version" interpretation
- GPL-3.0 is a later version that maintains all copyleft requirements
- The license upgrade is legally sound and appropriate

#### Key GPL-3.0 Requirements Met:
1. ✅ Source code is publicly available
2. ✅ License file (LICENSE) is present
3. ✅ Copyright notices are maintained
4. ✅ Modified versions are clearly marked as changed
5. ✅ GPL-3.0 license is applied to the entire work

## 2. Fork Relationship Analysis

### 2.1 Fork Chain and Timeline

The fork relationship is as follows:

1. **datawiza-inc/wordpress-proxy-auth-plugin** (Original Upstream)
   - **Created**: December 2020
   - **Last Update**: October 2021 (4+ years ago)
   - **Status**: Dormant/unmaintained
   - **Purpose**: Vendor-specific plugin for Datawiza Access Broker

2. **andrewheberle/wordpress-ah-jwt-auth** (Intermediate Fork)
   - **Forked From**: datawiza-inc/wordpress-proxy-auth-plugin
   - **Created**: April 2021
   - **Last Update**: March 2025 (actively maintained)
   - **Author**: Andrew Heberle
   - **Purpose**: Generic JWT authentication plugin with enhanced features

3. **blavity/wordpress-ah-jwt-auth** (Current Repository)
   - **Forked From**: andrewheberle/wordpress-ah-jwt-auth
   - **Current Maintainer**: Blavity
   - **Status**: Active development
   - **Purpose**: Continuing generic JWT authentication with Cloudflare Access focus

### 2.2 Key Observations

- **Original upstream (datawiza-inc)** has been dormant since October 2021
- **Intermediate fork (andrewheberle)** remains actively maintained through 2025
- **Current fork (blavity)** is building on andrewheberle's work, not directly on datawiza-inc

### 2.3 Key Differences from Upstream

#### Code Changes
- **Scale**: ~231 lines (upstream) → ~569 lines (current) = 2.5x growth
- **Namespace**: `Datawiza` → `AhJwtAuth`
- **Plugin Name**: "Datawiza Proxy Auth Plugin" → "AH JWT Auth"
- **Architecture**: Complete refactoring and generalization

#### Feature Additions
1. **JWKS (JSON Web Key Set) Support**
   - Automatic retrieval and caching of public keys
   - Daily refresh via WP-Cron
   - Support for RS256 algorithm (in addition to HS256)

2. **Configurable HTTP Headers**
   - Default: `Authorization` header (standard)
   - Configurable for any header (e.g., `Cf-Access-Jwt-Assertion` for Cloudflare)
   - More flexible than hardcoded `DW-TOKEN`

3. **Enhanced User Role Management**
   - Configurable default role for new users
   - Dynamic role updates based on JWT claims
   - Better WordPress integration

4. **WordPress.org Publishing**
   - Published on WordPress.org plugin directory
   - Automated deployment pipeline
   - Professional plugin structure

5. **Improved Security**
   - Better token verification
   - Enhanced error handling
   - Updated dependencies (firebase/php-jwt ^6.2)

## 3. Build System Analysis

### 3.1 Current Build Infrastructure

#### Dependency Management
- **Tool**: Composer
- **Dependencies**:
  - `firebase/php-jwt: ^6.2` (JWT encoding/decoding)
- **Automation**: Renovate Bot for dependency updates

#### CI/CD Pipeline (GitHub Actions)

1. **WPCS Check** (`.github/workflows/wpcs.yml`)
   - Runs on: Pull Requests
   - Purpose: WordPress Coding Standards validation
   - Tool: 10up/wpcs-action

2. **WordPress.org Deployment** (`.github/workflows/deploy.yml`)
   - Trigger: Git tags
   - Purpose: Automated release to WordPress.org
   - Tool: 10up/action-wordpress-plugin-deploy
   - Requires: SVN credentials (secrets)

3. **Asset/README Update** (`.github/workflows/update.yml`)
   - Trigger: Push to main branch
   - Purpose: Update plugin assets on WordPress.org
   - Tool: 10up/action-wordpress-plugin-asset-update

#### Additional Automation
- **Dependabot**: Weekly Composer dependency checks
- **Renovate**: Additional dependency management

### 3.2 Distribution
- **Primary**: WordPress.org Plugin Directory
- **Plugin Slug**: `ah-jwt-auth`
- **Version**: 1.5.4
- **Exclusions**: `.distignore` file defines non-distributed files

### 3.3 No Upstream Build System
The upstream project (datawiza-inc/wordpress-proxy-auth-plugin):
- ❌ No CI/CD workflows
- ❌ No automated testing
- ❌ No WordPress.org deployment
- ❌ No coding standards enforcement
- ⚠️ Inactive development (last update: Oct 2021)

## 4. Cloudflare Access Integration

### 4.1 Current Support
The current fork already supports Cloudflare Access integration:
- Configurable header support (`Cf-Access-Jwt-Assertion`)
- JWKS retrieval for RSA public key verification
- Compatible with Cloudflare's JWT structure

### 4.2 Cloudflare-Specific Features to Consider

#### Already Implemented
✅ Custom header configuration
✅ RS256 algorithm support (RSA signatures)
✅ JWKS endpoint support
✅ Automatic key refresh

#### Potential Enhancements for Cloudflare
1. **Cloudflare-Specific Claims**
   - `country`, `iat`, `email` already supported
   - Could add: `identity_nonce`, `custom` attributes
   
2. **Cloudflare Teams Integration**
   - Support for group claims
   - Team member synchronization
   - Access group mapping to WordPress roles

3. **Cloudflare Access Policies**
   - Integration with Access application policies
   - Service token support
   - mTLS certificate validation

4. **Enhanced Logging**
   - Cloudflare request ID tracking
   - Audit trail integration
   - Access-specific error handling

## 5. Recommendation: Hard Fork

### 5.1 Primary Recommendation
**✅ MAINTAIN THE HARD FORK**

#### Rationale

1. **Original Upstream (datawiza-inc) is Dormant**
   - No activity since October 2021 (4+ years)
   - Vendor-specific focus (Datawiza)
   - No indication of accepting generic contributions

2. **Intermediate Fork (andrewheberle) is Active but Different Path**
   - While andrewheberle's repo is actively maintained (last update March 2025)
   - Both blavity and andrewheberle repos have diverged from datawiza-inc
   - Blavity fork can pursue Cloudflare-specific enhancements independently

3. **Substantial Divergence from Original**
   - 2.5x code growth from original datawiza-inc
   - Complete architectural changes
   - Different target audience
   - Independent feature roadmap

3. **Different Purposes**
   - Upstream: Datawiza-specific integration
   - Current: Generic JWT auth with multi-provider support
   - Goal: Cloudflare Access focus (vendor-agnostic approach)

4. **Professional Infrastructure**
   - WordPress.org published plugin
   - Automated CI/CD pipeline
   - Active maintenance and security updates
   - Established user base

5. **License Compatibility**
   - GPL-3.0 is compatible with upstream GPL-2.0
   - No legal barriers to independent development
   - Proper attribution maintained

### 5.2 Recommended Actions

#### Immediate Actions
1. ✅ **Update Documentation**
   - Clarify fork relationship in README.md
   - Document differences from upstream
   - Highlight Cloudflare Access support

2. ✅ **Maintain Attribution**
   - Keep "Credits" section referencing upstream
   - Maintain copyright notices
   - Ensure GPL-3.0 compliance

3. ✅ **Independent Branding**
   - Continue using "AH JWT Auth" name
   - Maintain separate identity
   - Build independent reputation

#### For Cloudflare Access Features

**Option A: Enhance Current Fork (Recommended)**
- Add Cloudflare-specific features to current plugin
- Maintain generic compatibility
- Document Cloudflare Access best practices
- Create Cloudflare Access setup guide

**Option B: Create Cloudflare-Specific Extension**
- Keep core plugin generic
- Create companion plugin for Cloudflare-specific features
- Allows modularity and separation of concerns

### 5.3 Upstream Contribution Strategy

**Regarding datawiza-inc (Original Upstream)**
Given 4+ years of inactivity, contributions to datawiza-inc are **not recommended**.

**Regarding andrewheberle (Intermediate Fork)**
Andrew Heberle's fork remains actively maintained (last update March 2025). Considerations:
- Both repos (blavity and andrewheberle) have evolved independently
- Collaboration could be considered for generic JWT features
- However, Cloudflare-specific features should remain in blavity fork
- No obligation to contribute back given the fork's independent direction

**Recommendation**: 
- Maintain independent development for Cloudflare Access features
- Consider collaboration with andrewheberle only for generic JWT improvements that benefit both forks
- No action required toward dormant datawiza-inc upstream

## 6. Build System Recommendations

### 6.1 Current Build System: ✅ Excellent

The existing build system is well-architected:
- Automated quality checks (WPCS)
- Automated deployment (WordPress.org)
- Dependency management (Composer, Renovate, Dependabot)
- Version control and tagging

### 6.2 Recommended Enhancements

#### Testing Infrastructure
```yaml
# Suggested: .github/workflows/tests.yml
- PHPUnit tests for core functionality
- Integration tests for JWT validation
- WordPress version compatibility tests
- PHP version compatibility matrix (7.4-8.3)
  Note: PHP 7.0-7.3 are EOL; recommend PHP 7.4+ per WordPress requirements
```

#### Security Scanning
```yaml
# Suggested: .github/workflows/security.yml
- Automated vulnerability scanning
- Dependency security checks
- SAST (Static Application Security Testing)
```

#### Documentation Generation
```yaml
# Suggested: .github/workflows/docs.yml
- Automated API documentation (phpDocumentor)
- Cloudflare Access setup guides
- Configuration examples
```

## 7. Legal Compliance Checklist

### 7.1 GPL-3.0 Requirements
- [x] Source code publicly available
- [x] LICENSE file present and correct
- [x] Copyright notices maintained
- [x] Modified works marked as changed
- [x] Same license applied to derivative works
- [x] No additional restrictions imposed

### 7.2 Attribution Requirements
- [x] Original author credited (Andrew Heberle)
- [x] Upstream project referenced in README
- [x] License headers in source files
- [x] Copyright year and owner declared

### 7.3 WordPress.org Requirements
- [x] GPL-compatible license
- [x] Plugin header correctly formatted
- [x] readme.txt follows WordPress format
- [x] Proper versioning and changelog

## 8. Cloudflare Access Implementation Plan

### 8.1 Phase 1: Documentation (Immediate)
- [ ] Create comprehensive Cloudflare Access setup guide
- [ ] Document Cf-Access-Jwt-Assertion header configuration
- [ ] Provide example JWKS endpoint URLs
- [ ] Add troubleshooting section for Cloudflare-specific issues

### 8.2 Phase 2: Enhanced Support (Near-term)
- [ ] Add Cloudflare Access application detection
- [ ] Implement Cloudflare-specific claim validation
- [ ] Add support for Cloudflare custom claims
- [ ] Create Cloudflare Access preset configuration option

### 8.3 Phase 3: Advanced Features (Future)
- [ ] Cloudflare Teams integration
- [ ] Group-based role mapping
- [ ] Service token support
- [ ] Access audit log integration

## 9. Conclusion

**Final Recommendation**: ✅ **HARD FORK (Current Strategy)**

The wordpress-ah-jwt-auth project has evolved substantially beyond its upstream origins and serves a different purpose with a different target audience. The upstream project appears dormant, and the current fork has:

1. ✅ Superior architecture and features
2. ✅ Active maintenance and security updates
3. ✅ Professional build and deployment infrastructure
4. ✅ WordPress.org presence and user community
5. ✅ Legal compliance with GPL-3.0
6. ✅ Cloudflare Access compatibility (with room for enhancement)

**No upstream submission is recommended**. Instead, continue independent development with a focus on:
- Generic JWT authentication
- Enhanced Cloudflare Access support
- Multi-provider compatibility
- Professional WordPress plugin standards

The current approach is sound, legally compliant, and technically superior. Maintain the hard fork and continue building value independently.

---

## Document Metadata
- **Prepared**: 2025-12-13
- **Author**: Repository Analysis
- **Status**: Final Recommendation
- **License**: GPL-3.0 (consistent with project)
