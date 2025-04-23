# Notes

## TX construction

1. Derive address from public key

POST {{URL}}/construction/derive

Body:

{
    "network_identifier": {
        "blockchain": "prime",
        "network": "{{networkId}}"
    },
    "public_key": {
        "hex_bytes": "{{TestAccountPubKeyHexBytes}}",
        "curve_type": "{{curveType}}"
    },
    "metadata": {
        "address_type": "{{address_type}}",
        "staking_credential": {
          "hex_bytes": "{{TestAccountStakePubKeyHex}}",
          "curve_type": "{{curveType}}"
        }
    }
}

2. Get the Sender’s UTXOs
curl -X POST http://localhost:8080/account/coins \
  -H "Content-Type: application/json" \
  -d '{
    "network_identifier": { "blockchain": "prime", "network": "prime-public-testnet" },
    "account_identifier": { "address": "sender_addr_test1..." }
  }'
(Replace sender_addr_test1... with your address.)

2. Prepare the Transaction
bash
Copy
curl -X POST http://localhost:8080/construction/payloads \
  -H "Content-Type: application/json" \
  -d '{
    "network_identifier": { "blockchain": "cardano", "network": "testnet" },
    "operations": [
      {
        "type": "input",
        "coin_change": {
          "coin_identifier": { "identifier": "UTXO_TX_HASH#INDEX" },
          "coin_action": "coin_spent"
        }
      },
      {
        "type": "output",
        "account": { "address": "receiver_addr_test1..." },
        "amount": { "value": "1000000", "currency": { "symbol": "ADA", "decimals": 6 } }
      }
    ]
  }'
Replace:

UTXO_TX_HASH#INDEX → From Step 1’s response.

receiver_addr_test1... → Destination address.

"value": "1000000" → 1 ADA (in lovelace, 1 ADA = 1,000,000 lovelace).

3. Sign the Transaction
Use the signing_payload from Step 2’s response with your payment.skey:

bash
Copy
cardano-cli transaction sign \
  --tx-body-file tx.unsigned \
  --signing-key-file payment.skey \
  --testnet-magic 1 \
  --out-file tx.signed
4. Submit the Transaction
bash
Copy
curl -X POST http://localhost:8080/construction/submit \
  -H "Content-Type: application/json" \
  -d '{
    "network_identifier": { "blockchain": "cardano", "network": "testnet" },
    "signed_transaction": "SIGNED_TX_HEX"
  }'
(Get SIGNED_TX_HEX from tx.signed or Rosetta’s signing response.)

Done!
✅ Transaction sent. Check status with:

bash
Copy
cardano-cli query tx --testnet-magic 1 --tx-id TX_HASH