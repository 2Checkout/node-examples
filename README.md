2Checkout Payment API Node.js Tutorial
=========================

In this tutorial we will walk through integrating the 2Checkout Payment API to securely tokenize and charge a credit card using the [2Checkout Node Library](https://www.2checkout.com/documentation/libraries/node). You will need a 2Checkout sandbox account to complete the tutorial so if you have not already, [signup for an account](https://sandbox.2checkout.com/sandbox/signup) and [generate your Payment API keys](https://www.2checkout.com/documentation/sandbox/payment-api-testing).

----

### Application Setup

For our example application, we will be using node v0.10.26 and the express framework (4.x). You can install the dependencies below.

```
npm install express body-parser http
```

We will also need to install the twocheckout library which provides us with a simple bindings to the API, INS and Checkout process so that we can integrate each feature with only a few lines of code. In this example, we will only be using the Payment API functionality of the library.

```
git clone https://github.com/2Checkout/2checkout-node.git
npm install 2checkout-node
```

To start off, lets setup the file structure for our example application.

```
└── payment-api
    ├── app.js
    └── public
        └── index.html
```

Here we created a new directory for our application named 'payment-api' with a new file 'app.js' for our node script. We also created a sub-directory 'public' with an 'index.html' file to display our credit card form.

----

# Create a Token

Open the 'index.html' file under the public directory and create a basic HTML skeleton.

```
<!DOCTYPE html>
<html>
    <head>
        <title>Node Example</title>
    </head>
    <body>

    </body>
</html>
```

Next add a basic credit card form that allows our buyer to enter in their card number, expiration month and year and CVC.

```
<form id="myCCForm" action="/order" method="post">
    <input id="token" name="token" type="hidden" value="">
    <div>
        <label>
            <span>Card Number</span>
        </label>
        <input id="ccNo" type="text" size="20" value="" autocomplete="off" required />
    </div>
    <div>
        <label>
            <span>Expiration Date (MM/YYYY)</span>
        </label>
        <input type="text" size="2" id="expMonth" required />
        <span> / </span>
        <input type="text" size="2" id="expYear" required />
    </div>
    <div>
        <label>
            <span>CVC</span>
        </label>
        <input id="cvv" size="4" type="text" value="" autocomplete="off" required />
    </div>
    <input type="submit" value="Submit Payment">
</form>
```

Notice that we have a no 'name' attributes on the input elements that collect the credit card information. This will insure that no sensitive card data touches your server when the form is submitted. Also, we include a hidden input element for the token which we will submit to our server to make the authorization request.

Now we can add our JavaScript to make the token request call. Replace 'sandbox-seller-id' and 'sandbox-publishable-key' with your credentials.

```
<script src="//ajax.googleapis.com/ajax/libs/jquery/1.11.0/jquery.min.js"></script>
<script src="https://www.2checkout.com/checkout/api/2co.min.js"></script>

<script>
    // Called when token created successfully.
    var successCallback = function(data) {
        var myForm = document.getElementById('myCCForm');

        // Set the token as the value for the token input
        myForm.token.value = data.response.token.token;

        // IMPORTANT: Here we call `submit()` on the form element directly instead of using jQuery to prevent and infinite token request loop.
        myForm.submit();
    };

    // Called when token creation fails.
    var errorCallback = function(data) {
        if (data.errorCode === 200) {
            tokenRequest();
        } else {
            alert(data.errorMsg);
        }
    };

    var tokenRequest = function() {
        // Setup token request arguments
        var args = {
            sellerId: "sandbox-seller-id",
            publishableKey: "sandbox-private-key",
            ccNo: $("#ccNo").val(),
            cvv: $("#cvv").val(),
            expMonth: $("#expMonth").val(),
            expYear: $("#expYear").val()
        };

        // Make the token request
        TCO.requestToken(successCallback, errorCallback, args);
    };

    $(function() {
        // Pull in the public encryption key for our environment
        TCO.loadPubKey('sandbox');

        $("#myCCForm").submit(function(e) {
            // Call our token request function
            tokenRequest();

            // Prevent form from submitting
            return false;
        });
    });
</script>
```

Let's take a second to look at what we did here. First we pulled in a jQuery library to help us with manipulating the document.
(The 2co.js library does NOT require jQuery.)

Next we pulled in the 2co.js library so that we can make our token request with the card details.

```
<script src="https://www.2checkout.com/checkout/api/2co.min.js"></script>
```

This library provides us with 2 functions, one to load the public encryption key, and one to make the token request.

The `TCO.loadPubKey(String environment, Function callback)` function must be used to asynchronously load the public encryption key for the 'production' or 'sandbox' environment. In this example, we are going to call this as soon as the document is ready so it is not necessary to provide a callback.

```
TCO.loadPubKey('sandbox');
```

The the 'TCO.requestToken(Function callback, Function callback, Object arguments)' function is used to make the token request. This function takes 3 arguments:

* Your success callback function which accepts one argument and will be called when the request is successful.
* Your error callback function which accepts one argument and will be called when the request results in an error.
* An object containing the credit card details and your credentials.
    * **sellerId** : 2Checkout account number
    * **publishableKey** : Payment API publishable key
    * **ccNo** : Credit Card Number
    * **expMonth** : Card Expiration Month
    * **expYear** : Card Expiration Year
    * **cvv** : Card Verification Code

```
TCO.requestToken(successCallback, errorCallback, args);
```




In our example we created 'tokenRequest' function to setup our arguments by pulling the values entered on the credit card form and we make the token request.

```
var tokenRequest = function() {
    // Setup token request arguments
    var args = {
        sellerId: "sandbox-seller-id",
        publishableKey: "sandbox-publishable-key",
        ccNo: $("#ccNo").val(),
        cvv: $("#cvv").val(),
        expMonth: $("#expMonth").val(),
        expYear: $("#expYear").val()
    };

    // Make the token request
    TCO.requestToken(successCallback, errorCallback, args);
};
```

We then call this function from a submit handler function that we setup on the form.

```
$("#myCCForm").submit(function(e) {
    // Call our token request function
    tokenRequest();

    // Prevent form from submitting
    return false;
});
```

The 'successCallback' function is called if the token request is successful. In this function we set the token as the value for our 'token' hidden input element and we submit the form to our server.

```
var successCallback = function(data) {
    var myForm = document.getElementById('myCCForm');

    // Set the token as the value for the token input
    myForm.token.value = data.response.token.token;

    // IMPORTANT: Here we call `submit()` on the form element directly instead of using jQuery to prevent and infinite token request loop.
    myForm.submit();
};
```

The 'errorCallback' function is called if the token request fails. In our example function, we check for error code 200, which indicates that the ajax call has failed. If the error code was 200, we automatically re-attempt the tokenization, otherwise, we alert with the error message.

```
var errorCallback = function(data) {
    if (data.errorCode === 200) {
        tokenRequest();
    } else {
        alert(data.errorMsg);
    }
};
```

----

# Create the Sale

Open 'app.js' and add require the modules we need.

```
var Twocheckout = require('2checkout-node');
var express = require('express');
var http = require('http');
var bodyParser = require('body-parser');
```

Next lets create our new express instance, define routes for '/' and '/order' and instruct our application to start on port 3000.

```
var app = express();
app.use(express.static(__dirname + '/public'));
app.set('port', process.env.PORT || 3000);
app.use(bodyParser.urlencoded());


app.get('/', function(request, response) {

});


app.post('/order', function(request, response) {

});

http.createServer(app).listen(app.get('port'), function(){
    console.log('Express server listening on port ' + app.get('port'));
});
```

In the '/' route, we will render our 'index.html' and display it to the buyer.

```
app.get('/', function(request, response) {
    response.render('index')
});
```

In the order route, we will use the token passed from our credit card form to submit the authorization request and display the response. Replace 'sandbox-seller-id' and 'sandbox-private-key' with your credentials.


```
app.post('/order', function(request, response) {
    var tco = new Twocheckout({
        sellerId: "sandbox-seller-id",                                  // Seller ID, required for all non Admin API bindings 
        privateKey: "sandbox-private-key",                              // Payment API private key, required for checkout.authorize binding
        sandbox: true                                                   // Uses 2Checkout sandbox URL for all bindings
    });

    var params = {
        "merchantOrderId": "123",
        "token": request.body.token,
        "currency": "USD",
        "total": "10.00",
        "billingAddr": {
            "name": "Testing Tester",
            "addrLine1": "123 Test St",
            "city": "Columbus",
            "state": "Ohio",
            "zipCode": "43123",
            "country": "USA",
            "email": "example@2co.com",
            "phoneNumber": "5555555555"
        }
    };

    tco.checkout.authorize(params, function (error, data) {
        if (error) {
            response.send(error.message);
        } else {
            response.send(data.response.responseMsg);
        }
    });
});
```

Lets break down this method a bit and explain what were doing here.

First we setup our credentials and the environment by creating a new 'Twocheckout({})' instance. This function accepts an object as the argument containing the following attributes.
    * privateKey: Your Payment API private key
    * sellerId: 2Checkout account number
    * sandbox: Set to true to use sandbox (optional)

Next we create an object with our authorization attributes. In our example we are using hard coded strings for each required attribute except for the token which is passed in from the credit card form.

**Important Note: A token can only be used for one authorization call, and will expire after 30 minutes if not used.**

```
var params = {
    "merchantOrderId": "123",
    "token": request.body.token,
    "currency": "USD",
    "total": "10.00",
    "billingAddr": {
        "name": "Testing Tester",
        "addrLine1": "123 Test St",
        "city": "Columbus",
        "state": "Ohio",
        "zipCode": "43123",
        "country": "USA",
        "email": "example@2co.com",
        "phoneNumber": "5555555555"
    }
};
```

Finally we submit the charge using the 'tco.checkout.authorize(object, function (error, data)' function and display the result to the buyer. It is important that your callback function accepts 2 arguments to insure that you handle both the 'failure' and 'success' scenarios.

```
tco.checkout.authorize(params, function (error, data) {
    if (error) {
        response.send(error.message);
    } else {
        response.send(data.response.responseMsg);
    }
});
```

----

# Run the example application

In your console, navigate to the 'payment-api' directory and startup the application.

```
node app.js
```

In your browser, navigate to 'http://localhost:3000', and you should see a payment form where you can enter credit card information.

For your testing, you can use these values for a successful authorization
>Credit Card Number: 4000000000000002

>Expiration date: 10/2020

>cvv: 123

And these values for a failed authorization:

>Credit Card Number: 4333433343334333

>Expiration date: 10/2020

>cvv:123

If you have any questions, feel free to send them to techsupport@2co.com
