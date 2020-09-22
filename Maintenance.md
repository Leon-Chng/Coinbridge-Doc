# CoinBridge Maintenance Doc
  - [Genenal](#genenal)
  - [Add Position Field](#add-position-field)
  - [Add Exchange Account](#add-exchange-account)
  - [Add Instrument](#add-instrument)
  - [Upgrade Exchange Account Service](#upgrade-exchange-account-service)
  - [Upgrade Main Service](#upgrade-main-service)
  - [Upgrade Client](#upgrade-client)

## Genenal
- CoinBridge is based on `AWS Singapore`(Prop/Fund/Pokket) and we need to `RDP` these AWS machines for any updates.  
- All server machines are running in `private` security group, we provide `public` address for users/supporters to remote then you can reomote the private machines from public ones. Ask `administrator` if you don't have AWS login.  
(\*administrator is Leon for now)
- Use SSMS to run sql script, Database name IvyBridgeCoin with login Name IvyBridge.
- See our services in Windows Service(Ctrl+R, mstsc) and we can stop/start/restart if neccessary
    >1. Main Service - `IvyBridge Coin Trading Platform`
	>1. Exchange account service - `IvyBridge Coin (Exchange-Name)`  
	>e.g. IvyBridge Coin (Binance-1J). No account name means this service is for market feed only
	>1. Market Stability Service - `CoinStability`
- **Please send an email to `prop-dev@mingie.cn` after any update**

## Add Position Field
1. **Insert into Table "PositionFields"**  

column | ValueType | Description 
----|----|----
FieldName|string|Background field Name, main and unique key
FieldLableName|string|The field lable to display; If no other `PositionFieldValues` settings, uses this to map data from exchange 
FieldDescription|string|Not used
FieldIndex|int|Must be unique; Fields will show ordered by this index; Pokket alt coins should order by alpha
AlwaysHidden|bool|True - Hide this field
DecimalCount|int?|The decimal count to display, null = 0
1. **\*Insert into Table "PositionFieldValues"**  
This table is useful when  
a) we need to show multiple data in one cell  
b) the background data's key is different from the display name `FieldLableName` so we need some place to override the key setting
|Column | ValueType | Description |
|----|----|----|
|FieldName|string|Same with the key in `PositionFields`|
|ValueName|string|The value name, must be unique|
|ValueDescription|string|Not used|
|ValueIndex|int|If mutilple values for a FieldName, values are shown ordered by this index; If no multiple just set it 0|
|SupportedAccountFilter|bool?|Not used, just set it null|

## Add Exchange Account
1. **Insert into Table "Exchanges" and "Brokers"** (Skip if exchange already existing)  
`ExchangeName` and `BrokerName` must be same with enum `PlatformType`
2. **Insert into Table "BrokerAccounts"**
|Column | ValueType | Description |
|----|----|----|
|AccountId|int|Auto increased Id, the main key|
|AccountName|string|The displayed name|
|Platform|enum|In Coinbridge it's exchange type. Must be same with enum `PlatformType`|
|SecurityType|enum|`AccAndPsw=1`, `ConfigFile=2`; Set `ConfigFile` for market session or query session like Trezor; Set `AccAndPsw` for trade session|
|ConnectionInfo|string|The connection basic settings as xml format|
|AccountInfo|byte\[\]|Encrypted login info, set it null and let trader edit from UI|
|Index|int|Order by this index|
|UsageType|enum|`Market=1`,`Trade=2`,`PrivateQueryOnly=4`. Flag enum|
|AlwaysHidden|bool|False means hidden on UI|

3. **Insert into Table "ExchangeBrokerAccounts"**
- Priority must be unique for the same exchange and broker account
- For priority 0 means highest. Use this to order the supporting broker accounts for every exchange \- Put the most frequently used account top

## Add Instrument
1. If underlying coin not existing in DB, add this coin firstly (Table "Instruments" and inherited tables e.g. "Currency", "Index"))
1. Insert instrument itself if not existing (Table "Instruments" and inherited tables - Spot pair = "FX", SWAP/Future = "Future", Option = "Option", Fund = "Fund")  
1. Add alias if this symbol has different alias name in this exchange (Insert Table "InstrumentBrokerExtensions")
1. Add instrument's exchange supporting (Insert Table "InstrumentSupportExchanges")
1. Add position field if new symbol's coin or contract not existing [Jump](#add-position-field)

## Upgrade Exchange Account Service
Using Octopus http://team:81/ (Open in office network). Project `Coin Services`

## Upgrade Main Service
Using Octopus http://team:81/ (Open in office network). Project `Coin Trading Plarform`

## Upgrade Client
1. Download installer file .msi from http://team:8000/ (Open in office network) `Trading System -> Trading Client CoinTest -> Artifacts(The logo button at right)`.
1. Copy it to server machine `C:\Share\Latest\CoinPROD`
1. Open every client machine and try open CoinBridge client there to auto upgrade. If failed to auto upgrade, copy the installer to client and run it.
