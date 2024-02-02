resources

| where type == "microsoft.network/virtualnetworks"

| project vnetName = name, subnets = (properties.subnets), resourceGroup, subscriptionId

| join kind=leftouter (resourcecontainers 

| where type=='microsoft.resources/subscriptions'

| project SubcriptionName=name, subscriptionId) on subscriptionId

| mvexpand subnets

| extend subnet_name=tostring(subnets.name)

| extend route_table=subnets.properties.routeTable

| where isnull(subnets.properties.routeTable)

| extend sub_prefix=iff(isnotnull(subnets.properties.addressPrefix), subnets.properties.addressPrefix, strcat_array(subnets.properties.addressPrefixes, "," ))

| where sub_prefix !startswith '10.23.'

| project SubcriptionName, resourceGroup, vnetName, subnet_name, sub_prefix, route_table_name=split((split(tostring(route_table.id), '/'))[8], '"')[0]

 

 

 

//https://learn.microsoft.com/en-us/azure/templates/microsoft.recoveryservices/2016-06-01/vaults/backupfabrics/protectioncontainers/protecteditems?pivots=deployment-language-arm-template

RecoveryServicesResources

| where type == "microsoft.recoveryservices/vaults/backupfabrics/protectioncontainers/protecteditems"

| project propertiesJSON = parse_json(properties)

| where propertiesJSON.backupManagementType == "AzureIaasVM"

| project PolicyInconsistant=propertiesJSON.extendedInfo.policyInconsistent, ProtectionState=propertiesJSON.protectionState, VMNAME=propertiesJSON.friendlyName, VMID=propertiesJSON.sourceResourceId, PolicyID=propertiesJSON.policyId, PolicyName=propertiesJSON.policyName, LastRecoveryPoint=propertiesJSON.lastRecoveryPoint, OldestRecoveryPoint=propertiesJSON.extendedInfo.oldestRecoveryPoint

| where PolicyName == "Standard15" or PolicyName == "Standard30" or PolicyName == "Local15" or PolicyName == "Local30" or PolicyName == "DoNotDestroy" or PolicyName == "LocalOnlyDoNotDestroy"

 

resources

| where type =~ "Microsoft.Network/routeTables"

| mv-expand rules = properties.routes

| join kind=leftouter (resourcecontainers 

| where type=='microsoft.resources/subscriptions' 

| project SubcriptionName=name, subscriptionId) on subscriptionId

| extend subnet_name = split((split(tostring(properties.subnets), '/'))[10], '"')[0]

| extend addressPrefix = tostring(rules.properties.addressPrefix)

| extend nextHopType = tostring(rules.properties.nextHopType)

| extend nextHopIpAddress = tostring(rules.properties.nextHopIpAddress)

| extend hasBgpOverride = tostring(rules.properties.hasBgpOverride)

| extend provisioningState = tostring(rules.properties.provisioningState)

| extend udrname = rules.name

| extend rtname = name

| project SubcriptionName, resourceGroup, subnet_name, rtname, udrname, addressPrefix, nextHopType, nextHopIpAddress, provisioningState, hasBgpOverride

| sort by SubcriptionName, resourceGroup asc, rtname asc, addressPrefix asc

 

resources

| where type == "microsoft.network/virtualnetworks"

| project vnetName = name, subnets = (properties.subnets)

| mvexpand subnets

| extend subnet_name=tostring(subnets.name)

| extend sub_prefix=iff(isnotnull(subnets.properties.addressPrefix), subnets.properties.addressPrefix, strcat_array(subnets.properties.addressPrefixes, "," ))

| project vnetName, subnet_name, sub_prefix

 
