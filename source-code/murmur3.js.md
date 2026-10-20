/\*\*  
 \* FeatureFlag Pro \- 32-bit MurmurHash3 Engine  
 \* Computes deterministic, avalanche-tested 32-bit hashes for uniform user bucketing.  
 \* Compatible with standard MurmurHash3 x86 32-bit implementations in Python, Go, and C.  
 \*/

(function(root, factory) {  
  if (typeof module \=== 'object' && module.exports) {  
    module.exports \= factory();  
  } else {  
    root.Murmur3 \= factory();  
  }  
}(typeof self \!== 'undefined' ? self : this, function() {

  function hash32(key, seed \= 0\) {  
    let remainder, bytes, h1, k1, i, c1, c2, p1, p2;

    remainder \= key.length & 3; // key.length % 4  
    bytes \= key.length \- remainder;  
    h1 \= seed;

    c1 \= 0xcc9e2d51;  
    c2 \= 0x1b873593;

    i \= 0;  
    while (i \< bytes) {  
      k1 \=  
        ((key.charCodeAt(i) & 0xff)) |  
        ((key.charCodeAt(++i) & 0xff) \<\< 8\) |  
        ((key.charCodeAt(++i) & 0xff) \<\< 16\) |  
        ((key.charCodeAt(++i) & 0xff) \<\< 24);  
      \++i;

      k1 \= Math.imul(k1, c1);  
      k1 \= (k1 \<\< 15\) | (k1 \>\>\> 17);  
      k1 \= Math.imul(k1, c2);

      h1 ^= k1;  
      h1 \= (h1 \<\< 13\) | (h1 \>\>\> 19);  
      h1 \= Math.imul(h1, 5\) \+ 0xe6546b64;  
    }

    k1 \= 0;  
    switch (remainder) {  
      case 3:  
        k1 ^= (key.charCodeAt(i \+ 2\) & 0xff) \<\< 16;  
      case 2:  
        k1 ^= (key.charCodeAt(i \+ 1\) & 0xff) \<\< 8;  
      case 1:  
        k1 ^= (key.charCodeAt(i) & 0xff);  
        k1 \= Math.imul(k1, c1);  
        k1 \= (k1 \<\< 15\) | (k1 \>\>\> 17);  
        k1 \= Math.imul(k1, c2);  
        h1 ^= k1;  
    }

    h1 ^= key.length;

    h1 ^= h1 \>\>\> 16;  
    h1 \= Math.imul(h1, 0x85ebca6b);  
    h1 ^= h1 \>\>\> 13;  
    h1 \= Math.imul(h1, 0xc2b2ae35);  
    h1 ^= h1 \>\>\> 16;

    return h1 \>\>\> 0; // Return unsigned 32-bit integer  
  }

  /\*\*  
   \* Deterministically maps a userId and flagSalt to a bucket between 0 and 99 (inclusive).  
   \*/  
  function getBucket(userId, salt \= '', seed \= 0\) {  
    const compositeKey \= \`\${userId}:\${salt}\`;  
    const hashVal \= hash32(compositeKey, seed);  
    return hashVal % 100;  
  }

  return {  
    hash32,  
    getBucket  
  };  
}));  
