

# **Accepting Bitcoin Payments Without BTCPay Server: A Developer’s Guide**  

Bitcoin payments are becoming more common, but integrating them into a website often feels overwhelming. Many solutions, like BTCPay Server, require hosting, maintenance, and additional costs. A developer I know showed me a lightweight approach that removes the need for a VPS while still keeping full control over payments. This guide walks you through two methods to accept Bitcoin: using third-party APIs or tracking payments directly from the blockchain. Both are independent of any programming language—this guide just happens to use PHP.  

---

## **How Bitcoin Payments Work**  

When someone sends Bitcoin, the transaction is broadcasted to the Bitcoin network and added to the blockchain after verification. The transaction has:  

- **Inputs** (where the Bitcoin comes from)  
- **Outputs** (the recipient’s address and amount)  
- **Fees** (paid to miners for confirming the transaction)  

Once the payment gets enough **confirmations** (blocks mined after it), it's final. Unlike traditional payments, Bitcoin transactions can’t be reversed.  

---

## **Method 1: Using Third-Party APIs**  

Third-party APIs simplify Bitcoin payments by handling transactions while you focus on integration. Some popular APIs include:  

- **Bitnob** Offers Bitcoin payments, Lightning Network transactions, and fiat on/off-ramps
- **Explorer.cash** Direct Bitcoin payments without custody.
- **Lightspark** Lightning Network API for instant Bitcoin transactions.
- **ZeroHash** Institutional-grade Bitcoin payment APIs.  
- **Blockstream.info** Invoice tracking and payment notifications for Bitcoin 
- **NowPayments** Merchant-focused, supports fiat conversion 
- **CoinGate** Supports multiple cryptocurrencies 
- **BitcoinAverage** Market data and exchange rate APIs 

These services let you receive payments without running a full node.  

### **Step 1: Install the API Library**  
Using **Explorer.cash** as an example:  
```sh
composer require explorer-cash/api-php
```  

### **Step 2: Create a Payment Request**  
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

### **Step 3: Handle Payment Notifications**  
```php
use ExplorerCash\PaymentRequest;

$paymentRequest = new PaymentRequest();
$paymentData = $paymentRequest->pull();

if ($paymentData['status'] === 'PAID') {
    // Update order status
}
```  
This method reduces development effort but relies on third-party services.  

---

## **Method 2: Direct Blockchain Tracking**  

If you prefer full control, generate unique Bitcoin addresses for each payment and track transactions directly on the blockchain.  

### **Step 1: Install Required Dependencies**  
```sh
composer require bitwasp/bitcoin
```  

### **Step 2: Generate a Unique Bitcoin Address**  
```php
require 'vendor/autoload.php';

use BitWasp\Bitcoin\Key\Factory\PrivateKeyFactory;
use BitWasp\Bitcoin\Network\NetworkFactory;

$privateKey = PrivateKeyFactory::create();
$publicKey = $privateKey->getPublicKey();

$network = NetworkFactory::bitcoin();
$address = $publicKey->getAddress()->getAddress($network);

echo "Generated Bitcoin Address: " . $address . PHP_EOL;
```  
Each customer should get a unique address for better tracking. **Store private keys securely.**  

### **Step 3: Monitor Payments Using a Blockchain API**  
```php
$address = 'your_generated_bitcoin_address';
$apiUrl = "https://blockchain.info/rawaddr/$address";

$response = file_get_contents($apiUrl);
$data = json_decode($response, true);

$balance = $data['final_balance']; // Balance in satoshis
echo "Current Balance: " . $balance . " satoshis" . PHP_EOL;
```  
Payments are confirmed when the balance reaches the expected amount.  

### **Step 4: Automate Payment Confirmation with Webhooks**  
```php
$payload = file_get_contents('php://input');
$data = json_decode($payload, true);

if ($data && isset($data['tx_hash'])) {
    $txHash = $data['tx_hash'];
    $amount = $data['value'];
    
    file_put_contents('payments.log', "Payment received: $txHash, Amount: $amount\n", FILE_APPEND);
    
    echo "Payment recorded successfully!";
}
```  
Instead of manually checking, webhooks notify you when a payment is received.  

---

## **Security Considerations**  

- **Private Key Management**: Never expose private keys. Use secure storage.  
- **Data Validation**: Always sanitize data from external APIs.  
- **Regulatory Compliance**: Ensure you comply with local laws regarding cryptocurrency transactions.  

---

## **Conclusion**  

Accepting Bitcoin payments doesn’t have to be complicated. Whether you use a third-party API like **Explorer.cash**, **Blockstream.info**, or **NowPayments**, or track transactions directly, both methods eliminate the need for a full BTCPay Server setup. You gain flexibility, control, and a simple way to integrate Bitcoin into your website.  

Want full autonomy? Use **direct blockchain tracking**. Need a quick setup? **Third-party APIs** handle the heavy lifting. Either way, you’re now equipped to accept Bitcoin payments without unnecessary infrastructure.
