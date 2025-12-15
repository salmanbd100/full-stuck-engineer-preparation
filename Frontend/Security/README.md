# Web Security

## Overview

Web security is critical for protecting applications and users from malicious attacks. This module covers essential security concepts including XSS, CSRF, CSP, secure headers, and input validation. Understanding these topics is crucial for frontend interviews at companies handling sensitive user data.

**Key Focus Areas:**
- Cross-Site Scripting (XSS) prevention
- Cross-Site Request Forgery (CSRF) protection
- Content Security Policy (CSP)
- Security headers configuration
- Input validation and sanitization

## Why Security Matters

- **User Trust**: Security breaches destroy user confidence
- **Data Protection**: Prevent unauthorized access to sensitive data
- **Compliance**: GDPR, CCPA, PCI-DSS requirements
- **Business Impact**: Security incidents cost millions
- **Interview Focus**: Top tech companies prioritize security knowledge

## Study Plan

### Beginner Track (1 week)
**Goal**: Understand common web vulnerabilities and basic prevention

**Week 1: Security Fundamentals**
- Day 1-2: XSS attacks and prevention (4 hours)
  - Read 01-xss-prevention.md
  - Practice: Identify XSS vulnerabilities in code
  - Implement DOMPurify in a sample project
- Day 3-4: CSRF protection (3 hours)
  - Read 02-csrf-protection.md
  - Implement CSRF tokens in Express
  - Configure SameSite cookies
- Day 5: Content Security Policy (2 hours)
  - Read 03-csp-headers.md
  - Set up basic CSP headers
- Day 6: Security headers (2 hours)
  - Read 04-secure-headers.md
  - Configure helmet.js
- Day 7: Input validation (2 hours)
  - Read 05-input-sanitization.md
  - Practice with validation libraries

### Intermediate Track (2 weeks)
**Goal**: Implement security best practices in production apps

**Week 1: Offensive Security**
- Days 1-3: XSS deep dive
  - Study all XSS types (Reflected, Stored, DOM-based)
  - Practice exploiting vulnerable apps (DVWA, WebGoat)
  - Implement defense in React and Vue
- Days 4-5: CSRF advanced
  - Double-submit cookies
  - SameSite attribute deep dive
  - Token refresh strategies
- Days 6-7: CSP implementation
  - Nonce-based CSP
  - Hash-based CSP
  - Report-URI configuration

**Week 2: Defensive Security**
- Days 1-2: Security headers suite
  - HSTS, X-Frame-Options, X-Content-Type-Options
  - Permissions-Policy
  - Referrer-Policy
- Days 3-5: Input validation
  - Client and server-side validation
  - Sanitization libraries (DOMPurify, validator.js)
  - File upload security
- Days 6-7: Security testing
  - OWASP ZAP
  - Burp Suite basics
  - Security audits

### Advanced Track (3 weeks)
**Goal**: Master security architecture and advanced attack vectors

**Week 1: Advanced XSS & CSP**
- DOM-based XSS exploitation
- Mutation XSS (mXSS)
- CSP bypasses and mitigations
- Strict CSP implementation
- XSS in modern frameworks (React, Angular, Vue)

**Week 2: Authentication & Authorization**
- JWT security best practices
- OAuth 2.0 flows
- Session management
- Cookie security deep dive
- Token storage strategies

**Week 3: Security Architecture**
- Secure SDLC
- Threat modeling
- Security code review
- Penetration testing
- Bug bounty preparation

## Topics

### 1. XSS Prevention
**File**: [01-xss-prevention.md](./01-xss-prevention.md)

- Reflected XSS
- Stored XSS
- DOM-based XSS
- Output encoding
- DOMPurify library
- React XSS protection
- Content Security Policy
- Interview questions

**Key Concepts**: HTML encoding, JavaScript escaping, attribute encoding, URL encoding

### 2. CSRF Protection
**File**: [02-csrf-protection.md](./02-csrf-protection.md)

- CSRF attack vectors
- Synchronizer tokens
- Double-submit cookies
- SameSite cookie attribute
- Express.js implementation
- React integration
- State-changing operations
- Interview questions

**Key Concepts**: Anti-CSRF tokens, same-origin policy, cookie security

### 3. Content Security Policy
**File**: [03-csp-headers.md](./03-csp-headers.md)

- CSP directives
- script-src, style-src, img-src
- Nonce-based CSP
- Hash-based CSP
- Report-URI
- Strict CSP
- Implementation strategies
- Interview questions

**Key Concepts**: Allowlists, nonces, hashes, CSP violations

### 4. Security Headers
**File**: [04-secure-headers.md](./04-secure-headers.md)

- Strict-Transport-Security (HSTS)
- X-Content-Type-Options
- X-Frame-Options
- X-XSS-Protection
- Referrer-Policy
- Permissions-Policy
- helmet.js configuration
- Interview questions

**Key Concepts**: Defense in depth, browser security features

### 5. Input Sanitization
**File**: [05-input-sanitization.md](./05-input-sanitization.md)

- Client vs server validation
- Validation libraries (Zod, Yup, Joi)
- HTML sanitization (DOMPurify)
- SQL injection prevention
- Command injection prevention
- File upload security
- Regular expression DoS
- Interview questions

**Key Concepts**: Allowlisting, sanitization vs validation, defense in depth

## Prerequisites

- JavaScript fundamentals
- HTTP protocol basics
- Basic understanding of web architecture
- React or another frontend framework (helpful but not required)

## Interview Preparation

### Common Security Interview Topics

1. **XSS Prevention** (Most Common)
   - "How do you prevent XSS attacks?"
   - "What's the difference between reflected and stored XSS?"
   - "How does React prevent XSS?"

2. **CSRF Protection**
   - "Explain CSRF and how to prevent it"
   - "What is the SameSite cookie attribute?"
   - "How do you implement CSRF tokens?"

3. **CSP**
   - "What is Content Security Policy?"
   - "How do you implement CSP in a React app?"
   - "What are CSP nonces and hashes?"

4. **Security Headers**
   - "What security headers should every website have?"
   - "Explain HSTS and its benefits"
   - "What's the purpose of X-Frame-Options?"

5. **Input Validation**
   - "How do you validate user input?"
   - "Client-side vs server-side validation?"
   - "How do you prevent SQL injection?"

### Interview Tips

1. **Demonstrate Defense in Depth**
   - Never rely on a single security measure
   - Show multiple layers of protection
   - Example: Client validation + server validation + sanitization

2. **Explain the Attack First**
   - Describe how the attack works
   - Then explain the defense
   - Shows deeper understanding

3. **Production Experience**
   - Mention security tools you've used (helmet.js, DOMPurify)
   - Describe security incidents you've handled
   - Talk about security testing in CI/CD

4. **Stay Current**
   - OWASP Top 10 awareness
   - Recent CVEs
   - Modern security features (Trusted Types, Sanitizer API)

## Practical Projects

### Project 1: Secure Blog Platform
**Skills**: XSS, CSRF, Input validation

Build a blog with:
- User authentication
- Comment system (prevent stored XSS)
- CSRF protection on all forms
- Input validation and sanitization
- Security headers

### Project 2: Security Audit Tool
**Skills**: CSP, Security headers, Testing

Create a tool that:
- Analyzes security headers
- Checks CSP configuration
- Scans for XSS vulnerabilities
- Generates security report

### Project 3: Secure File Upload
**Skills**: File validation, Content-Type checking

Implement:
- File type validation
- Size limits
- Virus scanning integration
- Secure storage
- Content-Type verification

## Resources

### Official Documentation
- [OWASP Top 10](https://owasp.org/www-project-top-ten/)
- [MDN Web Security](https://developer.mozilla.org/en-US/docs/Web/Security)
- [Content Security Policy](https://content-security-policy.com/)
- [helmet.js Documentation](https://helmetjs.github.io/)

### Tools
- [OWASP ZAP](https://www.zaproxy.org/) - Security scanner
- [Burp Suite](https://portswigger.net/burp) - Penetration testing
- [DOMPurify](https://github.com/cure53/DOMPurify) - XSS sanitizer
- [SecurityHeaders.com](https://securityheaders.com/) - Header analyzer

### Learning Platforms
- [PortSwigger Web Security Academy](https://portswigger.net/web-security) - Free labs
- [OWASP WebGoat](https://owasp.org/www-project-webgoat/) - Vulnerable app for practice
- [HackerOne](https://www.hackerone.com/) - Bug bounty platform
- [CTF Time](https://ctftime.org/) - Capture the Flag competitions

### Books
- "The Web Application Hacker's Handbook" by Dafydd Stuttard
- "OWASP Testing Guide" (Free)
- "Tangled Web" by Michal Zalewski

## Security Checklist

Before deploying any web application:

### Headers
- [ ] Content-Security-Policy configured
- [ ] Strict-Transport-Security enabled (HTTPS only)
- [ ] X-Content-Type-Options: nosniff
- [ ] X-Frame-Options: DENY or SAMEORIGIN
- [ ] Referrer-Policy configured
- [ ] Permissions-Policy configured

### Authentication
- [ ] HTTPS enforced everywhere
- [ ] Secure cookie flags (Secure, HttpOnly, SameSite)
- [ ] Password requirements enforced
- [ ] Rate limiting on login
- [ ] CSRF protection on state-changing operations

### Input Handling
- [ ] Server-side validation on all inputs
- [ ] HTML sanitization for user content
- [ ] SQL parameterized queries
- [ ] File upload restrictions
- [ ] Output encoding

### Monitoring
- [ ] CSP violation reporting
- [ ] Error logging (without exposing sensitive data)
- [ ] Security incident response plan
- [ ] Regular security audits

## Progress Tracking

Track your progress as you complete each topic:

- [ ] XSS Prevention - Read and practice
- [ ] CSRF Protection - Implement in a project
- [ ] CSP Headers - Configure and test
- [ ] Security Headers - Set up helmet.js
- [ ] Input Sanitization - Practice validation libraries

## Next Steps

After completing this module:

1. **Practice**: Build secure applications
2. **Test**: Use OWASP ZAP to scan your projects
3. **Learn**: Explore advanced topics (JWT, OAuth)
4. **Contribute**: Participate in bug bounties
5. **Stay Updated**: Follow security researchers and blogs

## Related Topics

- **Backend Security**: SQL injection, authentication, authorization
- **DevOps Security**: Secrets management, container security
- **Network Security**: HTTPS, TLS, certificates
- **Testing**: Security testing, penetration testing, fuzzing

---

**Time to Complete**: 1-3 weeks depending on track
**Difficulty**: Intermediate to Advanced
**Interview Frequency**: Very High (Security questions in 70%+ of interviews)

Start with [01-xss-prevention.md](./01-xss-prevention.md) ’
