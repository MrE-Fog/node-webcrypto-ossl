# node-webcrypto-ossl

[![License](https://img.shields.io/badge/license-MIT-green.svg?style=flat)](https://raw.githubusercontent.com/PeculiarVentures/node-webcrypto-ossl/master/LICENSE)
![test](https://github.com/PeculiarVentures/node-webcrypto-ossl/workflows/test/badge.svg)
[![Coverage Status](https://coveralls.io/repos/github/PeculiarVentures/node-webcrypto-ossl/badge.svg?branch=master)](https://coveralls.io/github/PeculiarVentures/node-webcrypto-ossl?branch=master)
[![npm version](https://badge.fury.io/js/node-webcrypto-ossl.svg)](https://badge.fury.io/js/node-webcrypto-ossl)

[![NPM](https://nodei.co/npm/node-webcrypto-ossl.png)](https://nodei.co/npm/node-webcrypto-ossl/)

## Deprecated

__`node-webcrypto-ossl` was created in 2015 because at the time the Node team did not feel the need to have two crypto interfaces and they already had one before WebCrypto was defined.__

__That position has since changed since Node16.x (2021-10-26) there is a LTS build of Node that supports the WebCrypto interface.__

__We have also since created [@peculiar/webcrypto](https://www.npmjs.com/package/@peculiar/webcrypto) which no longer has the direct dependency on OpenSSL which makes it much easier to support in the wild. This package supports a number of algorithms that are not supported by standards compliant webcrypto implementations.__

__This allows applications to retain maximal code compatibility with standards while maintaining interoperability with systems that rely on these other algorithms. For those who have this requirement we recommend moving to [@peculiar/webcrypto](https://www.npmjs.com/package/@peculiar/webcrypto).__

## About

We wanted to be able to write Javascript that used crypto on both the client and the server but we did not want to rely on Javascript implementations of crypto. The only native cryptography available in browser is [Web Crypto](http://caniuse.com/#search=cryptography), this resulted in us creating a `node-webcrypto-ossl` a native polyfill for WebCrypto based on OpenSSL since at the time Node did not have a native WebCrypto implementation. As of [Node 15](https://nodejs.org/api/webcrypto.html) however they have added such a interface so you may not need this module any longer.

## Table Of Contents

* [WARNING](#warning)
* [Installing](#installing)
  * [Clone Repo](#clone-repo)
  * [Install Dependencies](#install-dependencies)
  * [Install](#install)
  * [Test](#test)
* [Threat Model](#threat-model)
  * [Assumptions](#assumptions)
  * [Threats From Weak Cryptography](#threats-from-weak-cryptography)
  * [Threats From Improper Use Of Cryptography](#threats-from-improper-use-of-cryptography)
* [Bug Reporting](#bug-reporting)
* [Related](#related)

## WARNING

**At this time this solution should be considered suitable for research and experimentation, further code and security review is needed before utilization in a production application.**

## Installation

### npm

```
npm install node-webcrypto-ossl
```

### Clone Repo

```
git clone https://github.com/PeculiarVentures/node-webcrypto-ossl
cd node-webcrypto-ossl
```

### Install 

```                          
npm install
```

### Test

```
mocha
```

## Supported algorithms

| Algorithm name    | generateKey | digest  | export/import | sign/verify | encrypt/decrypt | wrapKey/unwrapKey | derive  |
|-------------------|-------------|---------|---------------|-------------|-----------------|-------------------|---------|
| SHA-1             |             |    X    |               |             |                 |                   |         |
| SHA-256           |             |    X    |               |             |                 |                   |         |
| SHA-384           |             |    X    |               |             |                 |                   |         |
| SHA-512           |             |    X    |               |             |                 |                   |         |
| RSASSA-PKCS1-v1_5 |      X      |         |       X       |      X      |                 |                   |         |
| RSA-PSS           |      X      |         |       X       |      X      |                 |                   |         |
| RSA-OAEP          |      X      |         |       X       |             |        X        |         X         |         |
| AES-CBC           |      X      |         |       X       |             |        X        |         X         |         |
| AES-CTR           |      X      |         |       X       |             |        X        |         X         |         |
| AES-ECB <sub>2</sub> |      X      |         |       X       |             |        X        |         X         |         |
| AES-GCM           |      X      |         |       X       |             |        X        |         X         |         |
| AES-KW            |      X      |         |       X       |             |                 |         X         |         |
| AES-CMAC          |      X      |         |       X       |      X      |                 |                   |         |
| ECDSA             |      X      |         |       X       |      X      |                 |                   |         |
| ECDH              |      X      |         |       X       |             |                 |                   |    X    |
| HMAC              |      X      |         |       X       |      X      |                 |                   |         |
| PBKDF2            |             |         |       X       |             |                 |                   |    X    |
| DES-CBC           |      X      |         |       X       |             |        X        |         X         |         |
| DES-EDE3-CBC      |      X      |         |       X       |             |        X        |         X         |         |

<sub>2 ECB support is not defined by the WebCrypto specifications. Use of ECB in a safe way is hard, it was added for the purpose of enabling interoperability with an existing system. We recommend against its use unless needed for interoperability.</sub>

## Using

```js
import { Crypto } from "node-webcrypto-ossl";

const crypto = new Crypto();
```

## Elliptic curve secp256k1

`secp256k1` curve is not defined by the WebCrypto specifications. This module implements `K-256` curve for ECDSA algorithm.

[K-256 curve examples](https://github.com/PeculiarVentures/webcrypto-core/blob/master/spec/EC_K_256.md)

## KeyStorage

To use KeyStorage you should init WebCrypto with `directory` option. If `directory` option is missing then `keyStorage` is `null`

```javascript
const { Crypto } = require("node-webcrypto-ossl");

const crypto = new Crypto({
  directory: "key_storage"
})
```

KeyStorage implements interface of [W3 Storage](https://developer.mozilla.org/en-US/docs/Web/API/Storage)

```js
// generating RSA key
const keys = await crypto.subtle.generateKey({
    name: "RSASSA-PKCS1-v1_5",
    modulusLength: 1024,
    publicExponent: new Uint8Array([1, 0, 1]),
    hash: {
      name: "SHA-1"
    }
  },
    false,
    ["sign", "verify"]
  );

/** 
 * saving private RSA key to KeyStorage
 * creates file ./key_storage/prvRSA-1024
 */
await crypto.keyStorage.setItem(keyPairs.privateKey, "prvRSA-1024");
```

To get key from KeyStorage
```js
var rsaKey = await webcrypto.keyStorage.getItem("prvRSA-1024");
```

## Threat Model

The threat model is defined in terms of what each possible attacker can achieve. 

### Assumptions

TODO: ADD ASSUMPTIONS

### Threats From A node-webcrypto-ossl Defect

TODO: ADD THREATS FROM NODE-WEBCRYPTO-OSSL DEFECT

### Threats From Weak Cryptography

TODO: ADD THREATS FROM WEAK CRYPTOGRAPHY

### Threats From Improper Use Of Cryptography

TODO: ADD THREATS FOR IMPROPER USE OF CRYPTOGRAPHY


## Bug Reporting
Please report bugs either as pull requests or as issues in the issue tracker. node-webcrypto-ossl has a full disclosure vulnerability policy. Please do NOT attempt to report any security vulnerability in this code privately to anybody.


## Related
 - [node-webcrypto-p11](https://github.com/PeculiarVentures/node-webcrypto-p11)
 - [webcrypto-liner](https://github.com/PeculiarVentures/webcrypto-liner)
 - [WebCrypto Examples](https://github.com/diafygi/webcrypto-examples)
 - [OpenSSL](https://github.com/openssl/openssl)
 - [OpenSSL AES GCM encrypt/decrypt](https://wiki.openssl.org/index.php/EVP_Authenticated_Encryption_and_Decryption)
