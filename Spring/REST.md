REST stands for Representational State Transfer. The architecture is very scalable and can handle a lot of data without the complexity growing at the same pace. It's very commonly used for APIs. Roy Fielding is the man behind the definition.

REST is built on a set of rules that tells how you should handle data in the best way with certain things in mind.

- Scalability
	- The amount of data should be able to grow without getting more complex.
- Simplicity
	- Keep it as simple as possible.
- Uniformity
	- All retrievals in a standardised way, which means predictability. 
	- URLs that are used in a system should be uniform.
- Statelessness
	- Each request is self-contained with all necessary information.
- Resource-based
	- Data is represented as resources accessed through URLs and HTTP methods.

##### HATEOAS

^e46a60
In Fielding's definition HATEOAS is also included in REST.
HATEOAS stands for Hypermedia as the Engine of Application State. It's a way to provide information dynamically through hypermedia. This means that a request comes with example links that are related to the requests. 

[Wikipedia example](https://en.wikipedia.org/wiki/HATEOAS#Example)

```JSON

// Bank account with a positive balance got more links as it got more options.

{
    "account": {
        "account_number": 12345,
        "balance": {
            "currency": "usd",
            "value": 100.00
        },
        "links": {
            "deposits": "/accounts/12345/deposits",
            "withdrawals": "/accounts/12345/withdrawals",
            "transfers": "/accounts/12345/transfers",
            "close-requests": "/accounts/12345/close-requests"
        }
    }
}

// Bank account with a negative saldo only get the link for deposit.

{
    "account": {
        "account_number": 12345,
        "balance": {
            "currency": "usd",
            "value": -25.00
        },
        "links": {
            "deposits": "/accounts/12345/deposits"
        }
    }
}
```

