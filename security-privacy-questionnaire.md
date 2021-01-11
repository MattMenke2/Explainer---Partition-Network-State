### 2.1. What information might this feature expose to Web sites or other parties, and for what purposes is that exposure necessary?

While the entire purpose of this feature is to expose less cross-site information to web sites, it does, however, potentially make some types of information easier to probe. Global limits, for instance, become easier to reach with fewer origins, which can leak information across sites, or could be used to evict information stored on behalf of other sites.

That having been said, on balance, it makes it more difficult for a user to be tracked across sites, and for sites to spy on each other.

### 2.2. Is this specification exposing the minimum amount of information necessary to power the feature?
Regardless of what data is being exposed, is the specification exposing the bare minimum necessary to achieve the desired use cases? If not, why not and why expose the additional information?

This feature doesn't directly expose any new information to sites - it hides information.

### 2.3. How does this specification deal with personal information or personally-identifiable information or information derived thereof?
### 2.4. How does this specification deal with sensitive information?

The specification doesn't directly deal with any PII. It does shard network connections by top-level site.  Since reused network objects could be used to indirectly learn about what other sites a user visits (e.g., through timing attacks), this specification should improve user privacy.

### 2.5. Does this specification introduce new state for an origin that persists across browsing sessions?
No, though it does make state that was originally per origin now be per-top-level-site-per-origin.

### 2.6. What information from the underlying platform, e.g. configuration data, is exposed by this specification to an origin?
No new configuration information is exposed.

### 2.7. Does this specification allow an origin access to sensors on a user’s device
No.

### 2.8. What data does this specification expose to an origin? Please also document what data is identical to data exposed by other features, in the same or different contexts.
This exposes no new data to an origin, though it does shard existing data by top-level site to prevent users being tracked based on per-origin data.

### 2.9. Does this specification enable new script execution/loading mechanisms?
No.

### 2.10. Does this specification allow an origin to access other devices?
No.

### 2.11. Does this specification allow an origin some measure of control over a user agent’s native UI?
No.

### 2.12. What temporary identifiers might this this specification create or expose to the web?
This specification attempts to further shard network state, much of which can implicitly be used as user identifiers (reused connection, alternative service information, DNS cache entries, etc).

### 2.13. How does this specification distinguish between behavior in first-party and third-party contexts?
It keys network state on first-party site. It currently leaves open whether network state used in third-party context network state will additionally be keyed on third party site or not. Otherwise, it doesn't distinguish between the two.

### 2.14. How does this specification work in the context of a user agent’s Private Browsing or "incognito" mode?
It works much the same as it does in standard browsing mode.

### 2.15. Does this specification have a "Security Considerations" and "Privacy Considerations" section?
No. The spec was added directly to the Fetch spec. The fetch spec, for better or worse, is specified as a set of prescriptive rules without any comments, reasoning, or motivations, so such a section would seem out of place there, given the current style. That aside, partitioning network state should be a net privacy and security positive, better isolating top-level browsing contexts from each other, which providing no new avenues for attacks.



Documenting the various concerns and potential abuses in "Security Considerations" and "Privacy Considerations" sections of a document is a good way to help implementers and web developers understand the risks that a feature presents, and to ensure that adequate mitigations are in place. Simply adding a section to your specification with yes/no responses to the questions in this document is insufficient.

If it seems like a feature does not have security or privacy impacts, then say so inline in the spec section for that feature:

There are no known security or privacy impacts of this feature.

Saying so explicitly in the specification serves several purposes:

Shows that a spec author/editor has explicitly considered security and privacy when designing a feature.

Provides some sense of confidence that there might be no such impacts.

Challenges security and privacy minded individuals to think of and find even the potential for such impacts.

Demonstrates the spec author/editor’s receptivity to feedback about such impacts.

Demonstrates a desire that the specification should not be introducing security and privacy issues

[RFC3552] provides general advice as to writing Security Consideration sections. Generally, there should be a clear description of the kinds of privacy risks the new specification introduces to for users of the web platform. Below is a set of considerations, informed by that RFC, for writing a privacy considerations section.

Authors must describe:

What privacy attacks have been considered?

What privacy attacks have been deemed out of scope (and why)?

What privacy mitigations have been implemented?

What privacy mitigations have considered and not implemented (and why)?

In addition, attacks considered must include:

Fingerprinting risk;

Unexpected exfiltration of data through abuse of sensors;

Unexpected usage of the specification / feature by third parties;

If the specification includes identifiers, the authors must document what rotation period was selected for the identifiers and why.

If the specification introduces new state to the user agent, the authors must document what guidance regarding clearing said storage was given and why.

There should be a clear description of the residual risk to the user after the privacy mitigations has been implemented.

The crucial aspect is to actually considering security and privacy. All new specifications must have security and privacy considerations sections to be considered for wide reviews. Interesting features added to the web platform generally often already had security and/or privacy impacts.

2.16. Does this specification allow downgrading default security characteristics?
No.

2.17. What should this questionnaire have asked?
N/A
