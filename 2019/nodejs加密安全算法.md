## 深入了解NodeJS加密模块Crypto

## 哈希函数：Hash

## 对称密钥加密：Cipher、Decipher

过程：

- a）A 使用密钥加密数据
- b）A 将密文发送给 B
- c）B 收到密文后，使用相同的密钥对其进行解密，取得原始数据<br>
  优点：速度快<br>
  缺点：密钥被盗就被破解、密钥管理不方便（每个用户都要对应一个密钥）<br>
  实现算法有：凯撒密码，AES（Advanced Encryption Standard）、DES（Data Encryption Standard）、动态口令等。<br>
  推荐：AES

``` javascript
const crypto = require('crypto');

const algorithm = 'aes-192-cbc';
const password = 'Password used to generate key';
// Use the async `crypto.scrypt()` instead.
const key = crypto.scryptSync(password, 'salt', 24);
// Use `crypto.randomBytes` to generate a random iv instead of the static iv
// shown here.
const iv = Buffer.alloc(16, 0); // Initialization vector.

const cipher = crypto.createCipheriv(algorithm, key, iv);

let encrypted = cipher.update('some clear text data', 'utf8', 'hex');
encrypted += cipher.final('hex');
console.log(encrypted);
// Prints: e5f79c5915c02171eec6b212d5520d44480993d7d622a7c4c2da32f6efda0ffa
```

``` javascript
const crypto = require('crypto');

const algorithm = 'aes-192-cbc';
const password = 'Password used to generate key';
// Use the async `crypto.scrypt()` instead.
const key = crypto.scryptSync(password, 'salt', 24);
// The IV is usually passed along with the ciphertext.
const iv = Buffer.alloc(16, 0); // Initialization vector.

const decipher = crypto.createDecipheriv(algorithm, key, iv);

// Encrypted using same algorithm, key and iv.
const encrypted =
  'e5f79c5915c02171eec6b212d5520d44480993d7d622a7c4c2da32f6efda0ffa';
let decrypted = decipher.update(encrypted, 'hex', 'utf8');
decrypted += decipher.final('utf8');
console.log(decrypted);
// Prints: some clear text data
```

openssl list -cipher-algorithms可以列出支持的加密算法


## 非对称密钥加密

### 公开密钥加密

过程：

- a）首先由接收方 B 生成公钥和私钥
- b）B 把公钥发送给 A
- c）A 使用 B 发来的公钥加密数据，然后发送给 B
- d）B 使用私钥对密文进行解密，得到原始数据
  优点：安全性高、密钥管理方便
  缺点：加密速度慢、无法防止中间人攻击，A 不知道收到的公钥是否是来自 B
  实现算法有：RSA 算法、椭圆曲线加密算法等
  推荐：RSA
  RSA：privateEncrypt、privateDecrypt、publicEncrypt、publicDecrypt

``` javascript
// 公钥加密
let encryptString = crypto.publicEncrypt({
      key: publicKey,
      padding: crypto.constants.RSA_NO_PADDING
    }, Buffer.from('需要加密的内容'));
encryptString = encryptString.toString('base64');

// 私钥加密
let encryptString = crypto.privateEncrypt({
      key: privateKey,
      padding: crypto.constants.RSA_NO_PADDING
    }, Buffer.from('需要加密的内容'));
encryptString = encryptString.toString('base64');
```

``` javascript
crypto.privateDecrypt(privateKey, buffer);
crypto.publicDecrypt(key, buffer);
```

### 混合加密

过程：

- a）接收方 B 事先生成公钥和私钥
- b）B 将公钥发送给 A
- c）A 使用收到的公钥对共享密钥（对称密钥）进行加密，并发送给 B
- d）B 使用私钥解密，得到共享密钥
- e）接下来 A 只要使用对称密钥加密好数据发送给 B 即可<br>
  优点：使用处理速度快的对称密钥加密数据同时保证对称密钥的安全性
  HTTPS 的 TLS 加密就是用的混合加密

### 密钥交换协议

过程：

- a）A 生成密钥 P
- b）A 把密钥 P 发送给 B
- c）A 和 B 各自准备自己的私钥 SA 和 SB
- d）A 利用密钥 P 和私钥 SA 合成新的密钥 P-SA
- e）B 也利用密钥 P 和私有密钥 SB 合成新的密钥 P-SB
- f）A 将密钥 P-SA 发送给 B，B 也将密钥 P-SB 发送给 A
- g）A 将私有密钥 SA 和收到的密钥 P-SB 合成新的密钥 SA-P-SB（合成结果和合成顺序无关，合成密钥无法被分解）
  h）同样地，B 也将私钥 SB 和收到的密钥 P-SA 合成新的密钥 P-SA-SB。于是 A 和 B 都得到了密钥 P-SA-SB。这个密钥将作为“加密密钥”和“解密密钥”使用。<br>
  优点：DH 利用“离散对数问题”解决中间人攻击<br>
  实现算法有：DiffieHellman、DiffieHellmanGroup、ECDH

``` javascript
const crypto = require('crypto');
const assert = require('assert');

// Generate Alice's keys...
const alice = crypto.createDiffieHellman(2048);
const aliceKey = alice.generateKeys();

// Generate Bob's keys...
const bob = crypto.createDiffieHellman(alice.getPrime(), alice.getGenerator());
const bobKey = bob.generateKeys();

// Exchange and generate the secret...
const aliceSecret = alice.computeSecret(bobKey);
const bobSecret = bob.computeSecret(aliceKey);

// OK
assert.strictEqual(aliceSecret.toString('hex'), bobSecret.toString('hex'));
```

### 消息认证码 Hmac：Hmac

过程：

- a）A 向 B 发送密文前，A 生成了一个用于制作消息认证码的密钥，然后使用安全的方法将密钥发送给了 B
- b）A 使用密文和密钥生成一个值，例如生成 1abc。这个由密钥和密文生成的值就是消息认证码，简称 MAC（Message Authentication Code）
- c）A 将 MAC（1abc）和密文发送给 B
- d）和 A 一样，B 也需要使用密文和密钥生成 MAC。经过对比，B 可以确认自己计算出的值和 A 发的值是否一致。
- e）接下来 B 只需要使用密钥对密文进行解密即可
  实现算法有：HMAC（Hash-based MAC）、OMAC（One-Key MAC）、CMAC（Cipher-based MAC）<br>
  推荐：HMAC<br>
  优点：可以实现认证和检测篡改功能<br>
  缺点：MAC 不能确定密钥由哪方生成（A 或 B）

``` javascript
const crypto = require('crypto');
const hmac = crypto.createHmac('sha256', 'a secret');

hmac.update('some data to hash');
console.log(hmac.digest('hex'));
// Prints:
//   7fd04df92f636fd450bc841c9418e5825c17f33ad9c87c518115a45971f7f77e
```

### 数字签名： Sign、Verify

过程：

- a）消息发送者 A 准备好需要发送的消息、私钥和公钥
- b）A 将公钥发送给 B
- c）A 使用私钥加密消息，加密后的消息就是数字签名
- d）A 将消息和签名都发送给 B
- e）B 使用公钥对密文（签名）解密
- f）B 对解密后的消息进行确认，看它是否和收到的消息一致<br>
  优点：可以确认谁是消息发送者<br>
  缺点：无法确定公钥的制造者是谁<br>

``` javascript
let sign = crypto.createSign('RSA-SHA256')
      .update('签名内容')
      .sign(this.AppPrivateKey, 'base64');
```

``` javascript
let verify = crypto.createVerify('RSA-SHA256');
let result = verify.verify(publicKey, '签名内容');
```

### 数字证书：Certificate

过程：

- a）A 持有公钥 PA 和私钥 SA
- b）A 首先需要向认证中心申请发行证书，证明公钥 PA 确实由自己生成
- c）认证中心里保管着他们自己准备的公钥 PC 和私钥 SC
- d）A 将公钥 PA 和包含邮箱（域名）的个人资料发送给认证中心
- e）认证中心对收到的资料进行确认，判断其是否为 A 本人的资料。确认完毕后，认证中心使用自己的私钥 SC，根据 A 的资料生成数字签名
- f）认证中心将生成的数字签名和资料放进同一个文件中，然后把这个文件发送给 A，这个文件就是 A 的数字证书
- g）A 将作为公钥的数字证书发送给 B
- h）B 收到数字证书后，确认证书里的邮件地址（域名）确实是 A 的。接着，B 获取了认证中心的公钥
- i）B 对证书内的签名进行验证，判断它是否为认证中心给出的签名。证书中的签名只能用认证中心的公钥 PC 进行验证。如果验证结果没有异常，就能说明这份证书的确是由认证中心发行的
- j）确认了证书是由认证中心发行的，且邮件地址（域名）就是 A 的之后，B 从证书中取出 A 的公钥 PA<br>
  优点：解决数字签名的缺点

``` javascript
const { Certificate } = require('crypto');
const spkac = getSpkacSomehow();

const challenge = Certificate.exportChallenge(spkac);
console.log(challenge.toString('utf8'));
// Prints: the challenge as a UTF8 string

const publicKey = Certificate.exportPublicKey(spkac);
console.log(publicKey);
// Prints: the public key as <Buffer ...>

console.log(Certificate.verifySpkac(Buffer.from(spkac)));
// Prints: true or false
```
