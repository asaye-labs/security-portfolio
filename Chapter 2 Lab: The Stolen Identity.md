# [The Stolen Identity, "A walk through the kill-chain of a confused deputy attack."]

## Scenario
Reconstructed a five-stage OAuth consent-phishing kill chain in a live Azure tenant through forensic analysis of two linked app registrations.

## Environment
Live multi-user Azure training tenant, Microsoft Azure portal, App Registrations, Branding and Properties, Authentication (Preview), Certificates & secrets, API Permissions, Expose an API blades, Reader access

## Investigation
1. ENTRY. A user was phished, completed MFA, and had the resulting session token stolen. That token carried an MFA-satisfied claim, so it sailed past Conditional Access. That user was also, through years of drift, still an Owner on a legacy connector app.

![[Chapter 2 objective 1 found the flag in the note of the legacy app.jpg]]

2. ESCALATE. Using those Owner rights, the attacker minted a new client secret on the legacy app. That secret let them authenticate through the client credentials flow as the service principal itself, inheriting the app's directory permissions without ever signing in as a human again. Note the expiry date: set nearly a century out.

![[Chapter 2 objective 2 there was a flag in the certificates and secrets blade that had a VERY suspicious expiration.jpg]]

3. PIVOT. A single secret dies when it gets rotated. So the attacker registered their own app (every standard user can do this by default in Entra) and added its service principal to the legacy app's Owners list. Now they can re-credential the legacy app forever, even after the first secret is caught.

![[Chapter 2 objective 3 pivoted and found another flag within the Mad-Hat-Labs-App.jpg]]

4. PERSIST. Then the backup plan: a custom scope published on the legacy app's Expose an API blade. This turns the legacy app into a callable backend resource, which means the attacker's own app can request delegated access to it.
   
![[Chapter 2 objective 4 the attacker left a hidden flag to leave a persistence trail including another flag when looking at the expose an API blade.jpg]]

4. LOOT. Finally, a redirect URI on the rogue app pointing at attacker-controlled infrastructure. Combining the rogue app's client ID, that redirect URI, and the exposed API scope produces a working phishing URL. A victim who is already signed in on a corporate device clicks Accept on a consent prompt, and the authorization code lands on the attacker's server.
   
![[Chapter 2 objective 5 when looking at the authentication blade of the Mad-Hat-Labs-App i saw the attacker setup a redirected URI.jpg]]

## What broke / what surprised me
The OAuth consent-phishing kill chain here showed me that even with standard defensive measures in place to secure user accounts with standard measures such as Passkeys or MFA there is a way to not only bypass them but use the compromised user account's access to a legacy app with read all privileges to continously cause even more harm.

## Findings and recommendations
It is absolutely crucial to maintain an active scan of any security vulnerabilities that can help with confused deputy attacks such as this. The identiy plane was completely bypassed by a legacy app still having escalated privileges beyond the compromised user account and this all could have been avoided with a proper audit of legacy or unused apps.

## What I learned
- User accounts and databases are not the only thing that can become compromised, apps themselves can be used as attack vectors.

- How to navigate through different Registered Apps and their blades within Azure to follow clues that revealed a much larger security issue.
  
- Maintaining proper audits of legacy or unused apps helps maintain a secure environment.
