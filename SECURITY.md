# Security Policy

**Project:** kap-estate
**Last Updated:** November 20, 2025

---

## 🔐 Security Commitment

Security is a **core principle** of this project. We learned from 5 previous projects where API keys were exposed in git history. This project is built security-first from day one.

---

## 🚨 Critical Security Rules

### 1. **NEVER Commit Secrets**

❌ **Forbidden:**
- `.env` files
- API keys in code
- Passwords in code
- Credentials files (`googl-cred.json`, etc.)
- Private keys (`.key`, `.pem` files)

✅ **Required:**
- All secrets in `.env` (gitignored)
- `.env.example` for documentation (NO real values)
- Environment variable validation on startup
- Pre-commit hooks to prevent accidents

### 2. **API Key Management**

**Obtaining Keys:**
1. Sign up for each required service
2. Generate API keys
3. Store in password manager (1Password, LastPass, etc.)
4. Add to `.env` file (never commit)
5. Document in `.env.example` (without actual values)

**Required API Keys:**
- RapidAPI (Zillow data)
- Google Maps
- WalkScore
- US Census Bureau
- NYC Open Data (optional)

**Key Rotation:**
- Rotate keys every 90 days
- Rotate immediately if suspected exposure
- Document rotation in password manager

### 3. **Git History Protection**

If you accidentally commit a secret:

```bash
# 1. IMMEDIATELY revoke the exposed key
# Visit the service provider and revoke/regenerate

# 2. Remove from git history
git filter-branch --force --index-filter \
  "git rm --cached --ignore-unmatch .env .env.local" \
  --prune-empty --tag-name-filter cat -- --all

# 3. Generate new key and add to .env

# 4. Force push (coordinate with team first!)
git push origin --force --all
```

### 4. **Environment Variables**

**Loading Order:**
1. Read `.env` file
2. Validate all required variables exist
3. Validate format (e.g., JWT_SECRET min 32 chars)
4. Fail fast if any validation fails
5. Never log secret values

**Example Validation:**
```typescript
const requiredVars = [
  'DATABASE_URL',
  'JWT_SECRET',
  'RAPIDAPI_KEY'
];

for (const varName of requiredVars) {
  if (!process.env[varName]) {
    throw new Error(`Missing: ${varName}`);
  }
}

if (process.env.JWT_SECRET.length < 32) {
  throw new Error('JWT_SECRET too short');
}
```

---

## 🛡️ Security Measures Implemented

### Input Validation
- ✅ Zod schemas for all API inputs
- ✅ Type checking with TypeScript strict mode
- ✅ Sanitization of user inputs
- ✅ File upload restrictions (type, size)

### SQL Injection Prevention
- ✅ Prisma ORM with parameterized queries
- ✅ No raw SQL queries (unless absolutely necessary)
- ✅ Input validation before database calls

### XSS Prevention
- ✅ React auto-escaping
- ✅ Content Security Policy headers
- ✅ Sanitize HTML if user-generated content
- ✅ No `dangerouslySetInnerHTML` without sanitization

### CSRF Prevention
- ✅ SameSite cookie attributes
- ✅ CSRF tokens for state-changing operations
- ✅ Validate Origin/Referer headers

### Authentication
- ✅ JWT with strong secret (min 32 chars)
- ✅ Password hashing with bcrypt (cost: 12)
- ✅ Token expiration (24h default)
- ✅ Refresh token mechanism
- ✅ Rate limiting on auth endpoints

### Authorization
- ✅ Role-based access control (RBAC)
- ✅ Middleware to check permissions
- ✅ Audit logging for admin actions
- ✅ Principle of least privilege

### Rate Limiting
- ✅ Per-IP rate limiting on all endpoints
- ✅ Per-API-key rate limiting for external APIs
- ✅ Sophisticated RateLimiter class
- ✅ 429 responses with Retry-After header

### HTTPS
- ✅ TLS 1.3 minimum
- ✅ HSTS headers
- ✅ Redirect HTTP to HTTPS
- ✅ Cloudflare for SSL termination

### Database Security
- ✅ Encrypted connections (SSL)
- ✅ Least privilege database user
- ✅ Regular backups (encrypted)
- ✅ Audit logging enabled

### Monitoring
- ✅ Sentry for error tracking
- ✅ Structured logging (no secrets in logs)
- ✅ Failed auth attempt monitoring
- ✅ Anomaly detection alerts

---

## 🔍 Security Checklist

### Before Every Commit
- [ ] No `.env` files in commit
- [ ] No hardcoded API keys
- [ ] No passwords in code
- [ ] No credentials files
- [ ] Run `git status` to verify

### Before Every PR
- [ ] All tests pass
- [ ] Security tests pass
- [ ] No new secrets in code
- [ ] Input validation added for new endpoints
- [ ] Authorization checks in place

### Before Every Deployment
- [ ] All environment variables set
- [ ] API keys valid and not expired
- [ ] Database migrations tested
- [ ] Backup created
- [ ] Rollback plan ready
- [ ] Monitoring configured

### Monthly Security Review
- [ ] Review access logs for anomalies
- [ ] Check for outdated dependencies
- [ ] Review error logs in Sentry
- [ ] Test backup restoration
- [ ] Review user permissions
- [ ] Rotate API keys (every 90 days)

---

## 📋 OWASP Top 10 Mitigation

### A01:2021 - Broken Access Control
- ✅ RBAC enforced on all protected routes
- ✅ Default deny policy
- ✅ Audit logging

### A02:2021 - Cryptographic Failures
- ✅ TLS for all connections
- ✅ bcrypt for passwords
- ✅ Secure JWT secrets
- ✅ No sensitive data in URLs

### A03:2021 - Injection
- ✅ Prisma ORM (parameterized queries)
- ✅ Input validation with Zod
- ✅ Output encoding

### A04:2021 - Insecure Design
- ✅ Threat modeling in design phase
- ✅ Security requirements documented
- ✅ Defense in depth

### A05:2021 - Security Misconfiguration
- ✅ Hardened default configuration
- ✅ Security headers configured
- ✅ Error messages sanitized
- ✅ Unused features disabled

### A06:2021 - Vulnerable Components
- ✅ Dependabot enabled
- ✅ Regular dependency audits
- ✅ `npm audit` in CI/CD

### A07:2021 - Authentication Failures
- ✅ MFA ready (future phase)
- ✅ Rate limiting on auth
- ✅ Strong password policy
- ✅ Session management

### A08:2021 - Software/Data Integrity
- ✅ Code review process
- ✅ CI/CD pipeline
- ✅ Dependency lock files
- ✅ Integrity checks

### A09:2021 - Logging/Monitoring Failures
- ✅ Comprehensive logging (Pino)
- ✅ Error tracking (Sentry)
- ✅ Alerting configured
- ✅ No secrets in logs

### A10:2021 - SSRF
- ✅ Whitelist allowed domains
- ✅ Validate URLs before fetching
- ✅ Network segmentation

---

## 🚨 Incident Response

### If Security Issue Discovered

1. **Assess Severity**
   - Critical: Data breach, auth bypass, RCE
   - High: XSS, CSRF, significant data exposure
   - Medium: Minor info disclosure, DoS
   - Low: Configuration issues

2. **Contain**
   - For exposed keys: Revoke immediately
   - For vulnerabilities: Deploy fix ASAP
   - For breaches: Isolate affected systems

3. **Investigate**
   - Check logs for exploitation
   - Identify affected users/data
   - Document timeline

4. **Remediate**
   - Deploy fix
   - Rotate affected credentials
   - Notify affected users (if required)

5. **Learn**
   - Post-mortem analysis
   - Update security measures
   - Document lessons learned

### Contact

**Security Issues:** security@kap-estate.com
**PGP Key:** [To be added]

---

## 📊 Security Testing

### Automated Tests
```bash
# Run security-focused tests
npm run test:security

# Check for known vulnerabilities
npm audit

# Dependency audit
npm audit fix
```

### Manual Testing
- [ ] OWASP ZAP scan
- [ ] Burp Suite testing
- [ ] SQL injection attempts
- [ ] XSS payload testing
- [ ] Authentication bypass attempts
- [ ] Authorization bypass attempts

### Load Testing for Security
```bash
# Test rate limiting
k6 run tests/load/rate-limit.js

# Test auth under load
k6 run tests/load/auth-stress.js
```

---

## 🏆 Security Best Practices for Developers

### Code Review Checklist
- [ ] No secrets in code
- [ ] Input validation present
- [ ] Authorization checks in place
- [ ] Error messages don't leak info
- [ ] Logging doesn't include secrets
- [ ] SQL queries parameterized
- [ ] File uploads restricted
- [ ] Rate limiting applied

### Secure Coding Guidelines
1. **Never trust user input** - Validate everything
2. **Fail securely** - Default deny, explicit allow
3. **Minimize attack surface** - Remove unused code/features
4. **Defense in depth** - Multiple layers of security
5. **Least privilege** - Minimal permissions needed
6. **Keep it simple** - Complex code = more bugs

---

## 📚 Resources

### Internal
- [.env.example](./.env.example) - Environment variable template
- [IMPLEMENTATION_ROADMAP.md](./IMPLEMENTATION_ROADMAP.md) - Security in each phase

### External
- [OWASP Top 10](https://owasp.org/www-project-top-ten/)
- [OWASP Cheat Sheets](https://cheatsheetseries.owasp.org/)
- [Node.js Security Best Practices](https://nodejs.org/en/docs/guides/security/)
- [Next.js Security Headers](https://nextjs.org/docs/app/api-reference/next-config-js/headers)

---

## 🔒 Reporting Vulnerabilities

If you discover a security vulnerability:

1. **DO NOT** open a public issue
2. Email security@kap-estate.com with:
   - Description of vulnerability
   - Steps to reproduce
   - Potential impact
   - Suggested fix (if any)
3. Allow reasonable time for fix (90 days)
4. Disclosure coordination with maintainers

### Bug Bounty
Not currently offering rewards, but we deeply appreciate responsible disclosure.

---

## ✅ Security Sign-Off

**Phase 0 Security Checklist:**
- [x] .gitignore prevents secret commits
- [x] .env.example documents all variables
- [x] No secrets in repository
- [x] Security policy documented
- [ ] Pre-commit hooks configured (pending)
- [ ] Environment validation (pending)

**Future Phases:**
- Phase 1: Authentication security review
- Phase 2: API security testing
- Phase 3: Input validation audit
- Phase 6: Full security audit + penetration testing

---

**Remember: Security is not a feature, it's a requirement.**

**Last Security Review:** November 20, 2025
**Next Review:** December 20, 2025
