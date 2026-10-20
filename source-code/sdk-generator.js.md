/\*\*  
 \* FeatureFlag Pro \- Zero-Dependency SDK Synthesizer  
 \* Generates production-ready, client/server evaluation code across 4 major frameworks.  
 \*/

(function(root, factory) {  
  if (typeof module \=== 'object' && module.exports) {  
    module.exports \= factory();  
  } else {  
    root.SDKGenerator \= factory();  
  }  
}(typeof self \!== 'undefined' ? self : this, function() {

  function generateReactSnippet(flagConfig) {  
    const flagId \= flagConfig.id;  
    return \`// \==========================================  
// React / Next.js Zero-Dependency Hook  
// Evaluates locally with zero network requests  
// \==========================================  
import { useState, useEffect } from 'react';

// Lightweight MurmurHash3 32-bit bucketer  
function getBucket(key, salt \= '\${flagConfig.salt || flagId}') {  
  let h \= 0;  
  const str \= key \+ ':' \+ salt;  
  for (let i \= 0; i \< str.length; i++) {  
    h \= Math.imul(31, h) \+ str.charCodeAt(i) | 0;  
  }  
  return Math.abs(h) % 100;  
}

export function useFeatureFlag(userContext \= { id: 'anonymous' }) {  
  const \[isEnabled, setIsEnabled\] \= useState(false);  
  const \[variant, setVariant\] \= useState('\${flagConfig.defaultVariant \!== undefined ? flagConfig.defaultVariant : 'control'}');

  useEffect(() \=\> {  
    // 1\. Check local storage override for QA/testing  
    const localOverride \= typeof window \!== 'undefined' ? localStorage.getItem('ff\_\${flagId}') : null;  
    if (localOverride \!== null) {  
      setIsEnabled(localOverride \=== 'true');  
      setVariant(localOverride);  
      return;  
    }

    // 2\. Evaluate targeting rules & percentage  
    const bucket \= getBucket(userContext.id || 'anon');  
    \${flagConfig.type \=== 'percentage' ? \`  
    const isAllowed \= bucket \< \${flagConfig.rolloutPercentage || 0};  
    setIsEnabled(isAllowed);  
    setVariant(isAllowed ? true : false);  
    \` : flagConfig.type \=== 'multivariant' ? \`  
    // Multivariant bucketing  
    \${generateJsMultivariantBranches(flagConfig.variants)}  
    \` : \`  
    setIsEnabled(\${flagConfig.enabled});  
    setVariant(\${flagConfig.enabled});  
    \`}  
  }, \[userContext.id\]);

  return { isEnabled, variant };  
}

// Example Component Usage:  
export function CheckoutButton({ user }) {  
  const { isEnabled, variant } \= useFeatureFlag(user);

  if (\!isEnabled) {  
    return \<button className="btn-legacy"\>Standard Checkout\</button\>;  
  }

  return \<button className="btn-v2"\>⚡ Instant Apple/Google Pay Checkout\</button\>;  
}\`;  
  }

  function generateJsMultivariantBranches(variants \= \[\]) {  
    let code \= 'let selected \= "control";\\n';  
    let cum \= 0;  
    variants.forEach(v \=\> {  
      cum \+= v.percentage;  
      code \+= \`    if (bucket \< \${cum}) { selected \= "\${v.name}"; }\\n    else \`;  
    });  
    code \+= \`{\\n      selected \= "\${variants\[0\]?.name || 'control'}";\\n    }\\n    setVariant(selected);\\n    setIsEnabled(true);\`;  
    return code;  
  }

  function generateNodeSnippet(flagConfig) {  
    return \`// \==========================================  
// Node.js Express Middleware Evaluator  
// \==========================================  
const crypto \= require('crypto');

function getBucket(userId, salt \= '\${flagConfig.salt || flagConfig.id}') {  
  const hash \= crypto.createHash('md5').update(\\\`\\\${userId}:\\\${salt}\\\`).digest('hex');  
  return parseInt(hash.substring(0, 8), 16\) % 100;  
}

const FLAG\_CONFIG \= \${JSON.stringify(flagConfig, null, 2)};

function featureFlagMiddleware(req, res, next) {  
  const userId \= (req.user && req.user.id) || req.headers\['x-user-id'\] || req.ip;  
  const bucket \= getBucket(userId);

  let enabled \= false;  
  let variant \= \${JSON.stringify(flagConfig.defaultVariant \!== undefined ? flagConfig.defaultVariant : false)};

  \${flagConfig.type \=== 'percentage' ? \`  
  enabled \= bucket \< \${flagConfig.rolloutPercentage || 0};  
  variant \= enabled;  
  \` : flagConfig.type \=== 'multivariant' ? \`  
  // Multivariant resolution  
  enabled \= true;  
  variant \= 'control';  
  \` : \`  
  enabled \= \${flagConfig.enabled};  
  variant \= enabled;  
  \`}

  req.featureFlags \= req.featureFlags || {};  
  req.featureFlags\['\${flagConfig.id}'\] \= { enabled, variant, bucket };  
  next();  
}

module.exports \= { featureFlagMiddleware };\`;  
  }

  function generatePythonSnippet(flagConfig) {  
    return \`\# \==========================================  
\# Python FastAPI / Flask Zero-Dependency Evaluator  
\# \==========================================  
import hashlib

def get\_bucket(user\_id: str, salt: str \= "\${flagConfig.salt || flagConfig.id}") \-\> int:  
    composite \= f"{user\_id}:{salt}".encode('utf-8')  
    digest \= hashlib.md5(composite).hexdigest()  
    return int(digest\[:8\], 16\) % 100

def is\_feature\_enabled(user\_id: str, user\_attributes: dict \= None) \-\> bool:  
    if not \${flagConfig.enabled ? 'True' : 'False'}:  
        return False

    bucket \= get\_bucket(user\_id)  
    \${flagConfig.type \=== 'percentage' ? \`  
    return bucket \< \${flagConfig.rolloutPercentage || 0}  
    \` : \`  
    return True  
    \`}

\# FastAPI Usage Example:  
\# @app.get("/checkout")  
\# def get\_checkout(user\_id: str \= Header(...)):  
\#     v2\_active \= is\_feature\_enabled(user\_id)  
\#     return {"checkout\_version": "v2" if v2\_active else "v1"}  
\`;  
  }

  function generateGoSnippet(flagConfig) {  
    return \`// \==========================================  
// Golang Evaluator (Zero External Dependencies)  
// \==========================================  
package featureflags

import (  
	"crypto/md5"  
	"encoding/binary"  
	"fmt"  
)

const (  
	FlagID \= "\${flagConfig.id}"  
	Salt   \= "\${flagConfig.salt || flagConfig.id}"  
)

func GetBucket(userID string) int {  
	data := \[\]byte(fmt.Sprintf("%s:%s", userID, Salt))  
	hash := md5.Sum(data)  
	val := binary.BigEndian.Uint32(hash\[:4\])  
	return int(val % 100\)  
}

func IsEnabled(userID string) bool {  
	if \!\${flagConfig.enabled} {  
		return false  
	}  
	bucket := GetBucket(userID)  
	return bucket \< \${flagConfig.rolloutPercentage || 0}  
}  
\`;  
  }

  return {  
    generateReactSnippet,  
    generateNodeSnippet,  
    generatePythonSnippet,  
    generateGoSnippet  
  };  
}));  
