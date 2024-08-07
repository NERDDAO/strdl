```yaml
# 1. Edit this file with this https://www.serverless.com/framework/docs/providers/aws/guide/serverless.yml
# 2. Set up credentials with this http://slss.io/aws-creds-setup and deploy with the command
# $export AWS_ACCESS_KEY_ID=<your-key-here>
# $export AWS_SECRET_ACCESS_KEY=<your-secret-key-here>
# $yarn deploy:serverless

service: strudel-finance

useDotenv: true
variablesResolutionMode: 20210326
plugins:
  - serverless-apigw-binary
  - serverless-offline
  - serverless-dotenv-plugin

custom:
  serverless-offline:
    port: 3001
  apigwBinary:
    types:
      - 'application/bitcoin-paymentrequest'
      - 'application/bitcoincash-paymentrequest'
      - 'application/bitcoin-paymentack'
      - 'application/bitcoincash-paymentack'
      - 'application/bitcoin-payment'
      - 'application/bitcoincash-payment'

  table: strudel-${opt:stage}
  
  region: eu-west-1
  bcdRpc: 'bcd.strudel.finance'
  bcdPort: 8332
  providerUrl: https://mainnet.infura.io/v3/9487317cfa5f4c30834b75c0a975dd85
  bscProviderUrl: https://bsc-dataseed1.binance.org/
  harmonyProviderUrl: https://api.harmony.one


package:
  exclude:
    - src/**/*.test.js

provider:
  name: aws
  region: ${self:custom.region}
  # When you update this, you should update Node version here 
  runtime: nodejs14.x 
  timeout: 30
  lambdaHashingVersion: 20201221
  apiGateway:
    binaryMediaTypes:
      - 'application/bitcoin-paymentrequest'
      - 'application/bitcoincash-paymentrequest'
      - 'application/bitcoin-paymentack'
      - 'application/bitcoincash-paymentack'
  iamRoleStatements:
    - Effect: Allow
      Action:
       - sdb:GetAttributes
       - sdb:PutAttributes
       - sdb:Select
       - sdb:PutAttributes
      Resource: 
        - Fn::Join:
          - ""
          - - "arn:aws:sdb:*:*:domain/"
            - Ref: strudelDb

responseMappings: &response_mappings
  headers:
    Content-Type: "'application/json'"
  statusCodes:
      200:
          pattern: ''
          headers:
            Access-Control-Allow-Origin: "'*'"
          template:
            application/json: ""
      204:
          pattern: 'Created'
          headers:
            Access-Control-Allow-Origin: "'*'"
          template:
            application/json: ""
      400:
          pattern: 'Bad Request: .*'
          headers:
            Access-Control-Allow-Origin: "'*'"
          template:
            application/json: ""
      404:
          pattern: 'Not Found: .*'
          headers:
            Access-Control-Allow-Origin: "'*'"
          template:
            application/json: ""
      408:
          pattern: 'Not Found: .*'
          headers:
            Access-Control-Allow-Origin: "'*'"
          template:
            application/json: ""
      500:
          pattern: 'Error: .*'
          headers:
            Access-Control-Allow-Origin: "'*'"

functions:
  bridge-service:
    timeout: 30
    handler: src/strudel.handler
    environment:
      PROVIDER_URL: ${env:PROVIDER_URL, self:custom.providerUrl}
      BSC_PROVIDER_URL: ${env:BSC_PROVIDER_URL, self:custom.bscProviderUrl}
      HARMONY_PROVIDER_URL: ${env:HARMONY_PROVIDER_URL, self:custom.harmonyProviderUrl}
      BCD_RPC: ${env:BCD_RPC, self:custom.bcdRpc}
      BCD_PORT: ${env:BCD_PORT, self:custom.bcdPort}
      BCD_USERNAME: ${env:BCD_USERNAME}
      BCD_PASSWORD: ${env:BCD_PASSWORD}
      TABLE_NAME:
        Ref: strudelDb
    events:
      - http:
          method: get
          path: /account/{account}
          integration: lambda
          cors: true
          response: *response_mappings
      - http:
          method: post
          path: /account/{account}/addSig
          integration: lambda
          cors: true
          response: *response_mappings
      - http:
          method: post
          path: /payment/{txHash}/{blockchain} 
          integration: lambda
          cors: true
          response: *response_mappings
      - http:
          method: post
          path: /bch/payment/{txHash}
          integration: lambda
          cors: true
          response: *response_mappings
      - http:
          method: post
          path: /payment/{txHash}/output/{outputIndex}/addEthTx
          integration: lambda
          cors: true
          response: *response_mappings
      - http:
          method: post
          path: /bch/payment/{txHash}/output/{outputIndex}/addEthTx
          integration: lambda
          cors: true
          response: *response_mappings
      - http:
          method: get
          path: /syn/{destination}/{amount}
          integration: lambda
          request:
            headers:
              Content-Type: "'application/bitcoin-payment'"
          response:
            headers:
              Content-Type: "'application/bitcoin-paymentrequest'"
      - http:
          method: get
          path: /bch/syn/{destination}/{amount}
          integration: lambda
          request:
            headers:
              Content-Type: "'application/bitcoincash-payment'"
          response:
            headers:
              Content-Type: "'application/bitcoincash-paymentrequest'"
      - http:
          method: post
          path: /ack
          integration: lambda
          request:
            headers:
              Content-Type: "'application/bitcoin-payment'"
          response:
            headers:
              Content-Type: "'application/bitcoin-paymentack'"
      - http:
          method: post
          path: /bch/ack
          integration: lambda
          request:
            headers:
              Content-Type: "'application/bitcoincash-payment'"
          response:
            headers:
              Content-Type: "'application/bitcoincash-paymentack'"
      - http:
          method: get
          path: /watchlist
          integration: lambda
          cors: true
          response: *response_mappings
      - http:
          method: post
          path: /proof/{txHash}/{blockHash}
          integration: lambda
          cors: true
          response: *response_mappings
      - http:
          method: post
          path: /bch/proof/{txHash}/{blockHash}
          integration: lambda
          cors: true
          response: *response_mappings

resources:
  Resources:
    strudelDb:
      Type: "AWS::SDB::Domain"
      Properties:
        DomainName : ${self:custom.table}
        Description: "SDB Domain to store metadata"
```