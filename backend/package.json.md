```json
{
  "name": "backend",
  "version": "0.1.0",
  "description": "api and db to support ui functionality",
  "main": "index.js",
  "scripts": {
    "lint": "eslint --fix src/**/*.js",
    "test": "mocha --recursive 'src/**/*.test.js'",
    "start": "AWS_SDK_LOAD_CONFIG=1 npx serverless offline start -s dev",
    "deploy:serverless": "npx serverless deploy -s production",
    "deploy:sls": "npx sls deploy -s production"
  },
  "repository": {
    "type": "git",
    "url": "git@github.com:strudel-finance/monorepo.git"
  },
  "author": "Strudel",
  "license": "MPL-2.0",
  "devDependencies": {
    "chai": "latest",
    "eslint": "^5.13.0",
    "eth-sig-util": "^2.5.3",
    "mocha": "latest",
    "serverless-offline": "latest",
    "sinon": "^9.0.2",
    "sinon-chai": "^3.5.0"
  },
  "dependencies": {
    "@summa-tx/bitcoin-spv-js": "^4.0.2",
    "bitcoinjs-lib": "^5.2.0",
    "bitcore-lib": "^8.23.1",
    "bitcore-payment-protocol": "^8.1.1",
    "ethers": "^4.0.47",
    "jsbi-utils": "^1.0.1",
    "leap-lambda-boilerplate": "^1.3.0",
    "node-fetch": "^2.3.0",
    "pusher": "^3.0.1",
    "request": "^2.88.2",
    "serverless-apigw-binary": "^0.4.4",
    "serverless-apigwy-binary": "^1.0.0",
    "serverless-dotenv-plugin": "^3.12.2"
  },
  "postinstall": "find ./node_modules/**/node_modules -type d -name 'bitcore-lib' -exec rm -r {} + && echo 'Deleted duplicate bitcore-libs'"
}
```