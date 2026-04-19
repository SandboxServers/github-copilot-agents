## B2B vs B2C

### B2B (Business-to-Business)
| Aspect | Details |
|--------|---------|
| **Purpose** | External partners, vendors, contractors accessing YOUR resources |
| **Identity source** | Their own org's identity (Entra ID, Google, SAML/WS-Fed IdP, email OTP) |
| **Guest user** | Represented as guest in your directory (UserType = Guest) |
| **Licensing** | Your tenant needs Entra ID P1/P2 for conditional access on guests |
| **Governance** | Entra ID access reviews, entitlement management, access packages |
| **Common issues** | Over-permissioned guests, stale guest accounts, no access reviews |

### B2C (Business-to-Consumer) / External ID
| Aspect | Details |
|--------|---------|
| **Purpose** | Customer-facing apps with self-service sign-up |
| **Identity source** | Local accounts, social IdPs (Google, Facebook, Apple), SAML/OIDC |
| **Directory** | Separate B2C tenant (or External ID in workforce tenant) |
| **Customization** | Custom user flows, custom policies (Identity Experience Framework) |
| **Scale** | Millions of users, consumer-grade UX |
| **Common issues** | Custom policy XML complexity, branding consistency, API connector reliability |

### Decision Tree
```
External user accessing YOUR corporate resources? → B2B
  └── They have their own org identity? → B2B collaboration (guest)
  └── They need self-service sign-up to your customer app? → External ID or B2C

Customer-facing app with self-service registration? → B2C or External ID
  └── Need social login (Google, Facebook)? → B2C / External ID
  └── Need deep customization of auth flows? → B2C custom policies
  └── Simple flows, modern stack? → External ID (recommended going forward)
```
