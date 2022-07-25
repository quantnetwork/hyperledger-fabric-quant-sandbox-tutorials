# Hyperledger Fabric Quant Sandbox Tutorial - A Loyalty Program Use Case

## Requirements
We will need to have an mDApp registered with Quant. We can register one [here](https://developer.quant.network/login).

This short tutorial will guide us on how to create a simple contract interaction flow for a loyalty program use case.

In the Hyperledger Fabric Quant sandbox, we have deployed two extended QRC Tier 1 smart contracts.

The first contract we have available is a QRC-20 type contract with an additional token "request" functionality. Quant Fungible Tokens (QRC-20) are used to represent multiple individual but replaceable units, such as currency, shares, or fractionalised assets. In this tutorial, QRC-20 tokens represent Loyalty Points acquired through an airline loyalty program.

Here is a list of the most useful QRC-20 functions to us:

| Function Type  |  Function Name | What Does it Do?  | Extension (Y/N)  | Notes   |
|---|---|---|---|---|
| Write  | transfer  |  Transfers the given amount from the Payer to the Payee (Recipient) account | N  |   |
| Write  |  approve | Approves the given amount to be taken from the Payer’s (Spender) account by the Payee (Recipient) account  |  N |   |
| Write  | transferFrom  |  Initiated by the Payee, the function takes the given amount to be from the Payer’s (Sender) account and transfers it to the Payee (Recipient) account | N  |  This function requires pre-approval for the amount to be taken – see the Approve function |
| Write  | requestTokens  |  Sends the default amount of tokens to the transaction sender's account, if the new balance of the account does not exceed the maximum token amount. | Y  | Function added to support the Loyalty program use case  |
| Read  | balanceOf  |Takes an address and returns the current balance for that address of the QRC-20 token as a uint256|  N |   |
| Read  | decimals  |  Returns the number of decimals for the QRC-20 token as a uint8 | N  |   |
| Read  | name  |  Returns the name of the QRC-20 token as a string | N  |   |
| Read  | symbol  |  Returns the symbol of the QRC-20 token as a string |  N |   |


The second contract we have available is a QRC-721 contract with an additional token "redemption" functionality. Quant Non Fungible Tokens (QRC-721) are used to represent a unique asset like a piece of art, digital content, or media and is understood as an irrevocable digital certificate of ownership and authenticity for a given asset, whether digital or physical. In this tutorial, a QRC-721 token represents a Flight Reward acquired by exchanging Loyalty Points.

Here is a list of the most useful QRC-721 functions to us

| Function Type  |  Function Name | What Does it Do?  | Extension (Y/N)  | Notes   |
|---|---|---|---|---|
| Write  | transferFrom  |Transfers the given token ID to the given ‘to’ identity from the given ‘from’ identity  | N  |  If the caller of the TransferFrom is NOT the same as the ‘from’ address, or has not been approved to take the token using the approve function then the caller has to be recorded as an approved current operator of the ‘from’ address in the smart contract |
| Write  |  approve | Approves the given token ID to be transferred to the given ‘to’ identity   |  N |   |
| Write  | RedeemToken | Exchange QRC-20 tokens for a QRC-721 token | Y  | Function added to support the Loyalty program use case  |
| Read  | ownerOf  |Returns the owner of the token |  N |   |
| Read  | owner  |Returns the account with the owner role of this token contract, i.e. the account that can mint new tokens |  N |   |
| Read  | balanceOf  |Returns the number of tokens in an owner’s account |  N |   |
| Read  | name  |  Returns the name of the QRC-721 token as a string | N  |   |
| Read  | symbol  |  Returns the symbol of the QRC-721 token as a string |  N |   |


A comprehensive list of all the parameters for each function of the QRC-20 & QRC-721 contracts will be made available soon.


## Loyalty Program Use Case

Applications interact with the contracts in the following manner:

- 100 Loyalty Point tokens are requested by calling the "requestTokens" function in the QRC-20 contract
- 1 Flight Reward token is redeemed in exchange for 100 Loyalty Points by calling the "redeemToken" function in  the QRC-721 contract

## Step by Step Tutorial

To get an account created on the Hyperledger Fabric sandbox network, we need to register a new Application on the [Overledger Dashboard](https://developer.quant.network/login). The clientId assigned to the application will be automatically enrolled as the identity of the application on the sandbox network. Any previously registered application within the Overledger Dashboard will already have their clientId enrolled on the Hyperledger Fabric Quant Sandbox.

In this tutorial, we will go through a simple flow

1. Request Loyalty Point tokens
2. Check account balances for the amount of Loyalty Points we own
3. Redeem Flight Rewards for Loyalty Points

Each of the following sections follows a "preparation->execution" pattern which Overledger works by. For each API call, we will first prepare a request, then validate the response and finally execute the request.

### 1. Request Loyalty Points

As we use our clientId and access token for authentication and Hyperledger Fabric is a permissioned technology, we do not require client side management of private keys.

To claim loyalty Point tokens, we can construct a REST API request such as the following. 

Note: the `destinationId` field is set to `loyaltypoint` which is the id for the contract representing the Loyalty Point tokens.

Preparation Request:
POST `https://api.sandbox.overledger.io/v2/preparation/transaction`
```
{
    "type": "Contract Invoke",
    "location": {
        "technology": "Hyperledger Fabric",
        "network": "Quant Sandbox"
    },
    "urgency": "normal",
    "requestDetails": {
        "origin": [
            {
                "originId": "<your-clientId-here>"
            }
        ],
        "destination": [
            {
                "destinationId": "loyaltypoint",
                "smartContract": {
                    "function": {
                        "name": "requestTokens",
                        "inputParameters": [
                        ]
                    }
                }
            }
        ]
    }
}
```

Preparation Response:
```
{
    "requestId": "a58e0a1f-2251-407c-8e4b-2bf5efa092ec",
    "gatewayFee": {
        "amount": "0",
        "unit": "QNT"
    },
    "dltFee": {
        "amount": "0",
        "unit": "N/A"
    },
    "nativeData": {
        "origin": [
            {
                "originId": "<your-clientId-here>"
            }
        ],
        "destination": [
            {
                "destinationId": "loyaltypoint",
                "smartContract": {
                    "function": {
                        "name": "requestTokens",
                        "inputParameters": []
                    }
                }
            }
        ]
    }
}
```

The body of the API prepare request contains the following parameters:

| Parameters |  Definition| 
|---|---|
|type |Request type|
|location| The network to which we submit the transaction|
|urgency| A value that defines how fast a transaction is processed on a network. For Hyperledger Fabric, the value is normal|
|originId| Unique Identifier of the origin/sender|
|destinationId| The name of the token that we invoke a function of|
|name  | [within the function object] This is the function name that Overledger will invoke on the contract|


Now that we have prepared a transaction, we get a confirmation of the prepared request and its assigned requestId. We can execute the transaction by posting the requestId to the execution API.

Execution Request:
POST `https://api.sandbox.overledger.io/v2/execution/transaction`
```
{
    "requestId": "a58e0a1f-2251-407c-8e4b-2bf5efa092ec"
}
```

Execution Response:
```
{
    "requestId": "a58e0a1f-2251-407c-8e4b-2bf5efa092ec",
    "overledgerTransactionId": "0dfef426-40ef-4c8f-85fc-d573ed8830a9",
    "transactionId": "8b93b3662a0a467c08537131e60cac0a8fef0f6335a44cc0568d49f350cd29fd",
    "type": "contract invoke",
    "location": {
        "technology": "Hyperledger Fabric",
        "network": "Quant Sandbox"
    },
    "urgency": "normal",
    "status": {
        "value": "SUCCESSFUL",
        "code": "TXN1002",
        "description": "Transaction successful",
        "message": "Transaction successful",
        "timestamp": "1656937087"
    }
}
```

Please note that this function can only be invoked as long as the account does not hold any Loyalty Point tokens.

### 2. Check Account Balances

Now that we have our Loyalty Point tokens, we can query our account balance to check how many tokens we have received. The requestTokens function has been configured to deliver 100 tokens, and we can verify by calling the Smart Contract Read API.

The only input parameter required is the clientId.

Preparation Request:
POST `https://api.sandbox.overledger.io/v2/preparation/search/smartcontract`
```
{
    "location": {
        "technology": "Hyperledger Fabric",
        "network": "Quant Sandbox"
    },
    "requestDetails": {
        "destination": [
            {
                "smartContract": {
                    "smartContractId": "loyaltypoint",
                    "function": {
                        "name": "balanceOf",
                        "inputParameters": [
                            {
                                "type": "identity",
                                "value": "<your-clientId-here>"
                            }
                        ],
                        "outputParameters": [
                        ]
                    }
                }
            }
        ]
    }
}
```

Preparation Response:
```
{
    "requestId": "adbee021-43b6-431a-9f5a-d2bdc7118bf7",
    "gatewayFee": {
        "amount": "0",
        "unit": "QNT"
    }
}
```
| Parameters |  Definition| 
|---|---|
|location| The network to which we submit the transaction|
|smartContractId| The name of the token that we are making a search request for|
|name | [within the function object] This is the function name that Overledger will read|
|type| It defines the value type suplied in the request|
|value| In this example, it is the identity so that we can get our token balance|

For the execution requestId, we provide the requestId we have received from the smart contract read preparation response.

Execution Request:
POST `https://api.sandbox.overledger.io/v2/execution/search/smartcontract?requestId=adbee021-43b6-431a-9f5a-d2bdc7118bf7`

Execution Response:
```
{
    "location": {
        "technology": "Hyperledger Fabric",
        "network": "Quant Sandbox"
    },
    "smartContract": {
        "smartContractId": "loyaltypoint",
        "function": {
            "functionId": "",
            "name": "balanceOf",
            "inputParameters": [
                {
                    "value": "<your-clientId-here>",
                    "type": "identity"
                }
            ],
            "outputParameters": [
                {
                    "value": "100",
                    "type": "string"
                }
            ]
        }
    }
}
```

As we can see, we got a balance of 100 tokens in the output parameters of the smart contract read response.
We can now proceed to use these Loyalty point tokens to redeem a reward.

### 3. Redeem Flight Rewards

To redeem the rewards (which are represented by QRC-721 tokens), we interact with the QRC-721 contract. In order to do this, we need to change the destinationId,the function we are invoking and the input parameters we are passing in.

- For the destinationId (which is the contract id), we should set `flightreward`
- The function name is `redeemToken`
- The input parameters are:
  - Parameter 1, which should be a stringified random non-negative integer less than 2^256 - this will be the ID assigned to the reward.
  - Parameter 2, which can be any stringified JSON metadata - we have provided examples below

Preparation Request:
POST `https://api.sandbox.overledger.io/v2/preparation/transaction`
```
{
    "type": "Contract Invoke",
    "location": {
        "technology": "Hyperledger Fabric",
        "network": "Quant Sandbox"
    },
    "urgency": "normal",
    "requestDetails": {
        "origin": [
            {
                "originId": "<your-clientId-here>"
            }
        ],
        "destination": [
            {
                "destinationId": "flightreward",
                "smartContract": {
                    "function": {
                        "name": "redeemToken",
                        "inputParameters": [
                           {
                               "type": "string",
                               "value": "12345678"
                           },
                           {
                               "type": "string",
                               "value": "{\"from\":\"London\",\"to\":\"Athens\",\"date\":\"1656938960\"}"
                           }
                        ]
                    }
                }
            }
        ]
    }
}
```

Preparation Response:
```
{
    "requestId": "a4f8ce0c-1be7-4f08-b924-685e48a29694",
    "gatewayFee": {
        "amount": "0",
        "unit": "QNT"
    },
    "dltFee": {
        "amount": "0",
        "unit": "N/A"
    },
    "nativeData": {
        "origin": [
            {
                "originId": "<your-client-id>"
            }
        ],
        "destination": [
            {
                "destinationId": "flightreward",
                "smartContract": {
                    "function": {
                        "name": "redeemToken",
                        "inputParameters": [
                            {
                                "value": "12345678",
                                "type": "string"
                            },
                            {
                                "value": "{\"from\":\"London\",\"to\":\"Athens\",\"date\":\"1656938960\"}",
                                "type": "string"
                            }
                        ]
                    }
                }
            }
        ]
    }
}
```
| Parameters |  Definition| 
|---|---|
|type |Request type|
|location| The network to which we submit the transaction|
|urgency| A value that defines how fast a transaction is processed on a network. For Hyperledger Fabric, the value is normal|
|originId| Unique Identifier of the origin/sender|
|destinationId| The name of the token that we invoke a function of|
|name  | [within the function object] This is the function name that Overledger will invoke on the network|
|type| It defines the value type supplied in the request|
|value| In this example, it is the flight Reward reference and metadata|
Execution Request:
POST `https://api.sandbox.overledger.io/v2/execution/transaction`
```
{
    "requestId": "a4f8ce0c-1be7-4f08-b924-685e48a29694"
}
```

Execution Response:
```
{
    "requestId": "a4f8ce0c-1be7-4f08-b924-685e48a29694",
    "overledgerTransactionId": "d6795a5d-74d8-4583-b452-d90676c6c050",
    "transactionId": "46c7ff69e6eec7039dbd911e67a4cde9cdcf4c93ee1580ed7f49d59540a99918",
    "type": "contract invoke",
    "location": {
        "technology": "Hyperledger Fabric",
        "network": "Quant Sandbox"
    },
    "urgency": "normal",
    "status": {
        "value": "SUCCESSFUL",
        "code": "TXN1002",
        "description": "Transaction successful",
        "message": "Transaction successful",
        "timestamp": "1656939052"
    }
}
```

As the transaction is successful, we can now make the same balance query request we have used before to check the number of Flight Rewards we hold. All we need to do is change the `smartContractId` to the `flightreward` contract instead.

In addition, we can use any of the standard functions from the QRC contracts specification.
