# Chaincode for Report Generation
- Reference Material : the linux foundation lfd272

## Set Netowrk && Environment

### test-network 구동
- hyperledger fabric v2.2
- ./network.sh up createChannel -ca -s couchdb

### CouchDB URL
- http://localhost:5984/_utils 확인 가능
- COUCHDB_USER=admin
- COUCHDB_PASSWORD=adminpw

### Terminal 1 Environment variables for Org1MS 
- export PATH=${PWD}/../bin:$PATH
- export FABRIC_CFG_PATH=$PWD/../config/
- cd $HOME/go/src/github.com/pwhdgur/hyperledger/fabric-samples/test-network
- export CORE_PEER_TLS_ENABLED=true
- export CORE_PEER_LOCALMSPID="Org1MSP"
- export CORE_PEER_TLS_ROOTCERT_FILE=${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt
- export CORE_PEER_MSPCONFIGPATH=${PWD}/organizations/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp
- export CORE_PEER_ADDRESS=localhost:7051

## Deploy Chaincode
- ./network.sh deployCC -ccn reports -ccl javascript -ccp ~/go/src/github.com/pwhdgur/hyperledger/fabric-samples/lfd272/chaincodes/Lab6/reports_chaincode -ccv 1.0 -ccs 1 -cci init

### Generate the annual report for 2019 
- peer chaincode invoke -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile ${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C mychannel -n reports --peerAddresses localhost:7051 --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt --peerAddresses localhost:9051 --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt -c '{"function":"getAnnualReport","Args":["2019"]}'

### Generate a custom report with any query string. (example : a negative price)
- peer chaincode query -n reports -C mychannel -c '{"function":"generateCustomReport", "Args":["{\"selector\":{\"price\": {\"$lt\": 0}}}"]}'

### Add New Records
- peer chaincode invoke -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile ${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C mychannel -n reports --peerAddresses localhost:7051 --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt --peerAddresses localhost:9051 --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt -c '{"function":"putRecord","Args":["id6", "goods6", "50.0", "2021-08-22"]}'

## Simple_Chaincode Code View
- putRecord
- getAnnualReport
- generateCustomReport
- _getResultsForQueryString

### putRecord
#### check if the record ID is unique toavoid data rewriting.
- await ctx.stub.getState(compositeKey);

#### store a JSON-encoded record as avalue associated with its composite key
- await ctx.stub.putState(compositeKey, Buffer.from(JSON.stringify(record)));

### _getResultsForQueryString
#### perform a rich query using a query string. 
- const queryString = `{"selector":{"date":{"year":${year}}}, "use_index":["_design/indexYearDoc", "indexYear"]}`;

## Docker Log
- docker logs dev-peer0.org1.example.com-reports_1.0-3e5c30afbe220a0c5d91e6c9fe58a8e40938bd1c59176dcf4e3e728f13ba3e27 -f

## Clear Up
- ./network.sh down