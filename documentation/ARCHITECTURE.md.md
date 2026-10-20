# FeatureFlag Pro — System Architecture & Deterministic Bucketing

## 1\. Architectural Philosophy

FeatureFlag Pro is designed as a **zero-latency, zero-infrastructure feature flag and rollout studio**.

Traditional commercial feature flag solutions (LaunchDarkly, Split.io, Statsig) suffer from three core operational drawbacks:

1. **Network Latency & Edge Hops**: Client SDKs must poll or stream from remote SaaS endpoints, adding 50–200ms latency to page loads or blocking SSR hydration.  
2. **Third-Party Availability Risk**: Outages at the SaaS vendor freeze feature updates or cause cascading application failures.  
3. **Usage-Based Cost Inflation**: Enterprise pricing plans scale aggressively based on monthly active users (MAUs), frequently exceeding \$500–\$2,000/month.

FeatureFlag Pro decouples feature evaluation from remote servers by implementing a **deterministic 32-bit MurmurHash3 bucketing engine**. Flag manifests are bundled statically or served via CDN/edge key-value stores, and evaluations occur locally in sub-millisecond time.

---

## 2\. Deterministic Hash Bucketing Mechanics

### 2.1 The MurmurHash3 x86 Algorithm

To ensure that an identical user is deterministically mapped to the exact same percentage bucket \$\[0, 99\]\$ across multiple devices, browsers, and server processes, FeatureFlag Pro utilizes 32-bit MurmurHash3: \$\$\\text{bucket} \= \\text{MurmurHash3}(\\text{userId} \+ \\text{":"} \+ \\text{flagSalt}) \\pmod{100}\$\$

* **Avalanche Property**: Changing a single character in the `userId` or `flagSalt` alters more than 50% of the output bits, preventing clustering.  
* **Uniformity**: Hashes are uniformly distributed across the integer space \$\[0, 2^{32}-1\]\$, ensuring that a 25% rollout allocates almost exactly 25.0% across any sufficiently large sample of users.  
* **Salt Isolation**: Providing a unique salt per flag ensures that user bucketing is statistically independent across different feature experiments (a user in the 10% bucket for Feature A will not automatically be in the 10% bucket for Feature B).

---

## 3\. Evaluation Pipeline & Precedence Hierarchy

\[User Context: { id, tier, country, email }\]

                    │

                    ▼

       \[Global Enabled Check\] ──(False)──\> Return Default Inactive

                    │

                 (True)

                    ▼

       \[Targeting Rules Loop\]

    ├── Rule 1: Match Attribute Operator? ──(Yes)──\> Execute Rule Action (force\_true / force\_variant)

    ├── Rule 2: Match Attribute Operator? ──(Yes)──\> Execute Rule Action

    └── ...

                    │

                 (No Match)

                    ▼

     \[Default Rollout Bucketing\]

    ├── Calculate MurmurHash3(userId:salt) % 100

    ├── Percentage: Bucket \< Rollout % ? True : False

    ├── Multivariant: Cumulative Variant Slice Match

    └── Boolean: Return Flag.Enabled

---

## 4\. Supported Framework SDKs

FeatureFlag Pro provides zero-dependency code generators for:

1. **React / Next.js**: Custom `useFeatureFlag` hook with `localStorage` override support for QA testing and SSR hydration-safe fallbacks.  
2. **Node.js Express**: Non-blocking request context middleware with header inspection (`x-user-id`).  
3. **Python FastAPI / Flask**: Pure Python hashlib/MD5 fallback evaluator executing in under 0.05ms per request.  
4. **Golang**: Concurrent-safe struct and native bitwise Murmur3 implementation for microservices.