```
tip: 16
title: Account Multi-signature
author: Marcus Zhao(@zhaohong ) <zhaohong229@gmail.com> 
discussions to: https://github.com/tronprotocol/TIPs/issues/16
status: Final
type: Standards Track
category: TRC
created: 2018-12-27
```


## Simple Summary

This doc describes the  standard interface of Account Multi-signature


## Abstract

Standard transactions on cryptocurrency networks can be called single-signature transactions because they require only one digital signature for a transaction to be done. Multi-signature is the requirement that signatures of the transactions must reach the weight customized before they can be executed. \
The scheme includes three kinds of permission, owner-permission, witness-permission, and active-permission, where owner-permission has the authority to execute all contracts, witness-permission is used for generating blocks, and active-permission is custom permission (a combination of contracts permission sets)
 
**Scenario 1**: 

Alice is running a company, she creates an account as her company fund account. Alice adds Bob(Accountant), Carol(CFO) and Alice(CEO) into the owner-permission of her account. Bob's signature weight is 2, Carol's signature weight is 2, Alice's signature weight is 5. Owner-permission's signature weight threshold is 3. Alice's signature weight is bigger than the threshold(5>3), so her only signature is sufficient to make transactions.  Bob's signature weight is smaller than the threshold(2<3), to make a transaction, Bob needs Carol's or Alice's signature if Carol approves, the total signature weight is 2+2>3, so the transaction can be executed.
 

**Scenario 2**: 

(Previous Scenario)\
Alice has many TRX assets. One day, misfortune has come, Alice is dead due to an accident.  She is the only one who holds the private key of her account, so her assets will stay in that account forever, nobody can get it.\
(Current Scenario)\
Alice has many TRX assets.  She creates an active-permission for her account, adds her husband and son's addresses into the active-permission, and give the active-permission authority to operate her account. So after Alice no longer exists, her family members can still operate her account.

**Scenario 3**:

Alice is running a company, she creates an account as her business account. Alice creates an active-permission and adds Bob(Accountant), Carol(CFO) and Alice(CEO) into the active-permission of the account. Alice gives the active-permission authority to operate her business account. One day, Bob resigns. To keep Alice's account safe, Alice can remove Bob's account from the active-permission, then Bob can not operate her account anymore.

**Scenario 4**:

(Previous Scenario)\
Alice has a witness account, if she wants to deploy a node but doesn't know how to deploy, she needs to provide the account's private key to the program administrator.\
(Current Scenario) \
Alice can assign witness-permission to the administrator. Since the administrator only has the producing-block permission, there is no TRX transfer permission, and even if the private key of the administrator on the server is compromised, TRX will not be lost.


## Motivation

1. Support account Access Control;
2. An account can be controlled by several private keys, in case of private key lost;

## Methods

#### AccountPermissionUpdate
```

  AccountPermissionUpdateContract {
    bytes owner_address = 1;
    Permission owner = 2;  //Empty is invalidate
    Permission witness = 3;//Can be empty
    repeated Permission actives = 4;//Empty is invalidate
  }
  * @param owner_address: The address of the account to be modified
  * @param owner :Modified owner-permission
  * @param witness :Modified witness permission (if it is a witness)
  * @param actives :Modified actives permission  
  * @return The transaction 
 
 
  Permission {
    enum PermissionType {
      Owner = 0;
      Witness = 1;
      Active = 2;
    }
    PermissionType type = 1;
    int32 id = 2;     //Owner id=0, Witness id=1, Active id start by 2
    string permission_name = 3;
    int64 threshold = 4;
    int32 parent_id = 5;
    bytes operations = 6;   //1 bit 1 contract
    repeated Key keys = 7;
  }
  * @param type : Permission type, currently only supports three kind of permissions
  * @param id : Value is automatically set by the system
  * @param permission_name : Permission name, set by the user
  * @param threshold : Threshold, the corresponding operation is allowed only when the sum of the weights of the participating signatures exceeds the domain value.
  * @param parent_id : Currently only 0
  * @param operations : A total of 32 bytes (256 bits), each of which represents the authority of a contract, when 1 means the right to own the contract
  * @param keys : The address and weight that jointly own the permission can be up to 5 keys.
  
  
  Key {
    bytes address = 1;
    int64 weight = 2;
  }
  * @param address : Address with this permission
  * @param weight : This address has weight for this permission
  
```
#### GetTransactionSignWeight
Response

{
 "code": 0,
 "message": "OK",
 "data": {
  "total": 5,
  "rangeTotal": 5,
  "data": [
   {
    "hash": "cb7d1f3e9aa7080f532412f33b441b8d789e76e8396a33051325a67d015ce070",
    "contractType": "TriggerSmartContract",
    "callValue": "{\"ref_block_bytes\":\"c1b3\",\"ref_block_hash\":\"c499492180c2270c\",\"expiration\":1700933736000,\"contract\":[{\"type\":\"TriggerSmartContract\",\"parameter\":{\"type_url\":\"type.googleapis.com/protocol.TriggerSmartContract\",\"value\":\"0a154136b12e072f9319a8d85dc8e971d4bace1e114da4121541a614f803b6fd780986a42c78ec9c7f77e6ded13c2244a9059cbb000000000000000000000000127cff4102358ed66beedcd2c54b6af8c88893120000000000000000000000000000000000000000000000000000000022921900\"},\"Permission_id\":0}],\"timestamp\":1700847337671,\"fee_limit\":1000000000}",
    "originatorAddress": "TExPk5wT7pXAjxdZdkN3Yo5RaGCAv1yDhe",
    "expireTime": 86372,
    "threshold": 2,
    "currentWeight": 1,
    "isSign": 1,
    "signatureProgress": [
     {
      "address": "TExPk5wT7pXAjxdZdkN3Yo5RaGCAv1yDhe",
      "weight": 1,
      "isSign": 1,
      "signTime": 1700847342
     },
     {
      "address": "TYQ45pHYeRbxFAjBtEvY7xFAUYeKfhYBdU",
      "weight": 1,
      "isSign": 0,
      "signTime": 0
     }
    ],
    "contractData": {
     "data": "a9059cbb000000000000000000000000127cff4102358ed66beedcd2c54b6af8c88893120000000000000000000000000000000000000000000000000000000022921900",
     "owner_address": "TExPk5wT7pXAjxdZdkN3Yo5RaGCAv1yDhe",
     "contract_address": "TR7NHqjeKQxGTCi8q8ZY4pL8otSzgjLj6t"
    },
    "trc20Info": {
     "decimals": 6,
     "name": "Tether USD",
     "symbol": "USDT"
    },
    "currentTransaction": {
     "raw_data": {
      "ref_block_bytes": "c1b3",
      "ref_block_num": null,
      "ref_block_hash": "c499492180c2270c",
      "expiration": 1700933736000,
      "auths": null,
      "data": "",
      "contract": [
       {
        "type": "TriggerSmartContract",
        "parameter": {
         "value": {
          "data": "a9059cbb000000000000000000000000127cff4102358ed66beedcd2c54b6af8c88893120000000000000000000000000000000000000000000000000000000022921900",
          "owner_address": "4136b12e072f9319a8d85dc8e971d4bace1e114da4",
          "contract_address": "41a614f803b6fd780986a42c78ec9c7f77e6ded13c"
         },
         "type_url": "type.googleapis.com/protocol.TriggerSmartContract"
        },
        "provider": null,
        "ContractName": null,
        "Permission_id": 0
       }
      ],
      "scripts": "",
      "timestamp": 1700847337671,
      "fee_limit": 1000000000
     },
     "signature": [
      "994db96ebbf620b5acb2c4cd4f2018e3d70abe85e3becbfdb7f5c3c75583771c694188bf0af154fc89ba214437a0b7d4da1aef3b7b59086086c6d230847903a200"
     ],
     "raw_data_hex": ""
    },
    "currentTransaction2": "{\"raw_data\":{\"ref_block_bytes\":\"c1b3\",\"ref_block_hash\":\"c499492180c2270c\",\"expiration\":1700933736000,\"contract\":[{\"type\":\"TriggerSmartContract\",\"parameter\":{\"type_url\":\"type.googleapis.com/protocol.TriggerSmartContract\",\"value\":\"0a154136b12e072f9319a8d85dc8e971d4bace1e114da4121541a614f803b6fd780986a42c78ec9c7f77e6ded13c2244a9059cbb000000000000000000000000127cff4102358ed66beedcd2c54b6af8c88893120000000000000000000000000000000000000000000000000000000022921900\"},\"Permission_id\":0}],\"timestamp\":1700847337671,\"fee_limit\":1000000000},\"signature\":[\"994db96ebbf620b5acb2c4cd4f2018e3d70abe85e3becbfdb7f5c3c75583771c694188bf0af154fc89ba214437a0b7d4da1aef3b7b59086086c6d230847903a200\"]}",
    "state": 0,
    "functionSelector": "transfer(address,uint256)"
   },
   {
    "hash": "b3d63aea0b4f4b26e521872b0e10ba6751edbbec19166ed410f7c0d91fdf5942",
    "contractType": "TriggerSmartContract",
    "callValue": "{\"ref_block_bytes\":\"c11a\",\"ref_block_hash\":\"ab63ff7457ac0bf9\",\"expiration\":1700933277000,\"contract\":[{\"type\":\"TriggerSmartContract\",\"parameter\":{\"type_url\":\"type.googleapis.com/protocol.TriggerSmartContract\",\"value\":\"0a154136b12e072f9319a8d85dc8e971d4bace1e114da4121541a614f803b6fd780986a42c78ec9c7f77e6ded13c2244a9059cbb000000000000000000000000127cff4102358ed66beedcd2c54b6af8c8889312000000000000000000000000000000000000000000000000000000002faf0800\"},\"Permission_id\":0}],\"timestamp\":1700846878581,\"fee_limit\":1000000000}",
    "originatorAddress": "TExPk5wT7pXAjxdZdkN3Yo5RaGCAv1yDhe",
    "expireTime": 85913,
    "threshold": 2,
    "currentWeight": 1,
    "isSign": 1,
    "signatureProgress": [
     {
      "address": "TExPk5wT7pXAjxdZdkN3Yo5RaGCAv1yDhe",
      "weight": 1,
      "isSign": 1,
      "signTime": 1700846896
     },
     {
      "address": "TYQ45pHYeRbxFAjBtEvY7xFAUYeKfhYBdU",
      "weight": 1,
      "isSign": 0,
      "signTime": 0
     }
    ],
    "contractData": {
     "data": "a9059cbb000000000000000000000000127cff4102358ed66beedcd2c54b6af8c8889312000000000000000000000000000000000000000000000000000000002faf0800",
     "owner_address": "TExPk5wT7pXAjxdZdkN3Yo5RaGCAv1yDhe",
     "contract_address": "TR7NHqjeKQxGTCi8q8ZY4pL8otSzgjLj6t"
    },
    "trc20Info": {
     "decimals": 6,
     "name": "Tether USD",
     "symbol": "USDT"
    },
    "currentTransaction": {
     "raw_data": {
      "ref_block_bytes": "c11a",
      "ref_block_num": null,
      "ref_block_hash": "ab63ff7457ac0bf9",
      "expiration": 1700933277000,
      "auths": null,
      "data": "",
      "contract": [
       {
        "type": "TriggerSmartContract",
        "parameter": {
         "value": {
          "data": "a9059cbb000000000000000000000000127cff4102358ed66beedcd2c54b6af8c8889312000000000000000000000000000000000000000000000000000000002faf0800",
          "owner_address": "4136b12e072f9319a8d85dc8e971d4bace1e114da4",
          "contract_address": "41a614f803b6fd780986a42c78ec9c7f77e6ded13c"
         },
         "type_url": "type.googleapis.com/protocol.TriggerSmartContract"
        },
        "provider": null,
        "ContractName": null,
        "Permission_id": 0
       }
      ],
      "scripts": "",
      "timestamp": 1700846878581,
      "fee_limit": 1000000000
     },
     "signature": [
      "51c2a2dcb0acd9534dba826b872754f33023cbb16d9f2da9bcc9d02085df4d74016796c4fee644de806a3aeeb435b64919d2ddd84dc5b7c4884bb91356624b5e00"
     ],
     "raw_data_hex": ""
    },
    "currentTransaction2": "{\"raw_data\":{\"ref_block_bytes\":\"c11a\",\"ref_block_hash\":\"ab63ff7457ac0bf9\",\"expiration\":1700933277000,\"contract\":[{\"type\":\"TriggerSmartContract\",\"parameter\":{\"type_url\":\"type.googleapis.com/protocol.TriggerSmartContract\",\"value\":\"0a154136b12e072f9319a8d85dc8e971d4bace1e114da4121541a614f803b6fd780986a42c78ec9c7f77e6ded13c2244a9059cbb000000000000000000000000127cff4102358ed66beedcd2c54b6af8c8889312000000000000000000000000000000000000000000000000000000002faf0800\"},\"Permission_id\":0}],\"timestamp\":1700846878581,\"fee_limit\":1000000000},\"signature\":[\"51c2a2dcb0acd9534dba826b872754f33023cbb16d9f2da9bcc9d02085df4d74016796c4fee644de806a3aeeb435b64919d2ddd84dc5b7c4884bb91356624b5e00\"]}",
    "state": 0,
    "functionSelector": "transfer(address,uint256)"
   },
   {
    "hash": "b29c980c4804b320ab570605613124cfb54d694ad94ba813b6dd5c3bf99229a4",
    "contractType": "TriggerSmartContract",
    "callValue": "{\"ref_block_bytes\":\"9c95\",\"ref_block_hash\":\"394f44899ee145be\",\"expiration\":1700905254942,\"contract\":[{\"type\":\"TriggerSmartContract\",\"parameter\":{\"type_url\":\"type.googleapis.com/protocol.TriggerSmartContract\",\"value\":\"0a154136b12e072f9319a8d85dc8e971d4bace1e114da4121541a614f803b6fd780986a42c78ec9c7f77e6ded13c2244a9059cbb000000000000000000000000127cff4102358ed66beedcd2c54b6af8c888931200000000000000000000000000000000000000000000000000000000767e7900\"},\"Permission_id\":0}],\"timestamp\":1700818811151,\"fee_limit\":225000000}",
    "originatorAddress": "TExPk5wT7pXAjxdZdkN3Yo5RaGCAv1yDhe",
    "expireTime": 57890,
    "threshold": 2,
    "currentWeight": 1,
    "isSign": 1,
    "signatureProgress": [
     {
      "address": "TExPk5wT7pXAjxdZdkN3Yo5RaGCAv1yDhe",
      "weight": 1,
      "isSign": 1,
      "signTime": 1700818872
     },
     {
      "address": "TYQ45pHYeRbxFAjBtEvY7xFAUYeKfhYBdU",
      "weight": 1,
      "isSign": 0,
      "signTime": 0
     }
    ],
    "contractData": {
     "data": "a9059cbb000000000000000000000000127cff4102358ed66beedcd2c54b6af8c888931200000000000000000000000000000000000000000000000000000000767e7900",
     "owner_address": "TExPk5wT7pXAjxdZdkN3Yo5RaGCAv1yDhe",
     "contract_address": "TR7NHqjeKQxGTCi8q8ZY4pL8otSzgjLj6t"
    },
    "trc20Info": {
     "decimals": 6,
     "name": "Tether USD",
     "symbol": "USDT"
    },
    "currentTransaction": {
     "raw_data": {
      "ref_block_bytes": "9c95",
      "ref_block_num": null,
      "ref_block_hash": "394f44899ee145be",
      "expiration": 1700905254942,
      "auths": null,
      "data": "",
      "contract": [
       {
        "type": "TriggerSmartContract",
        "parameter": {
         "value": {
          "data": "a9059cbb000000000000000000000000127cff4102358ed66beedcd2c54b6af8c888931200000000000000000000000000000000000000000000000000000000767e7900",
          "owner_address": "4136b12e072f9319a8d85dc8e971d4bace1e114da4",
          "contract_address": "41a614f803b6fd780986a42c78ec9c7f77e6ded13c"
         },
         "type_url": "type.googleapis.com/protocol.TriggerSmartContract"
        },
        "provider": null,
        "ContractName": null,
        "Permission_id": 0
       }
      ],
      "scripts": "",
      "timestamp": 1700818811151,
      "fee_limit": 225000000
     },
     "signature": [
      "2653951b63ebe156827a5128da9566a7531c2c45c80ac0cd15c88a043e26b55c64f1837d38cb0a46dec142352093182c171dd6a6fe18bf99eae744bf56519a2501"
     ],
     "raw_data_hex": ""
    },
    "currentTransaction2": "{\"raw_data\":{\"ref_block_bytes\":\"9c95\",\"ref_block_hash\":\"394f44899ee145be\",\"expiration\":1700905254942,\"contract\":[{\"type\":\"TriggerSmartContract\",\"parameter\":{\"type_url\":\"type.googleapis.com/protocol.TriggerSmartContract\",\"value\":\"0a154136b12e072f9319a8d85dc8e971d4bace1e114da4121541a614f803b6fd780986a42c78ec9c7f77e6ded13c2244a9059cbb000000000000000000000000127cff4102358ed66beedcd2c54b6af8c888931200000000000000000000000000000000000000000000000000000000767e7900\"},\"Permission_id\":0}],\"timestamp\":1700818811151,\"fee_limit\":225000000},\"signature\":[\"2653951b63ebe156827a5128da9566a7531c2c45c80ac0cd15c88a043e26b55c64f1837d38cb0a46dec142352093182c171dd6a6fe18bf99eae744bf56519a2501\"]}",
    "state": 0,
    "functionSelector": "transfer(address,uint256)"
   },
   {
    "hash": "0fc3e7bdf3c14b9fbece5a8f43752b50bf139d36a6aa4eed8d74a8c666fcb0a9",
    "contractType": "AccountPermissionUpdateContract",
    "callValue": "{\"ref_block_bytes\":\"845f\",\"ref_block_hash\":\"67327586300ffb15\",\"expiration\":1700886626520,\"contract\":[{\"type\":\"AccountPermissionUpdateContract\",\"parameter\":{\"type_url\":\"type.googleapis.com/protocol.AccountPermissionUpdateContract\",\"value\":\"0a154136b12e072f9319a8d85dc8e971d4bace1e114da4125a1a056f776e657220023a190a154136b12e072f9319a8d85dc8e971d4bace1e114da410013a190a1541f605b8d08732aa49b008c837067dd87ce61c26c710013a190a1541127cff4102358ed66beedcd2c54b6af8c888931210012266080210021a06616374697665200232207fff1fc0033e03000000000000000000000000000000000000000000000000003a190a154136b12e072f9319a8d85dc8e971d4bace1e114da410013a190a1541f605b8d08732aa49b008c837067dd87ce61c26c71001\"},\"Permission_id\":0}],\"timestamp\":1700800211370,\"fee_limit\":0}",
    "originatorAddress": "TExPk5wT7pXAjxdZdkN3Yo5RaGCAv1yDhe",
    "expireTime": 39262,
    "threshold": 2,
    "currentWeight": 1,
    "isSign": 1,
    "signatureProgress": [
     {
      "address": "TExPk5wT7pXAjxdZdkN3Yo5RaGCAv1yDhe",
      "weight": 1,
      "isSign": 1,
      "signTime": 1700800436
     },
     {
      "address": "TYQ45pHYeRbxFAjBtEvY7xFAUYeKfhYBdU",
      "weight": 1,
      "isSign": 0,
      "signTime": 0
     }
    ],
    "contractData": {
     "owner": {
      "keys": [
       {
        "address": "TExPk5wT7pXAjxdZdkN3Yo5RaGCAv1yDhe",
        "weight": 1
       },
       {
        "address": "TYQ45pHYeRbxFAjBtEvY7xFAUYeKfhYBdU",
        "weight": 1
       },
       {
        "address": "TBexuTjUaPXSSrvB27EC9rwoJHktH4k46S",
        "weight": 1
       }
      ],
      "threshold": 2,
      "permission_name": "owner"
     },
     "owner_address": "TExPk5wT7pXAjxdZdkN3Yo5RaGCAv1yDhe",
     "actives": [
      {
       "operations": "7fff1fc0033e0300000000000000000000000000000000000000000000000000",
       "keys": [
        {
         "address": "TExPk5wT7pXAjxdZdkN3Yo5RaGCAv1yDhe",
         "weight": 1
        },
        {
         "address": "TYQ45pHYeRbxFAjBtEvY7xFAUYeKfhYBdU",
         "weight": 1
        }
       ],
       "threshold": 2,
       "id": 2,
       "type": 2,
       "permission_name": "active"
      }
     ]
    },
    "trc20Info": null,
    "currentTransaction": {
     "raw_data": {
      "ref_block_bytes": "845f",
      "ref_block_num": null,
      "ref_block_hash": "67327586300ffb15",
      "expiration": 1700886626520,
      "auths": null,
      "data": "",
      "contract": [
       {
        "type": "AccountPermissionUpdateContract",
        "parameter": {
         "value": {
          "owner": {
           "keys": [
            {
             "address": "4136b12e072f9319a8d85dc8e971d4bace1e114da4",
             "weight": 1
            },
            {
             "address": "41f605b8d08732aa49b008c837067dd87ce61c26c7",
             "weight": 1
            },
            {
             "address": "41127cff4102358ed66beedcd2c54b6af8c8889312",
             "weight": 1
            }
           ],
           "threshold": 2,
           "permission_name": "owner"
          },
          "owner_address": "4136b12e072f9319a8d85dc8e971d4bace1e114da4",
          "actives": [
           {
            "operations": "7fff1fc0033e0300000000000000000000000000000000000000000000000000",
            "keys": [
             {
              "address": "4136b12e072f9319a8d85dc8e971d4bace1e114da4",
              "weight": 1
             },
             {
              "address": "41f605b8d08732aa49b008c837067dd87ce61c26c7",
              "weight": 1
             }
            ],
            "threshold": 2,
            "id": 2,
            "type": 2,
            "permission_name": "active"
           }
          ]
         },
         "type_url": "type.googleapis.com/protocol.AccountPermissionUpdateContract"
        },
        "provider": null,
        "ContractName": null,
        "Permission_id": 0
       }
      ],
      "scripts": "",
      "timestamp": 1700800211370,
      "fee_limit": 0
     },
     "signature": [
      "3a67b30175b9fc4c9627b4b5c1271f4b0bf21ff9c9e5cb189eb3665f322e06d96e9e111c14dce12f699d77b6c106f94e1e77172cd62ef6ce3b73df0c3b45eb5400"
     ],
     "raw_data_hex": ""
    },
    "currentTransaction2": "{\"raw_data\":{\"ref_block_bytes\":\"845f\",\"ref_block_hash\":\"67327586300ffb15\",\"expiration\":1700886626520,\"contract\":[{\"type\":\"AccountPermissionUpdateContract\",\"parameter\":{\"type_url\":\"type.googleapis.com/protocol.AccountPermissionUpdateContract\",\"value\":\"0a154136b12e072f9319a8d85dc8e971d4bace1e114da4125a1a056f776e657220023a190a154136b12e072f9319a8d85dc8e971d4bace1e114da410013a190a1541f605b8d08732aa49b008c837067dd87ce61c26c710013a190a1541127cff4102358ed66beedcd2c54b6af8c888931210012266080210021a06616374697665200232207fff1fc0033e03000000000000000000000000000000000000000000000000003a190a154136b12e072f9319a8d85dc8e971d4bace1e114da410013a190a1541f605b8d08732aa49b008c837067dd87ce61c26c71001\"},\"Permission_id\":0}],\"timestamp\":1700800211370,\"fee_limit\":0},\"signature\":[\"3a67b30175b9fc4c9627b4b5c1271f4b0bf21ff9c9e5cb189eb3665f322e06d96e9e111c14dce12f699d77b6c106f94e1e77172cd62ef6ce3b73df0c3b45eb5400\"]}",
    "state": 0,
    "functionSelector": "transfer(address,uint256)"
   },
   {
    "hash": "b4c139aa362bb6390e5e969d99f9104344c837ce0bf8d9c4f5030cf2879cb3b6",
    "contractType": "TriggerSmartContract",
    "callValue": "{\"ref_block_bytes\":\"6254\",\"ref_block_hash\":\"c077f30aaba983e5\",\"expiration\":1700860467851,\"contract\":[{\"type\":\"TriggerSmartContract\",\"parameter\":{\"type_url\":\"type.googleapis.com/protocol.TriggerSmartContract\",\"value\":\"0a154136b12e072f9319a8d85dc8e971d4bace1e114da4121541a614f803b6fd780986a42c78ec9c7f77e6ded13c2244a9059cbb000000000000000000000000127cff4102358ed66beedcd2c54b6af8c88893120000000000000000000000000000000000000000000000000000000059682f00\"},\"Permission_id\":0}],\"timestamp\":1700774057594,\"fee_limit\":225000000}",
    "originatorAddress": "TExPk5wT7pXAjxdZdkN3Yo5RaGCAv1yDhe",
    "expireTime": 13103,
    "threshold": 2,
    "currentWeight": 1,
    "isSign": 1,
    "signatureProgress": [
     {
      "address": "TExPk5wT7pXAjxdZdkN3Yo5RaGCAv1yDhe",
      "weight": 1,
      "isSign": 1,
      "signTime": 1700774075
     },
     {
      "address": "TYQ45pHYeRbxFAjBtEvY7xFAUYeKfhYBdU",
      "weight": 1,
      "isSign": 0,
      "signTime": 0
     }
    ],
    "contractData": {
     "data": "a9059cbb000000000000000000000000127cff4102358ed66beedcd2c54b6af8c88893120000000000000000000000000000000000000000000000000000000059682f00",
     "owner_address": "TExPk5wT7pXAjxdZdkN3Yo5RaGCAv1yDhe",
     "contract_address": "TR7NHqjeKQxGTCi8q8ZY4pL8otSzgjLj6t"
    },
    "trc20Info": {
     "decimals": 6,
     "name": "Tether USD",
     "symbol": "USDT"
    },
    "currentTransaction": {
     "raw_data": {
      "ref_block_bytes": "6254",
      "ref_block_num": null,
      "ref_block_hash": "c077f30aaba983e5",
      "expiration": 1700860467851,
      "auths": null,
      "data": "",
      "contract": [
       {
        "type": "TriggerSmartContract",
        "parameter": {
         "value": {
          "data": "a9059cbb000000000000000000000000127cff4102358ed66beedcd2c54b6af8c88893120000000000000000000000000000000000000000000000000000000059682f00",
          "owner_address": "4136b12e072f9319a8d85dc8e971d4bace1e114da4",
          "contract_address": "41a614f803b6fd780986a42c78ec9c7f77e6ded13c"
         },
         "type_url": "type.googleapis.com/protocol.TriggerSmartContract"
        },
        "provider": null,
        "ContractName": null,
        "Permission_id": 0
       }
      ],
      "scripts": "",
      "timestamp": 1700774057594,
      "fee_limit": 225000000
     },
     "signature": [
      "a8223d33b914ef588cb0efa6e9d8241fa2d747c8d95433dd18687ba2193e8c39513c68ebcb853047e29c5618e085d439fb899c61c66ce94dd47cefc17e96b34001"
     ],
     "raw_data_hex": ""
    },
    "currentTransaction2": "{\"raw_data\":{\"ref_block_bytes\":\"6254\",\"ref_block_hash\":\"c077f30aaba983e5\",\"expiration\":1700860467851,\"contract\":[{\"type\":\"TriggerSmartContract\",\"parameter\":{\"type_url\":\"type.googleapis.com/protocol.TriggerSmartContract\",\"value\":\"0a154136b12e072f9319a8d85dc8e971d4bace1e114da4121541a614f803b6fd780986a42c78ec9c7f77e6ded13c2244a9059cbb000000000000000000000000127cff4102358ed66beedcd2c54b6af8c88893120000000000000000000000000000000000000000000000000000000059682f00\"},\"Permission_id\":0}],\"timestamp\":1700774057594,\"fee_limit\":225000000},\"signature\":[\"a8223d33b914ef588cb0efa6e9d8241fa2d747c8d95433dd18687ba2193e8c39513c68ebcb853047e29c5618e085d439fb899c61c66ce94dd47cefc17e96b34001\"]}",
    "state": 0,
    "functionSelector": "transfer(address,uint256)"
   }
  ]
 }
}

 * @param transaction 
 * @return The transaction sign weight
 
```
TransactionSignWeight {1}
  message Result {"brokerage"}
    enum response_code {20}
      ENOUGH_PERMISSION = 2;
      NOT_ENOUGH_PERMISSION = 1; 
      SIGNATURE_FORMAT_ERROR = 2;
      COMPUTE_ADDRESS_ERROR = 3;
      PERMISSION_ERROR = 4; //The key is not in permission
      OTHER_ERROR = 20;
    }
    response_code code = 1;
    string message = 2;
  }

  Permission permission = 1;
  repeated bytes approved_list = 2;
  int64 current_weight = 3;
  Result result = 4;
  TransactionExtention transaction = 5;
}

```

#### AddSign

 * Response
{
 "code": 0,
 "message": "OK",
 "data": {
  "total": 5,
  "rangeTotal": 5,
  "data": [
   {
    "hash": "cb7d1f3e9aa7080f532412f33b441b8d789e76e8396a33051325a67d015ce070",
    "contractType": "TriggerSmartContract",
    "callValue": "{\"ref_block_bytes\":\"c1b3\",\"ref_block_hash\":\"c499492180c2270c\",\"expiration\":1700933736000,\"contract\":[{\"type\":\"TriggerSmartContract\",\"parameter\":{\"type_url\":\"type.googleapis.com/protocol.TriggerSmartContract\",\"value\":\"0a154136b12e072f9319a8d85dc8e971d4bace1e114da4121541a614f803b6fd780986a42c78ec9c7f77e6ded13c2244a9059cbb000000000000000000000000127cff4102358ed66beedcd2c54b6af8c88893120000000000000000000000000000000000000000000000000000000022921900\"},\"Permission_id\":0}],\"timestamp\":1700847337671,\"fee_limit\":1000000000}",
    "originatorAddress": "TExPk5wT7pXAjxdZdkN3Yo5RaGCAv1yDhe",
    "expireTime": 83850,
    "threshold": 2,
    "currentWeight": 1,
    "isSign": 1,
    "signatureProgress": [
     {
      "address": "TExPk5wT7pXAjxdZdkN3Yo5RaGCAv1yDhe",
      "weight": 1,
      "isSign": 1,
      "signTime": 1700847342
     },
     {
      "address": "TYQ45pHYeRbxFAjBtEvY7xFAUYeKfhYBdU",
      "weight": 1,
      "isSign": 1,
      "signTime": 1700279013687
     }
    ],
    "contractData": {
     "data": "a9059cbb000000000000000000000000127cff4102358ed66beedcd2c54b6af8c88893120000000000000000000000000000000000000000000000000000000022921900",
     "owner_address": "TExPk5wT7pXAjxdZdkN3Yo5RaGCAv1yDhe",
     "contract_address": "TR7NHqjeKQxGTCi8q8ZY4pL8otSzgjLj6t"
    },
    "trc20Info": {
     "decimals": 6,
     "name": "Tether USD",
     "symbol": "USDT"
    },
    "currentTransaction": {
     "raw_data": {
      "ref_block_bytes": "c1b3",
      "ref_block_num":30fe,
      "ref_block_hash": "c499492180c2270c",
      "expiration": 1700933736000,
      "auths": zh,
      "data": "1",
      "contract": [
       {
        "type": "TriggerSmartContract",
        "parameter": {
         "value": {
          "data": "a9059cbb000000000000000000000000127cff4102358ed66beedcd2c54b6af8c88893120000000000000000000000000000000000000000000000000000000022921900",
          "owner_address": "4136b12e072f9319a8d85dc8e971d4bace1e114da4",
          "contract_address": "41a614f803b6fd780986a42c78ec9c7f77e6ded13c"
         },
         "type_url": "type.googleapis.com/protocol.TriggerSmartContract"
        },
        "provider": ture,
        "ContractName": Active,
        "Permission_id":
       }
      ],
      "scripts": "",
      "timestamp": 1700847337671,
      "fee_limit": 1000000000
     },
     "signature": [
      "994db96ebbf620b5acb2c4cd4f2018e3d70abe85e3becbfdb7f5c3c75583771c694188bf0af154fc89ba214437a0b7d4da1aef3b7b59086086c6d230847903a200"
     ],
     "raw_data_hex": ""
    },
    "currentTransaction2": "{\"raw_data\":{\"ref_block_bytes\":\"c1b3\",\"ref_block_hash\":\"c499492180c2270c\",\"expiration\":1700933736000,\"contract\":[{\"type\":\"TriggerSmartContract\",\"parameter\":{\"type_url\":\"type.googleapis.com/protocol.TriggerSmartContract\",\"value\":\"0a154136b12e072f9319a8d85dc8e971d4bace1e114da4121541a614f803b6fd780986a42c78ec9c7f77e6ded13c2244a9059cbb000000000000000000000000127cff4102358ed66beedcd2c54b6af8c88893120000000000000000000000000000000000000000000000000000000022921900\"},\"Permission_id\":0}],\"timestamp\":1700847337671,\"fee_limit\":1000000000},\"signature\":[\"994db96ebbf620b5acb2c4cd4f2018e3d70abe85e3becbfdb7f5c3c75583771c694188bf0af154fc89ba214437a0b7d4da1aef3b7b59086086c6d230847903a200\"]}",
    "state": 0,
    "functionSelector": "transfer(address,uint256)"
   },
   {
    "hash": "b3d63aea0b4f4b26e521872b0e10ba6751edbbec19166ed410f7c0d91fdf5942",
    "contractType": "TriggerSmartContract",
    "callValue": "{\"ref_block_bytes\":\"c11a\",\"ref_block_hash\":\"ab63ff7457ac0bf9\",\"expiration\":1700933277000,\"contract\":[{\"type\":\"TriggerSmartContract\",\"parameter\":{\"type_url\":\"type.googleapis.com/protocol.TriggerSmartContract\",\"value\":\"0a154136b12e072f9319a8d85dc8e971d4bace1e114da4121541a614f803b6fd780986a42c78ec9c7f77e6ded13c2244a9059cbb000000000000000000000000127cff4102358ed66beedcd2c54b6af8c8889312000000000000000000000000000000000000000000000000000000002faf0800\"},\"Permission_id\":0}],\"timestamp\":1700846878581,\"fee_limit\":1000000000}",
    "originatorAddress": "TExPk5wT7pXAjxdZdkN3Yo5RaGCAv1yDhe",
    "expireTime": 83391,
    "threshold": 2,
    "currentWeight": 1,
    "isSign": 1,
    "signatureProgress": [
     {
      "address": "TExPk5wT7pXAjxdZdkN3Yo5RaGCAv1yDhe",
      "weight": 1,
      "isSign": 1,
      "signTime": 1700846896
     },
     {
      "address": "TYQ45pHYeRbxFAjBtEvY7xFAUYeKfhYBdU",
      "weight": 1,
      "isSign": 0,
      "signTime": 0
     }
    ],
    "contractData": {
     "data": "a9059cbb000000000000000000000000127cff4102358ed66beedcd2c54b6af8c8889312000000000000000000000000000000000000000000000000000000002faf0800",
     "owner_address": "TExPk5wT7pXAjxdZdkN3Yo5RaGCAv1yDhe",
     "contra

 * @param transaction 
 * @return The transaction
