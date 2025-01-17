# Pay.nl C# SDK
---

- [Quick start and examples](#usage)

---

This SDK is available as DotNet Assembly. 

With this SDK you will be able to start transactions and retrieve transactions with their status for the Pay.nl payment service provider.

### Usage

Setting the configuration:
```c#
PAYNLSDK.API.RequestBase.ApiToken = "e41f83b246b706291ea9ad798ccfd9f0fee5e0ab";
PAYNLSDK.API.RequestBase.ServiceId = "SL-3490-4320";
```

Getting a list of available payment methods, use the Getservice.
```c#
PAYNLSDK.API.RequestBase.ApiToken = "e41f83b246b706291ea9ad798ccfd9f0fee5e0ab";
PAYNLSDK.API.RequestBase.ServiceId = "SL-3490-4320";
PAYNLSDK.API.Transaction.GetService.Response response = PAYNLSDK.Transaction.GetService(paymentMethodId);
//paymentMethodId: is optional
//The ID of the payment method. Only the payment options linked to the provided payment method ID will be returned if an ID is provided.
//If omitted, all available payment options are returned. Use the following IDs to filter the options:
//1. SMS.
//2. Pay fixed price.
//3. Pay per call.
//4. Pay per transaction
//5. Pay per minute.
```

Starting a transaction:
```c#
PAYNLSDK.API.RequestBase.ApiToken = "e41f83b246b706291ea9ad798ccfd9f0fee5e0ab";
PAYNLSDK.API.RequestBase.ServiceId = "SL-3490-4320";

PAYNLSDK.API.Transaction.Start.Request request = PAYNLSDK.Transaction.CreateTransactionRequest("127.0.0.1", "http://example.org/visitor-return-after-payment");
request.Amount = 7184; // Amount in cents

request.PaymentOptionId = 10; // Payment profile/option
// request.PaymentOptionSubId = 5081; // Set bank id for iDEAL (example)

// Transaction data
request.Transaction = new PAYNLSDK.Objects.TransactionData();
request.Transaction.Currency = "EUR";
request.Transaction.CostsVat = null;
request.Transaction.OrderExchangeUrl = "https://example.org/exchange.php";
request.Transaction.Description = "TEST PAYMENT"; // Description for client
request.Transaction.ExpireDate = DateTime.Now.AddDays(14);

// Optional Stats data
request.StatsData = new PAYNLSDK.Objects.StatsDetails();
request.StatsData.Info = "your information";
request.StatsData.Tool = "C#.NET";
request.StatsData.Extra1 = "X";
request.StatsData.Extra2 = "Y";
request.StatsData.Extra3 = "Z";

// Initialize Salesdata
request.SalesData = new PAYNLSDK.Objects.SalesData();
request.SalesData.InvoiceDate = DateTime.Now;
request.SalesData.DeliveryDate = DateTime.Now;
request.SalesData.OrderData = new System.Collections.Generic.List<PAYNLSDK.Objects.OrderData>();

// Add products
request.SalesData.OrderData.Add(new PAYNLSDK.Objects.OrderData("SKU-8489", "Testproduct 1", 2995, "H", 1));
request.SalesData.OrderData.Add(new PAYNLSDK.Objects.OrderData("SKU-8421", "Testproduct 2", 995, "H", 1));
request.SalesData.OrderData.Add(new PAYNLSDK.Objects.OrderData("SKU-2359", "Testproduct 3", 2499, "H", 1));

// Add shipping
request.SalesData.OrderData.Add(new PAYNLSDK.Objects.OrderData("SHIPPINGCOST", "Shipping of products", 695, "H", 1, "SHIPPING"));

// enduser
request.Enduser = new PAYNLSDK.Objects.EndUser();
request.Enduser.Language = "NL"; // End user language
request.Enduser.Initials = "J.";
request.Enduser.Lastname = "Buyer";
request.Enduser.Gender = PAYNLSDK.Enums.Gender.Male;
request.Enduser.BirthDate = new DateTime(1991, 1, 23, 0, 0, 0, DateTimeKind.Local);
request.Enduser.PhoneNumber = "0612345678";
request.Enduser.EmailAddress = "email@domain.com";
request.Enduser.BankAccount = "";
request.Enduser.IBAN = "NL08INGB0000000555";
request.Enduser.BIC = "";

// enduser address
request.Enduser.Address = new PAYNLSDK.Objects.Address();
request.Enduser.Address.StreetName = "Streetname";
request.Enduser.Address.StreetNumber = "8";
request.Enduser.Address.ZipCode = "1234AB";
request.Enduser.Address.City = "City";
request.Enduser.Address.CountryCode = "NL";

// invoice address
request.Enduser.InvoiceAddress = new PAYNLSDK.Objects.Address();
request.Enduser.InvoiceAddress.Initials = "J.";
request.Enduser.InvoiceAddress.LastName = "Jansen";
request.Enduser.InvoiceAddress.Gender = PAYNLSDK.Enums.Gender.Male;
request.Enduser.InvoiceAddress.StreetName = "InvoiceStreetname";
request.Enduser.InvoiceAddress.StreetNumber = "10";
request.Enduser.InvoiceAddress.ZipCode = "1234BC";
request.Enduser.InvoiceAddress.City = "City";
request.Enduser.InvoiceAddress.CountryCode = "NL";

//System.Diagnostics.Debug.WriteLine(request.ToString()); // JSON view of data before sending


PAYNLSDK.API.Transaction.Start.Response response = PAYNLSDK.Transaction.Start(request); // Sending to PAY.
```

To determine if a transaction has been paid, you can use:
```c#
PAYNLSDK.API.RequestBase.ApiToken = "e41f83b246b706291ea9ad798ccfd9f0fee5e0ab";
PAYNLSDK.API.RequestBase.ServiceId = "SL-3490-4320";

PAYNLSDK.API.Transaction.Start.Response response;
// Perform transaction to get response object. Alternately, you could work with a stored ID.

PAYNLSDK.API.Transaction.Info.Response info = PAYNLSDK.Transaction.Info(response.transactionId);
PAYNLSDK.Enums.PaymentStatus result = info.PaymentDetails.State;

if (PAYNLSDK.Transaction.IsPaid(result) || PAYNLSDK.Transaction.IsPending(result))
{
    // redirect user to thank you page
}
else
{
    // it has not been paid yet, so redirect user back to checkout
}
```

When implementing the exchange script (where you should process the order in your backend):
```c#
PAYNLSDK.API.RequestBase.ApiToken = "e41f83b246b706291ea9ad798ccfd9f0fee5e0ab";
PAYNLSDK.API.RequestBase.ServiceId = "SL-3490-4320";

PAYNLSDK.API.Transaction.Info.Response info = PAYNLSDK.Transaction.Info(response.transactionId);
PAYNLSDK.Enums.PaymentStatus result = info.State;

if (PAYNLSDK.Transaction.IsPaid(result))
{
    // process the payment
}
else 
{
 if(PAYNLSDK.Transaction.IsCancelled(result)){
    // payment canceled, restock items
 }
}

response.Write("TRUE| ");
// Optionally you can send a message after TRUE|, you can view these messages in the logs.
// https://admin.pay.nl/logs/payment_state
response.Write("Paid");
```

### Contributing



### License

The Assembly is available as open source under the terms of the [MIT License](http://opensource.org/licenses/MIT).
