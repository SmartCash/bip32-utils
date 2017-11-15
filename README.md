# bip32-utils-smart

[![NPM](http://img.shields.io/npm/v/bip32-utils-smart.svg)](https://www.npmjs.org/package/bip32-utils-smart)


A set of utilities for working with BIP32.
Compatible with smartcashjs-lib `^2.0.0` and `^3.0.0`.


## Example

#### BIP32 Account
``` javascript
var smartcash = require('smartcashjs-lib')
var bip32utils = require('bip32-utils-smart')

// ...

var seedHex = "0000000000000000000000000000000000000000000000000000000000000000"
var m = smartcash.HDNode.fromSeedHex(seedHex)
var i = m.deriveHardened(0)
var external = i.derive(0)
var internal = i.derive(1)
var account = new bip32utils.Account([
  new bip32utils.Chain(external.neutered()),
  new bip32utils.Chain(internal.neutered())
])

console.log(account.getChainAddress(0))
// => SZxotLyNBNeipNc3LQReTgavbWB84VThES

account.nextChainAddress(0)

console.log(account.getChainAddress(0))
// => SgEJfVHfiPvRLWJmVVCMFNwCgsGoGNgwB3

console.log(account.getChainAddress(1))
// => SSDgMJVrQyktpGvJkuZZ8KCGVDgnTHm5w2

console.log(account.derive('SSDgMJVrQyktpGvJkuZZ8KCGVDgnTHm5w2').toBase58())
// => xpub6DJB5bmtQyrkk35UMaCu3hSM2bFMEEFzLq8Dne3jK8YXdjVY5qGWQWBqcRnVv2iDm8HnCCcZE3nNRt2EiWDyByFnh5BKdaVcCQeDCfuew9Q

// NOTE: passing in the parent nodes allows for private key escalation (see xprv vs xpub)

console.log(account.derive('SSDgMJVrQyktpGvJkuZZ8KCGVDgnTHm5w2', [external, internal]).toBase58())
// => xprv9zJpg6EzacJTXZ11FYftgZVcUZQrpmY8ycCczFe7ko1YkwAPYHxFrhsMm9YjnaiS9SLpzjabw7JrrWLRb7tPnzaUYhT9UeWt3tFpw26wX7T
```


#### BIP32 Chains
``` javascript
var smartcash = require('smartcashjs-lib')
var bip32utils = require('bip32-utils-smart')

// ...

var seedHex = "0000000000000000000000000000000000000000000000000000000000000000"
var hdNode = smartcash.HDNode.fromSeedHex(seedHex)
var chain = new bip32utils.Chain(hdNode)

for (var k = 0; k < 10; ++k) chain.next()

var address = chain.get()

console.log(chain.find(address))
// => 10

console.log(chain.pop())
// => address
// => SkEHK9j8nyWs7EydNPeEkSXZ4MiUN3JeSq
```


#### BIP32 Discovery (manual)
``` javascript
var bip32utils = require('bip32-utils-smart')
var smartcash = require('smartcashjs-lib')
var Blockchain = require('cb-blockr')

// ...

var seedHex = "0000000000000000000000000000000000000000000000000000000000000000"
var blockchain = new Blockchain('testnet')
var hdNode = smartcash.HDNode.fromSeedHex(seedHex)
var chain = bip32utils.Chain(hdNode)
var GAP_LIMIT = 20

bip32utils.discovery(chain, GAP_LIMIT, function(addresses, callback) {
  blockchain.addresses.summary(addresses, function(err, results) {
    if (err) return callback(err)

    var areUsed = results.map(function(result) {
      return result.totalReceived > 0
    })

    callback(undefined, areUsed)
  })
}, function(err, used, checked) {
  if (err) throw err

  console.log('Discovered at most ' + used + ' used addresses')
  console.log('Checked ' + checked + ' addresses')
  console.log('With at least ' + (checked - used) + ' unused addresses')

  // throw away ALL unused addresses AFTER the last unused address
  var unused = checked - used
  for (var i = 1; i < unused; ++i) chain.pop()

  // remember used !== total, chain may have started at a k-index > 0
  console.log('Total number of addresses (after prune): ', chain.addresses.length)
})
```


## LICENSE [MIT](LICENSE)
