### The Security–Usability Balance

The lead's hardest job is balancing security with usability:

| Too Secure | Too Permissive | The Balance |
|-----------|---------------|-------------|
| Block all external sharing | Allow anonymous sharing links | External sharing with approval workflow and expiring links |
| Require managed device for everything | Allow any device unrestricted | App protection policies for unmanaged, full access for compliant devices |
| Block all Power Platform connectors | Allow all connectors | DLP policies with business/non-business/blocked groups |
| Disable guest access entirely | Allow guests with no restrictions | Guest access with Conditional Access, time-limited, with access reviews |
| Require MFA on every single action | No MFA | Risk-based MFA — challenge when risk is elevated, not on every click |

The principle: **Make the secure way the easy way.** If the secure path has more friction than the insecure workaround, users will find the workaround.
