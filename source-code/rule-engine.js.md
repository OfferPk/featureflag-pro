/\*\*  
 \* FeatureFlag Pro \- Rule Evaluation & Targeting Engine  
 \*/

(function(root, factory) {  
  if (typeof module \=== 'object' && module.exports) {  
    const Murmur3 \= require('./murmur3.js');  
    module.exports \= factory(Murmur3);  
  } else {  
    root.RuleEngine \= factory(root.Murmur3);  
  }  
}(typeof self \!== 'undefined' ? self : this, function(Murmur3) {

  function evaluateOperator(userValue, operator, ruleValue) {  
    if (userValue \=== undefined || userValue \=== null) {  
      return operator \=== 'not\_equals' || operator \=== 'not\_in\_list';  
    }

    const uStr \= String(userValue).trim().toLowerCase();  
    const rStr \= String(ruleValue).trim().toLowerCase();

    switch (operator) {  
      case 'equals':  
        return uStr \=== rStr;  
      case 'not\_equals':  
        return uStr \!== rStr;  
      case 'contains':  
        return uStr.includes(rStr);  
      case 'not\_contains':  
        return \!uStr.includes(rStr);  
      case 'in\_list': {  
        const items \= rStr.split(',').map(s \=\> s.trim());  
        return items.includes(uStr);  
      }  
      case 'not\_in\_list': {  
        const items \= rStr.split(',').map(s \=\> s.trim());  
        return \!items.includes(uStr);  
      }  
      case 'greater\_than':  
        return Number(userValue) \> Number(ruleValue);  
      case 'less\_than':  
        return Number(userValue) \< Number(ruleValue);  
      case 'regex':  
        try {  
          const re \= new RegExp(ruleValue, 'i');  
          return re.test(String(userValue));  
        } catch (e) {  
          return false;  
        }  
      default:  
        return false;  
    }  
  }

  function resolveFlag(flagConfig, userContext \= {}) {  
    const userId \= String(userContext.id || userContext.userId || 'anonymous');  
    const salt \= flagConfig.salt || flagConfig.id || 'default\_salt';  
    const bucket \= Murmur3.getBucket(userId, salt);

    // Global disable check  
    if (flagConfig.enabled \=== false) {  
      return {  
        flagId: flagConfig.id,  
        enabled: false,  
        variant: flagConfig.defaultVariant \!== undefined ? flagConfig.defaultVariant : false,  
        bucket,  
        matchedRule: null,  
        reason: 'FLAG\_DISABLED\_GLOBALLY'  
      };  
    }

    // 1\. Evaluate explicit targeting rules  
    const rules \= flagConfig.rules || \[\];  
    for (const rule of rules) {  
      const userVal \= userContext\[rule.attribute\];  
      if (evaluateOperator(userVal, rule.operator, rule.value)) {  
        // Matched rule action  
        if (rule.action \=== 'force\_true') {  
          return {  
            flagId: flagConfig.id,  
            enabled: true,  
            variant: true,  
            bucket,  
            matchedRule: rule,  
            reason: \`RULE\_MATCHED: \${rule.id || rule.description}\`  
          };  
        } else if (rule.action \=== 'force\_false') {  
          return {  
            flagId: flagConfig.id,  
            enabled: false,  
            variant: false,  
            bucket,  
            matchedRule: rule,  
            reason: \`RULE\_MATCHED: \${rule.id || rule.description}\`  
          };  
        } else if (rule.action \=== 'force\_variant') {  
          return {  
            flagId: flagConfig.id,  
            enabled: true,  
            variant: rule.variant,  
            bucket,  
            matchedRule: rule,  
            reason: \`RULE\_MATCHED: \${rule.id || rule.description}\`  
          };  
        } else if (rule.action \=== 'percentage') {  
          const ruleRollout \= rule.rolloutPercentage \!== undefined ? rule.rolloutPercentage : 50;  
          const isEligible \= bucket \< ruleRollout;  
          return {  
            flagId: flagConfig.id,  
            enabled: isEligible,  
            variant: isEligible ? true : false,  
            bucket,  
            matchedRule: rule,  
            reason: \`RULE\_PERCENTAGE: Bucket \${bucket} \< \${ruleRollout}%\`  
          };  
        }  
      }  
    }

    // 2\. Default Type Evaluation  
    const type \= flagConfig.type || 'percentage';

    if (type \=== 'boolean') {  
      return {  
        flagId: flagConfig.id,  
        enabled: Boolean(flagConfig.enabled),  
        variant: Boolean(flagConfig.enabled),  
        bucket,  
        matchedRule: null,  
        reason: 'DEFAULT\_BOOLEAN'  
      };  
    }

    if (type \=== 'percentage') {  
      const rollout \= flagConfig.rolloutPercentage \!== undefined ? flagConfig.rolloutPercentage : 0;  
      const isIncluded \= bucket \< rollout;  
      return {  
        flagId: flagConfig.id,  
        enabled: isIncluded,  
        variant: isIncluded ? true : false,  
        bucket,  
        matchedRule: null,  
        reason: \`DEFAULT\_PERCENTAGE: Bucket \${bucket} \< \${rollout}%\`  
      };  
    }

    if (type \=== 'multivariant') {  
      const variants \= flagConfig.variants || \[\];  
      let cumulative \= 0;  
      let selectedVariant \= flagConfig.defaultVariant || (variants\[0\] && variants\[0\].name) || 'control';

      for (const v of variants) {  
        cumulative \+= v.percentage;  
        if (bucket \< cumulative) {  
          selectedVariant \= v.name;  
          break;  
        }  
      }

      return {  
        flagId: flagConfig.id,  
        enabled: true,  
        variant: selectedVariant,  
        bucket,  
        matchedRule: null,  
        reason: \`MULTIVARIANT\_BUCKET: Bucket \${bucket} mapped to \${selectedVariant}\`  
      };  
    }

    return {  
      flagId: flagConfig.id,  
      enabled: false,  
      variant: false,  
      bucket,  
      matchedRule: null,  
      reason: 'UNKNOWN\_TYPE\_FALLBACK'  
    };  
  }

  return {  
    evaluateOperator,  
    resolveFlag  
  };  
}));  
