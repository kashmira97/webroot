Simple common functions reside in localsite/locasite.js

let hash = getHash();

goHash({"key":value}); // Invokes hashChangeEvent

updateHash({"key":value}); // Avoids invoking hashChangeEvent

// Watches for hash changes at the top of .js pages:
document.addEventListener('hashChangeEvent', function (elem) {
}, false);