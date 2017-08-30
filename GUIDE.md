## Using GloBee Java Client Library

This SDK provides a convenient abstraction of GloBee's [cryptographically-secure API](https://globee.com/api) and allows payment gateway developers to focus on payment flow/e-commerce integration rather than on the specific details of client-server interaction using the API.  This SDK optionally provides the flexibility for developers to have control over important details, including the handling of private keys needed for client-server communication.

This SDK implements GloBee's remote client authentication and authorization strategy.  No private or shared-secret information is ever transmitted over the wire.

### Dependencies

You must have a GloBee merchant account to use this SDK.  It's free to [sign-up for a BitPay merchant account](https://globee.com/register).

### Usage

This library was built and tested using the Eclipse IDE; the source code tree is directly compatible with Eclipse.
Library dependencies can be downloaded by executing the following command at the root of the library:

```
mvn clean dependency:copy-dependencies -DoutputDirectory=./lib
```

### Handling your client private key

Each client paired with the GloBee server requires a public and private key.  This provides the security mechanism for all client interaction with the GloBee server. The public key is used to derive the specific client identity that is displayed on your GloBee dashboard.  The public key is also used for securely signing all API requests from the client.  See the [GloBee API](https://globee.com/api) for more information.

The private key should be stored in the client environment such that it cannot be compromised.  If your private key is compromised you should revoke the compromised client identity from the GloBee server and re-pair your client, see the [API tokens](https://globee.com/api-tokens) for more information.

This SDK provides the capability of internally storing the private key on the client local file system.  If the local file system is secure then this is a good option.  It is also possible to generate the key yourself (using the SDK) and store the key as required.  It is not recommended to transmit the private key over any public or unsecured networks.

```java
// Let the SDK store the private key on the clients local file system.
BitPay bitpay = new BitPay();
```

```java
// Create the private key using the SDK, store it as required, and inject the private key into the SDK.
ECKey key = KeyUtils.createEcKey();
this.bitpay = new BitPay(key);
```

```java
// Create the private key external to the SDK, store it in a file, and inject the private key into the SDK.
String privateKey = KeyUtils.getKeyStringFromFile(privateKeyFile);
ECKey key = KeyUtils.createEcKeyFromHexString(privateKey);
this.bitpay = new BitPay(key);
```

###Pair your client with GloBee

Your client must be paired with the GloBee server.  The pairing initializes authentication and authorization for your client to communicate with GloBee for your specific merchant account.  There are two pairing modes available; client initiated and server initiated.

#### Client initiated pairing

Pairing is accomplished by having your client request a pairing code from the GloBee server.  The pairing code is then entered into the GloBee merchant dashboard for the desired merchant.  Your interactive authentication at https://globee.com/login provides the authentication needed to create finalize the client-server pairing request.

```java
String clientName = "server 1";
BitPay bitpay = new BitPay(clientName);        
        
if (!bitpay.clientIsAuthorized(BitPay.FACADE_POS))
{
  // Get POS facade authorization code.
  String pairingCode = bitpay.requestClientAuthorization(BitPay.FACADE_POS);
  
  // Signal the device operator that this client needs to be paired with a merchant account.
  System.out.print("Info: Pair this client with your merchant account using the pairing code: " + pairingCode);
  throw new BitPayException("Error: client is not authorized for POS facade.");
}
```

#### Server initiated pairing

Pairing is accomplished by obtaining a pairing code from the GloBee server.  The pairing code is then injected into your client (typically during client initialization/configuration).  Your interactive authentication at https://globee.com/login provides the authentication needed to create finalize the client-server pairing request.

```java
// Obtain a pairingCode from your BitPay account administrator. 
String pairingCode = "xxxxxxx";
String clientName = "server 1";
BitPay bitpay = new BitPay(clientName);

// Is this client already authorized to use the POS facade?
if (!bitpay.clientIsAuthorized(BitPay.FACADE_POS))
{
  // Get POS facade authorization.
  bitpay.authorizeClient(pairingCode);
}	
```

### Create an invoice

```java
Invoice invoice = bitpay.createInvoice(100, "USD");

String invoiceUrl = invoice.getURL();

String status = invoice.getStatus();
```

### Create an invoice (extended)

You can add optional attributes to the invoice.  Attributes that are not set are ignored or given default values. For example:

```java
InvoiceBuyer buyer = new InvoiceBuyer();
buyer.setName("Satoshi");
buyer.setEmail("satoshi@bitpay.com");
	
Invoice invoice = new Invoice(100.0, "USD");
invoice.setBuyer(buyer);
invoice.setFullNotifications(true);
invoice.setNotificationEmail("satoshi@bitpay.com");
invoice.setPosData("ABCDEFGHIJKLMNOPQRSTUVWXYZ1234567890");

invoice = this.bitpay.createInvoice(invoice);
```

### Retreive an invoice

```java
invoice = bitpay.getInvoice(invoice.getId());
```

### Get exchange Rates

You can retrieve GloBee's [BBB exchange rates](https://globee.com):

```java
Rates rates = this.bitpay.getRates();

double rate = rates.getRate("USD");

rates.update();
```

