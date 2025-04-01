# Accepting Bitcoin Payments Without BTCPay Server: A Developer's Guide

## Table of Contents
- [Overview](#overview)
- [How Bitcoin Payments Work](#how-bitcoin-payments-work)
- [Method 1: Using Third-Party APIs](#method-1-using-third-party-apis)
- [Method 2: Direct Blockchain Tracking](#method-2-direct-blockchain-tracking)
- [HD Wallet Implementation](#hd-wallet-implementation)
- [Security Considerations](#security-considerations)
- [Conclusion](#conclusion)

## Overview
Bitcoin payments are becoming more common, but integrating them into a website often feels overwhelming. Many solutions, like BTCPay Server, require hosting, maintenance, and additional costs. This guide demonstrates a lightweight approach that removes the need for a VPS while maintaining full control over payments.

## How Bitcoin Payments Work
When someone sends Bitcoin, the transaction is broadcasted to the Bitcoin network and added to the blockchain after verification. Each transaction consists of:

- **Inputs**: Where the Bitcoin comes from
- **Outputs**: The recipient's address and amount
- **Fees**: Paid to miners for confirming the transaction

Once the payment receives enough **confirmations** (blocks mined after it), it's final and irreversible.

## Method 1: Using Third-Party APIs

### Available APIs
- **Bitnob**: Bitcoin payments, Lightning Network transactions, fiat on/off-ramps
- **Explorer.cash**: Direct Bitcoin payments without custody
- **Lightspark**: Lightning Network API for instant transactions
- **ZeroHash**: Institutional-grade payment APIs
- **Blockstream.info**: Invoice tracking and payment notifications
- **NowPayments**: Merchant-focused with fiat conversion
- **CoinGate**: Multi-cryptocurrency support
- **BitcoinAverage**: Market data and exchange rate APIs

### Implementation

1. **Install the API Library**
```sh
composer require explorer-cash/api-php
```

2. **Create a Payment Request**
```php
use ExplorerCash\PaymentRequest;

$paymentRequest = new PaymentRequest();
$paymentRequest->push([
    'unit'              => 'BTC',
    'address'           => 'your_bitcoin_address',
    'payment_reference' => 'ORDER-12345',
    'amount'            => '0.01',
    'callback_url'      => 'https://yourdomain.com/payment-callback',
    'timeout'           => 10,
    'confirmations'     => 3,
]);

$paymentId = $paymentRequest->paymentId();
```

3. **Handle Payment Notifications**
```php
use ExplorerCash\PaymentRequest;

$paymentRequest = new PaymentRequest();
$paymentData = $paymentRequest->pull();

if ($paymentData['status'] === 'PAID') {
    // Update order status
}
```

## Method 2: Direct Blockchain Tracking

### Implementation

1. **Install Dependencies**
```sh
composer require bitwasp/bitcoin
```

2. **Generate Bitcoin Address**
```php
require 'vendor/autoload.php';

use BitWasp\Bitcoin\Key\Factory\PrivateKeyFactory;
use BitWasp\Bitcoin\Network\NetworkFactory;

$privateKey = PrivateKeyFactory::create();
$publicKey = $privateKey->getPublicKey();
$network = NetworkFactory::bitcoin();
$address = $publicKey->getAddress()->getAddress($network);
```

3. **Monitor Payments**
```php
$address = 'your_generated_bitcoin_address';
$apiUrl = "https://blockchain.info/rawaddr/$address";

$response = file_get_contents($apiUrl);
$data = json_decode($response, true);

$balance = $data['final_balance']; // Balance in satoshis
```

4. **Implement Webhook Handler**
```php
$payload = file_get_contents('php://input');
$data = json_decode($payload, true);

if ($data && isset($data['tx_hash'])) {
    $txHash = $data['tx_hash'];
    $amount = $data['value'];
    file_put_contents('payments.log', "Payment received: $txHash, Amount: $amount\n", FILE_APPEND);
}
```

## HD Wallet Implementation

### Required Imports
```php
use BitWasp\Bitcoin\Address\PayToPubKeyHashAddress;
use BitWasp\Bitcoin\Address\SegwitAddress;
use BitWasp\Bitcoin\Bitcoin;
use BitWasp\Bitcoin\Key\Factory\HierarchicalKeyFactory;
use BitWasp\Bitcoin\Mnemonic\Bip39\Bip39SeedGenerator;
use BitWasp\Bitcoin\Mnemonic\MnemonicFactory;
use BitWasp\Bitcoin\Network\NetworkFactory;
```

### Wallet Initialization
```php
function initializeWallet() {
    $bip39 = MnemonicFactory::bip39();
    $mnemonic = $bip39->create(128); // 12 words
    
    $seedGenerator = new Bip39SeedGenerator();
    $seed = $seedGenerator->getSeed($mnemonic);
    
    $hierarchicalKeyFactory = new HierarchicalKeyFactory();
    $masterKey = $hierarchicalKeyFactory->fromEntropy($seed);
    
    return $masterKey;
}
```

### Address Generation
```php
function generateBitcoinAddress($accountIndex = 0, $addressIndex = null, $masterKey, $network) {
    if ($addressIndex === null) {
        $addressIndex = mt_rand(0, 2147483647);
    }

    $path = "84'/0'/{$accountIndex}'/0/{$addressIndex}";
    $childKey = $masterKey->derivePath($path);
    $publicKey = $childKey->getPublicKey();
    
    $witnessProgram = WitnessProgram::v0($publicKey->getPubKeyHash());
    $address = new SegwitAddress($witnessProgram);

    return [
        'address' => $address->getAddress($network),
        'path' => $path,
        'index' => $addressIndex
    ];
}
```

## **Conclusion**  

Accepting Bitcoin payments doesn’t have to be complicated. Whether you use a third-party API like **Explorer.cash**, **Blockstream.info**, or **NowPayments**, or track transactions directly, both methods eliminate the need for a full BTCPay Server setup. You gain flexibility, control, and a simple way to integrate Bitcoin into your website.  

Want full autonomy? Use **direct blockchain tracking**. Need a quick setup? **Third-party APIs** handle the heavy lifting. Either way, you’re now equipped to accept Bitcoin payments without unnecessary infrastructure.
