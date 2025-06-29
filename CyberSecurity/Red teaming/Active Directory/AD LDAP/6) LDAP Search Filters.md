## Basic LDAP Filter Syntax and Operators

The `LDAPFilter` parameter with the same cmdlets lets us use LDAP search filters when searching for information. The syntax for these filters is defined in [RFC 4515 - Lightweight Directory Access Protocol (LDAP): String Representation of Search Filters](https://tools.ietf.org/html/rfc4515).

LDAP filters must have one or more criteria. If more than one criteria exist, they can be concatenated together using logical `AND` or `OR` operators. These operators are always placed in the front of the criteria (operands), which is also referred to as [Polish Notation](https://en.wikipedia.org/wiki/Polish_notation).

Filter rules are enclosed in parentheses and can be grouped by surrounding the group in parentheses and using one of the following comparison operators:

|**Operator**|**Function**|
|---|---|
|`&`|and|
|`\|`|or|
|`!`|not|

Some example `AND` and `OR` operations are as follows:

`AND` Operation:

- One criteria: `(& (..C1..) (..C2..))`
- More than two criteria: `(& (..C1..) (..C2..) (..C3..))`

`OR` Operation:

- One criteria: `(| (..C1..) (..C2..))`
- More than two criteria: `(| (..C1..) (..C2..) (..C3..))`

We can also have nested operations, for example "`(|(& (..C1..) (..C2..))(& (..C3..) (..C4..)))`" translates to "`(C1 AND C2) OR (C3 AND C4)`".

---
## Search Criteria
This [link](https://docs.bmc.com/docs/fpsc121/ldap-attributes-and-associated-fields-495323340.html) contains a large listing of User Attributes, and the below is a list of all Base Attributes.

Full list of Base Attributes

|**LDAP Display Name**|**CN**|**Attribute ID**|
|---|---|---|
|`accountExpires`|Account-Expires|1.2.840.113556.1.4.159|
|`accountNameHistory`|Account-Name-History|1.2.840.113556.1.4.1307|
|`aCSAggregateTokenRatePerUser`|ACS-Aggregate-Token-Rate-Per-User|1.2.840.113556.1.4.760|
|`aCSAllocableRSVPBandwidth`|ACS-Allocable-RSVP-Bandwidth|1.2.840.113556.1.4.766|
|`aCSCacheTimeout`|ACS-Cache-Timeout|1.2.840.113556.1.4.779|
|`aCSDirection`|ACS-Direction|1.2.840.113556.1.4.757|
|`aCSDSBMDeadTime`|ACS-DSBM-DeadTime|1.2.840.113556.1.4.778|
|`aCSDSBMPriority`|ACS-DSBM-Priority|1.2.840.113556.1.4.776|
|`aCSDSBMRefresh`|ACS-DSBM-Refresh|1.2.840.113556.1.4.777|
|`aCSEnableACSService`|ACS-Enable-ACS-Service|1.2.840.113556.1.4.770|
|`aCSEnableRSVPAccounting`|ACS-Enable-RSVP-Accounting|1.2.840.113556.1.4.899|
|`aCSEnableRSVPMessageLogging`|ACS-Enable-RSVP-Message-Logging|1.2.840.113556.1.4.768|
|`aCSEventLogLevel`|ACS-Event-Log-Level|1.2.840.113556.1.4.769|
|`aCSIdentityName`|ACS-Identity-Name|1.2.840.113556.1.4.784|
|`aCSMaxAggregatePeakRatePerUser`|ACS-Max-Aggregate-Peak-Rate-Per-User|1.2.840.113556.1.4.897|
|`aCSMaxDurationPerFlow`|ACS-Max-Duration-Per-Flow|1.2.840.113556.1.4.761|
|`aCSMaximumSDUSize`|ACS-Maximum-SDU-Size|1.2.840.113556.1.4.1314|
|`aCSMaxNoOfAccountFiles`|ACS-Max-No-Of-Account-Files|1.2.840.113556.1.4.901|
|`aCSMaxNoOfLogFiles`|ACS-Max-No-Of-Log-Files|1.2.840.113556.1.4.774|
|`aCSMaxPeakBandwidth`|ACS-Max-Peak-Bandwidth|1.2.840.113556.1.4.767|
|`aCSMaxPeakBandwidthPerFlow`|ACS-Max-Peak-Bandwidth-Per-Flow|1.2.840.113556.1.4.759|
|`aCSMaxSizeOfRSVPAccountFile`|ACS-Max-Size-Of-RSVP-Account-File|1.2.840.113556.1.4.902|
|`aCSMaxSizeOfRSVPLogFile`|ACS-Max-Size-Of-RSVP-Log-File|1.2.840.113556.1.4.775|
|`aCSMaxTokenBucketPerFlow`|ACS-Max-Token-Bucket-Per-Flow|1.2.840.113556.1.4.1313|
|`aCSMaxTokenRatePerFlow`|ACS-Max-Token-Rate-Per-Flow|1.2.840.113556.1.4.758|
|`aCSMinimumDelayVariation`|ACS-Minimum-Delay-Variation|1.2.840.113556.1.4.1317|
|`aCSMinimumLatency`|ACS-Minimum-Latency|1.2.840.113556.1.4.1316|
|`aCSMinimumPolicedSize`|ACS-Minimum-Policed-Size|1.2.840.113556.1.4.1315|
|`aCSNonReservedMaxSDUSize`|ACS-Non-Reserved-Max-SDU-Size|1.2.840.113556.1.4.1320|
|`aCSNonReservedMinPolicedSize`|ACS-Non-Reserved-Min-Policed-Size|1.2.840.113556.1.4.1321|
|`aCSNonReservedPeakRate`|ACS-Non-Reserved-Peak-Rate|1.2.840.113556.1.4.1318|
|`aCSNonReservedTokenSize`|ACS-Non-Reserved-Token-Size|1.2.840.113556.1.4.1319|
|`aCSNonReservedTxLimit`|ACS-Non-Reserved-Tx-Limit|1.2.840.113556.1.4.780|
|`aCSNonReservedTxSize`|ACS-Non-Reserved-Tx-Size|1.2.840.113556.1.4.898|
|`aCSPermissionBits`|ACS-Permission-Bits|1.2.840.113556.1.4.765|
|`aCSPolicyName`|ACS-Policy-Name|1.2.840.113556.1.4.772|
|`aCSPriority`|ACS-Priority|1.2.840.113556.1.4.764|
|`aCSRSVPAccountFilesLocation`|ACS-RSVP-Account-Files-Location|1.2.840.113556.1.4.900|
|`aCSRSVPLogFilesLocation`|ACS-RSVP-Log-Files-Location|1.2.840.113556.1.4.773|
|`aCSServerList`|ACS-Server-List|1.2.840.113556.1.4.1312|
|`aCSServiceType`|ACS-Service-Type|1.2.840.113556.1.4.762|
|`aCSTimeOfDay`|ACS-Time-Of-Day|1.2.840.113556.1.4.756|
|`aCSTotalNoOfFlows`|ACS-Total-No-Of-Flows|1.2.840.113556.1.4.763|
|`additionalTrustedServiceNames`|Additional-Trusted-Service-Names|1.2.840.113556.1.4.889|
|`addressBookRoots`|Address-Book-Roots|1.2.840.113556.1.4.1244|
|`addressEntryDisplayTable`|Address-Entry-Display-Table|1.2.840.113556.1.2.324|
|`addressEntryDisplayTableMSDOS`|Address-Entry-Display-Table-MSDOS|1.2.840.113556.1.2.400|
|`addressSyntax`|Address-Syntax|1.2.840.113556.1.2.255|
|`addressType`|Address-Type|1.2.840.113556.1.2.350|
|`adminContextMenu`|Admin-Context-Menu|1.2.840.113556.1.4.614|
|`adminCount`|Admin-Count|1.2.840.113556.1.4.150|
|`adminDescription`|Admin-Description|1.2.840.113556.1.2.226|
|`adminDisplayName`|Admin-Display-Name|1.2.840.113556.1.2.194|
|`adminPropertyPages`|Admin-Property-Pages|1.2.840.113556.1.4.562|
|`allowedAttributes`|Allowed-Attributes|1.2.840.113556.1.4.913|
|`allowedAttributesEffective`|Allowed-Attributes-Effective|1.2.840.113556.1.4.914|
|`allowedChildClasses`|Allowed-Child-Classes|1.2.840.113556.1.4.911|
|`allowedChildClassesEffective`|Allowed-Child-Classes-Effective|1.2.840.113556.1.4.912|
|`altSecurityIdentities`|Alt-Security-Identities|1.2.840.113556.1.4.867|
|`aNR`|ANR|1.2.840.113556.1.4.1208|
|`applicationName`|Application-Name|1.2.840.113556.1.4.218|
|`appliesTo`|Applies-To|1.2.840.113556.1.4.341|
|`appSchemaVersion`|App-Schema-Version|1.2.840.113556.1.4.848|
|`assetNumber`|Asset-Number|1.2.840.113556.1.4.283|
|`assistant`|Assistant|1.2.840.113556.1.4.652|
|`assocNTAccount`|Assoc-NT-Account|1.2.840.113556.1.4.1213|
|`attributeDisplayNames`|Attribute-Display-Names|1.2.840.113556.1.4.748|
|`attributeID`|Attribute-ID|1.2.840.113556.1.2.30|
|`attributeSecurityGUID`|Attribute-Security-GUID|1.2.840.113556.1.4.149|
|`attributeSyntax`|Attribute-Syntax|1.2.840.113556.1.2.32|
|`attributeTypes`|Attribute-Types|2.5.21.5|
|`auditingPolicy`|Auditing-Policy|1.2.840.113556.1.4.202|
|`authenticationOptions`|Authentication-Options|1.2.840.113556.1.4.11|
|`authorityRevocationList`|Authority-Revocation-List|2.5.4.38|
|`auxiliaryClass`|Auxiliary-Class|1.2.840.113556.1.2.351|
|`badPasswordTime`|Bad-Password-Time|1.2.840.113556.1.4.49|
|`badPwdCount`|Bad-Pwd-Count|1.2.840.113556.1.4.12|
|`birthLocation`|Birth-Location|1.2.840.113556.1.4.332|
|`bridgeheadServerListBL`|Bridgehead-Server-List-BL|1.2.840.113556.1.4.820|
|`bridgeheadTransportList`|Bridgehead-Transport-List|1.2.840.113556.1.4.819|
|`builtinCreationTime`|Builtin-Creation-Time|1.2.840.113556.1.4.13|
|`builtinModifiedCount`|Builtin-Modified-Count|1.2.840.113556.1.4.14|
|`businessCategory`|Business-Category|2.5.4.15|
|`bytesPerMinute`|Bytes-Per-Minute|1.2.840.113556.1.4.284|
|`c`|Country-Name|2.5.4.6|
|`cACertificate`|CA-Certificate|2.5.4.37|
|`cACertificateDN`|CA-Certificate-DN|1.2.840.113556.1.4.697|
|`cAConnect`|CA-Connect|1.2.840.113556.1.4.687|
|`canonicalName`|Canonical-Name|1.2.840.113556.1.4.916|
|`canUpgradeScript`|Can-Upgrade-Script|1.2.840.113556.1.4.815|
|`catalogs`|Catalogs|1.2.840.113556.1.4.675|
|`categories`|Categories|1.2.840.113556.1.4.672|
|`categoryId`|Category-Id|1.2.840.113556.1.4.322|
|`cAUsages`|CA-Usages|1.2.840.113556.1.4.690|
|`cAWEBURL`|CA-WEB-URL|1.2.840.113556.1.4.688|
|`certificateAuthorityObject`|Certificate-Authority-Object|1.2.840.113556.1.4.684|
|`certificateRevocationList`|Certificate-Revocation-List|2.5.4.39|
|`certificateTemplates`|Certificate-Templates|1.2.840.113556.1.4.823|
|`classDisplayName`|Class-Display-Name|1.2.840.113556.1.4.610|
|`cn`|Common-Name|2.5.4.3|
|`co`|Text-Country|1.2.840.113556.1.2.131|
|`codePage`|Code-Page|1.2.840.113556.1.4.16|
|`cOMClassID`|COM-ClassID|1.2.840.113556.1.4.19|
|`cOMCLSID`|COM-CLSID|1.2.840.113556.1.4.249|
|`cOMInterfaceID`|COM-InterfaceID|1.2.840.113556.1.4.20|
|`comment`|User-Comment|1.2.840.113556.1.4.156|
|`cOMOtherProgId`|COM-Other-Prog-Id|1.2.840.113556.1.4.253|
|`company`|Company|1.2.840.113556.1.2.146|
|`cOMProgID`|COM-ProgID|1.2.840.113556.1.4.21|
|`cOMTreatAsClassId`|COM-Treat-As-Class-Id|1.2.840.113556.1.4.251|
|`cOMTypelibId`|COM-Typelib-Id|1.2.840.113556.1.4.254|
|`cOMUniqueLIBID`|COM-Unique-LIBID|1.2.840.113556.1.4.250|
|`contentIndexingAllowed`|Content-Indexing-Allowed|1.2.840.113556.1.4.24|
|`contextMenu`|Context-Menu|1.2.840.113556.1.4.499|
|`controlAccessRights`|Control-Access-Rights|1.2.840.113556.1.4.200|
|`cost`|Cost|1.2.840.113556.1.2.135|
|`countryCode`|Country-Code|1.2.840.113556.1.4.25|
|`createDialog`|Create-Dialog|1.2.840.113556.1.4.810|
|`createTimeStamp`|Create-Time-Stamp|2.5.18.1|
|`createWizardExt`|Create-Wizard-Ext|1.2.840.113556.1.4.812|
|`creationTime`|Creation-Time|1.2.840.113556.1.4.26|
|`creationWizard`|Creation-Wizard|1.2.840.113556.1.4.498|
|`creator`|Creator|1.2.840.113556.1.4.679|
|`cRLObject`|CRL-Object|1.2.840.113556.1.4.689|
|`cRLPartitionedRevocationList`|CRL-Partitioned-Revocation-List|1.2.840.113556.1.4.683|
|`crossCertificatePair`|Cross-Certificate-Pair|2.5.4.40|
|`currentLocation`|Current-Location|1.2.840.113556.1.4.335|
|`currentParentCA`|Current-Parent-CA|1.2.840.113556.1.4.696|
|`currentValue`|Current-Value|1.2.840.113556.1.4.27|
|`currMachineId`|Curr-Machine-Id|1.2.840.113556.1.4.337|
|`dBCSPwd`|DBCS-Pwd|1.2.840.113556.1.4.55|
|`dc`|Domain-Component|0.9.2342.19200300.100.1.25|
|`defaultClassStore`|Default-Class-Store|1.2.840.113556.1.4.213|
|`defaultGroup`|Default-Group|1.2.840.113556.1.4.480|
|`defaultHidingValue`|Default-Hiding-Value|1.2.840.113556.1.4.518|
|`defaultLocalPolicyObject`|Default-Local-Policy-Object|1.2.840.113556.1.4.57|
|`defaultObjectCategory`|Default-Object-Category|1.2.840.113556.1.4.783|
|`defaultPriority`|Default-Priority|1.2.840.113556.1.4.232|
|`defaultSecurityDescriptor`|Default-Security-Descriptor|1.2.840.113556.1.4.224|
|`deltaRevocationList`|Delta-Revocation-List|2.5.4.53|
|`department`|Department|1.2.840.113556.1.2.141|
|`description`|Description|2.5.4.13|
|`desktopProfile`|Desktop-Profile|1.2.840.113556.1.4.346|
|`destinationIndicator`|Destination-Indicator|2.5.4.27|
|`dhcpClasses`|dhcp-Classes|1.2.840.113556.1.4.715|
|`dhcpFlags`|dhcp-Flags|1.2.840.113556.1.4.700|
|`dhcpIdentification`|dhcp-Identification|1.2.840.113556.1.4.701|
|`dhcpMask`|dhcp-Mask|1.2.840.113556.1.4.706|
|`dhcpMaxKey`|dhcp-MaxKey|1.2.840.113556.1.4.719|
|`dhcpObjDescription`|dhcp-Obj-Description|1.2.840.113556.1.4.703|
|`dhcpObjName`|dhcp-Obj-Name|1.2.840.113556.1.4.702|
|`dhcpOptions`|dhcp-Options|1.2.840.113556.1.4.714|
|`dhcpProperties`|dhcp-Properties|1.2.840.113556.1.4.718|
|`dhcpRanges`|dhcp-Ranges|1.2.840.113556.1.4.707|
|`dhcpReservations`|dhcp-Reservations|1.2.840.113556.1.4.709|
|`dhcpServers`|dhcp-Servers|1.2.840.113556.1.4.704|
|`dhcpSites`|dhcp-Sites|1.2.840.113556.1.4.708|
|`dhcpState`|dhcp-State|1.2.840.113556.1.4.717|
|`dhcpSubnets`|dhcp-Subnets|1.2.840.113556.1.4.705|
|`dhcpType`|dhcp-Type|1.2.840.113556.1.4.699|
|`dhcpUniqueKey`|dhcp-Unique-Key|1.2.840.113556.1.4.698|
|`dhcpUpdateTime`|dhcp-Update-Time|1.2.840.113556.1.4.720|
|`directReports`|Reports|1.2.840.113556.1.2.436|
|`displayName`|Display-Name|1.2.840.113556.1.2.13|
|`displayNamePrintable`|Display-Name-Printable|1.2.840.113556.1.2.353|
|`distinguishedName`|Obj-Dist-Name|2.5.4.49|
|`dITContentRules`|DIT-Content-Rules|2.5.21.2|
|`division`|Division|1.2.840.113556.1.4.261|
|`dMDLocation`|DMD-Location|1.2.840.113556.1.2.36|
|`dmdName`|DMD-Name|1.2.840.113556.1.2.598|
|`dNReferenceUpdate`|DN-Reference-Update|1.2.840.113556.1.4.1242|
|`dnsAllowDynamic`|Dns-Allow-Dynamic|1.2.840.113556.1.4.378|
|`dnsAllowXFR`|Dns-Allow-XFR|1.2.840.113556.1.4.379|
|`dNSHostName`|DNS-Host-Name|1.2.840.113556.1.4.619|
|`dnsNotifySecondaries`|Dns-Notify-Secondaries|1.2.840.113556.1.4.381|
|`dNSProperty`|DNS-Property|1.2.840.113556.1.4.1306|
|`dnsRecord`|Dns-Record|1.2.840.113556.1.4.382|
|`dnsRoot`|Dns-Root|1.2.840.113556.1.4.28|
|`dnsSecureSecondaries`|Dns-Secure-Secondaries|1.2.840.113556.1.4.380|
|`dNSTombstoned`|DNS-Tombstoned|1.2.840.113556.1.4.1414|
|`domainCAs`|Domain-Certificate-Authorities|1.2.840.113556.1.4.668|
|`domainCrossRef`|Domain-Cross-Ref|1.2.840.113556.1.4.472|
|`domainID`|Domain-ID|1.2.840.113556.1.4.686|
|`domainIdentifier`|Domain-Identifier|1.2.840.113556.1.4.755|
|`domainPolicyObject`|Domain-Policy-Object|1.2.840.113556.1.4.32|
|`domainPolicyReference`|Domain-Policy-Reference|1.2.840.113556.1.4.422|
|`domainReplica`|Domain-Replica|1.2.840.113556.1.4.158|
|`domainWidePolicy`|Domain-Wide-Policy|1.2.840.113556.1.4.421|
|`driverName`|Driver-Name|1.2.840.113556.1.4.229|
|`driverVersion`|Driver-Version|1.2.840.113556.1.4.276|
|`dSASignature`|DSA-Signature|1.2.840.113556.1.2.74|
|`dSCorePropagationData`|DS-Core-Propagation-Data|1.2.840.113556.1.4.1357|
|`dSHeuristics`|DS-Heuristics|1.2.840.113556.1.2.212|
|`dSUIAdminMaximum`|DS-UI-Admin-Maximum|1.2.840.113556.1.4.1344|
|`dSUIAdminNotification`|DS-UI-Admin-Notification|1.2.840.113556.1.4.1343|
|`dSUIShellMaximum`|DS-UI-Shell-Maximum|1.2.840.113556.1.4.1345|
|`dynamicLDAPServer`|Dynamic-LDAP-Server|1.2.840.113556.1.4.537|
|`eFSPolicy`|EFSPolicy|1.2.840.113556.1.4.268|
|`employeeID`|Employee-ID|1.2.840.113556.1.4.35|
|`employeeNumber`|Employee-Number|1.2.840.113556.1.2.610|
|`employeeType`|Employee-Type|1.2.840.113556.1.2.613|
|`Enabled`|Enabled|1.2.840.113556.1.2.557|
|`enabledConnection`|Enabled-Connection|1.2.840.113556.1.4.36|
|`enrollmentProviders`|Enrollment-Providers|1.2.840.113556.1.4.825|
|`extendedAttributeInfo`|Extended-Attribute-Info|1.2.840.113556.1.4.909|
|`extendedCharsAllowed`|Extended-Chars-Allowed|1.2.840.113556.1.2.380|
|`extendedClassInfo`|Extended-Class-Info|1.2.840.113556.1.4.908|
|`extensionName`|Extension-Name|1.2.840.113556.1.2.227|
|`facsimileTelephoneNumber`|Facsimile-Telephone-Number|2.5.4.23|
|`fileExtPriority`|File-Ext-Priority|1.2.840.113556.1.4.816|
|`flags`|Flags|1.2.840.113556.1.4.38|
|`flatName`|Flat-Name|1.2.840.113556.1.4.511|
|`forceLogoff`|Force-Logoff|1.2.840.113556.1.4.39|
|`foreignIdentifier`|Foreign-Identifier|1.2.840.113556.1.4.356|
|`friendlyNames`|Friendly-Names|1.2.840.113556.1.4.682|
|`fromEntry`|From-Entry|1.2.840.113556.1.4.910|
|`fromServer`|From-Server|1.2.840.113556.1.4.40|
|`frsComputerReference`|Frs-Computer-Reference|1.2.840.113556.1.4.869|
|`frsComputerReferenceBL`|Frs-Computer-Reference-BL|1.2.840.113556.1.4.870|
|`fRSControlDataCreation`|FRS-Control-Data-Creation|1.2.840.113556.1.4.871|
|`fRSControlInboundBacklog`|FRS-Control-Inbound-Backlog|1.2.840.113556.1.4.872|
|`fRSControlOutboundBacklog`|FRS-Control-Outbound-Backlog|1.2.840.113556.1.4.873|
|`fRSDirectoryFilter`|FRS-Directory-Filter|1.2.840.113556.1.4.484|
|`fRSDSPoll`|FRS-DS-Poll|1.2.840.113556.1.4.490|
|`fRSExtensions`|FRS-Extensions|1.2.840.113556.1.4.536|
|`fRSFaultCondition`|FRS-Fault-Condition|1.2.840.113556.1.4.491|
|`fRSFileFilter`|FRS-File-Filter|1.2.840.113556.1.4.483|
|`fRSFlags`|FRS-Flags|1.2.840.113556.1.4.874|
|`fRSLevelLimit`|FRS-Level-Limit|1.2.840.113556.1.4.534|
|`fRSMemberReference`|FRS-Member-Reference|1.2.840.113556.1.4.875|
|`fRSMemberReferenceBL`|FRS-Member-Reference-BL|1.2.840.113556.1.4.876|
|`fRSPartnerAuthLevel`|FRS-Partner-Auth-Level|1.2.840.113556.1.4.877|
|`fRSPrimaryMember`|FRS-Primary-Member|1.2.840.113556.1.4.878|
|`fRSReplicaSetGUID`|FRS-Replica-Set-GUID|1.2.840.113556.1.4.533|
|`fRSReplicaSetType`|FRS-Replica-Set-Type|1.2.840.113556.1.4.31|
|`fRSRootPath`|FRS-Root-Path|1.2.840.113556.1.4.487|
|`fRSRootSecurity`|FRS-Root-Security|1.2.840.113556.1.4.535|
|`fRSServiceCommand`|FRS-Service-Command|1.2.840.113556.1.4.500|
|`fRSServiceCommandStatus`|FRS-Service-Command-Status|1.2.840.113556.1.4.879|
|`fRSStagingPath`|FRS-Staging-Path|1.2.840.113556.1.4.488|
|`fRSTimeLastCommand`|FRS-Time-Last-Command|1.2.840.113556.1.4.880|
|`fRSTimeLastConfigChange`|FRS-Time-Last-Config-Change|1.2.840.113556.1.4.881|
|`fRSUpdateTimeout`|FRS-Update-Timeout|1.2.840.113556.1.4.485|
|`fRSVersion`|FRS-Version|1.2.840.113556.1.4.882|
|`fRSVersionGUID`|FRS-Version-GUID|1.2.840.113556.1.4.43|
|`fRSWorkingPath`|FRS-Working-Path|1.2.840.113556.1.4.486|
|`fSMORoleOwner`|FSMO-Role-Owner|1.2.840.113556.1.4.369|
|`garbageCollPeriod`|Garbage-Coll-Period|1.2.840.113556.1.2.301|
|`generatedConnection`|Generated-Connection|1.2.840.113556.1.4.41|
|`generationQualifier`|Generation-Qualifier|2.5.4.44|
|`givenName`|Given-Name|2.5.4.42|
|`globalAddressList`|Global-Address-List|1.2.840.113556.1.4.1245|
|`governsID`|Governs-ID|1.2.840.113556.1.2.22|
|`gPCFileSysPath`|GPC-File-Sys-Path|1.2.840.113556.1.4.894|
|`gPCFunctionalityVersion`|GPC-Functionality-Version|1.2.840.113556.1.4.893|
|`gPCMachineExtensionNames`|GPC-Machine-Extension-Names|1.2.840.113556.1.4.1348|
|`gPCUserExtensionNames`|GPC-User-Extension-Names|1.2.840.113556.1.4.1349|
|`gPLink`|GP-Link|1.2.840.113556.1.4.891|
|`gPOptions`|GP-Options|1.2.840.113556.1.4.892|
|`groupAttributes`|Group-Attributes|1.2.840.113556.1.4.152|
|`groupMembershipSAM`|Group-Membership-SAM|1.2.840.113556.1.4.166|
|`groupPriority`|Group-Priority|1.2.840.113556.1.4.345|
|`groupsToIgnore`|Groups-to-Ignore|1.2.840.113556.1.4.344|
|`groupType`|Group-Type|1.2.840.113556.1.4.750|
|`hasMasterNCs`|Has-Master-NCs|1.2.840.113556.1.2.14|
|`hasPartialReplicaNCs`|Has-Partial-Replica-NCs|1.2.840.113556.1.2.15|
|`helpData16`|Help-Data16|1.2.840.113556.1.2.402|
|`helpData32`|Help-Data32|1.2.840.113556.1.2.9|
|`helpFileName`|Help-File-Name|1.2.840.113556.1.2.327|
|`homeDirectory`|Home-Directory|1.2.840.113556.1.4.44|
|`homeDrive`|Home-Drive|1.2.840.113556.1.4.45|
|`homePhone`|Phone-Home-Primary|0.9.2342.19200300.100.1.20|
|`homePostalAddress`|Address-Home|1.2.840.113556.1.2.617|
|`iconPath`|Icon-Path|1.2.840.113556.1.4.219|
|`implementedCategories`|Implemented-Categories|1.2.840.113556.1.4.320|
|`indexedScopes`|IndexedScopes|1.2.840.113556.1.4.681|
|`info`|Comment|1.2.840.113556.1.2.81|
|`initialAuthIncoming`|Initial-Auth-Incoming|1.2.840.113556.1.4.539|
|`initialAuthOutgoing`|Initial-Auth-Outgoing|1.2.840.113556.1.4.540|
|`initials`|Initials|2.5.4.43|
|`installUiLevel`|Install-Ui-Level|1.2.840.113556.1.4.847|
|`instanceType`|Instance-Type|1.2.840.113556.1.2.1|
|`internationalISDNNumber`|International-ISDN-Number|2.5.4.25|
|`interSiteTopologyFailover`|Inter-Site-Topology-Failover|1.2.840.113556.1.4.1248|
|`interSiteTopologyGenerator`|Inter-Site-Topology-Generator|1.2.840.113556.1.4.1246|
|`interSiteTopologyRenew`|Inter-Site-Topology-Renew|1.2.840.113556.1.4.1247|
|`invocationId`|Invocation-Id|1.2.840.113556.1.2.115|
|`ipPhone`|Phone-Ip-Primary|1.2.840.113556.1.4.721|
|`ipsecData`|Ipsec-Data|1.2.840.113556.1.4.623|
|`ipsecDataType`|Ipsec-Data-Type|1.2.840.113556.1.4.622|
|`ipsecFilterReference`|Ipsec-Filter-Reference|1.2.840.113556.1.4.629|
|`ipsecID`|Ipsec-ID|1.2.840.113556.1.4.621|
|`ipsecISAKMPReference`|Ipsec-ISAKMP-Reference|1.2.840.113556.1.4.626|
|`ipsecName`|Ipsec-Name|1.2.840.113556.1.4.620|
|`iPSECNegotiationPolicyAction`|IPSEC-Negotiation-Policy-Action|1.2.840.113556.1.4.888|
|`ipsecNegotiationPolicyReference`|Ipsec-Negotiation-Policy-Reference|1.2.840.113556.1.4.628|
|`iPSECNegotiationPolicyType`|IPSEC-Negotiation-Policy-Type|1.2.840.113556.1.4.887|
|`ipsecNFAReference`|Ipsec-NFA-Reference|1.2.840.113556.1.4.627|
|`ipsecOwnersReference`|Ipsec-Owners-Reference|1.2.840.113556.1.4.624|
|`ipsecPolicyReference`|Ipsec-Policy-Reference|1.2.840.113556.1.4.517|
|`isCriticalSystemObject`|Is-Critical-System-Object|1.2.840.113556.1.4.868|
|`isDefunct`|Is-Defunct|1.2.840.113556.1.4.661|
|`isDeleted`|Is-Deleted|1.2.840.113556.1.2.48|
|`isEphemeral`|Is-Ephemeral|1.2.840.113556.1.4.1212|
|`isMemberOfPartialAttributeSet`|Is-Member-Of-Partial-Attribute-Set|1.2.840.113556.1.4.639|
|`isPrivilegeHolder`|Is-Privilege-Holder|1.2.840.113556.1.4.638|
|`isSingleValued`|Is-Single-Valued|1.2.840.113556.1.2.33|
|`keywords`|Keywords|1.2.840.113556.1.4.48|
|`knowledgeInformation`|Knowledge-Information|2.5.4.2|
|`l`|Locality-Name|2.5.4.7|
|`lastBackupRestorationTime`|Last-Backup-Restoration-Time|1.2.840.113556.1.4.519|
|`lastContentIndexed`|Last-Content-Indexed|1.2.840.113556.1.4.50|
|`lastKnownParent`|Last-Known-Parent|1.2.840.113556.1.4.781|
|`lastLogoff`|Last-Logoff|1.2.840.113556.1.4.51|
|`lastLogon`|Last-Logon|1.2.840.113556.1.4.52|
|`lastSetTime`|Last-Set-Time|1.2.840.113556.1.4.53|
|`lastUpdateSequence`|Last-Update-Sequence|1.2.840.113556.1.4.330|
|`lDAPAdminLimits`|LDAP-Admin-Limits|1.2.840.113556.1.4.843|
|`lDAPDisplayName`|LDAP-Display-Name|1.2.840.113556.1.2.460|
|`lDAPIPDenyList`|LDAP-IPDeny-List|1.2.840.113556.1.4.844|
|`legacyExchangeDN`|Legacy-Exchange-DN|1.2.840.113556.1.4.655|
|`linkID`|Link-ID|1.2.840.113556.1.2.50|
|`linkTrackSecret`|Link-Track-Secret|1.2.840.113556.1.4.269|
|`lmPwdHistory`|Lm-Pwd-History|1.2.840.113556.1.4.160|
|`localeID`|Locale-ID|1.2.840.113556.1.4.58|
|`localizationDisplayId`|Localization-Display-Id|1.2.840.113556.1.4.1353|
|`localizedDescription`|Localized-Description|1.2.840.113556.1.4.817|
|`localPolicyFlags`|Local-Policy-Flags|1.2.840.113556.1.4.56|
|`localPolicyReference`|Local-Policy-Reference|1.2.840.113556.1.4.457|
|`location`|Location|1.2.840.113556.1.4.222|
|`lockoutDuration`|Lockout-Duration|1.2.840.113556.1.4.60|
|`lockOutObservationWindow`|Lock-Out-Observation-Window|1.2.840.113556.1.4.61|
|`lockoutThreshold`|Lockout-Threshold|1.2.840.113556.1.4.73|
|`lockoutTime`|Lockout-Time|1.2.840.113556.1.4.662|
|`logonCount`|Logon-Count|1.2.840.113556.1.4.169|
|`logonHours`|Logon-Hours|1.2.840.113556.1.4.64|
|`logonWorkstation`|Logon-Workstation|1.2.840.113556.1.4.65|
|`lSACreationTime`|LSA-Creation-Time|1.2.840.113556.1.4.66|
|`lSAModifiedCount`|LSA-Modified-Count|1.2.840.113556.1.4.67|
|`machineArchitecture`|Machine-Architecture|1.2.840.113556.1.4.68|
|`machinePasswordChangeInterval`|Machine-Password-Change-Interval|1.2.840.113556.1.4.520|
|`machineRole`|Machine-Role|1.2.840.113556.1.4.71|
|`machineWidePolicy`|Machine-Wide-Policy|1.2.840.113556.1.4.459|
|`mail`|E-mail-Addresses|0.9.2342.19200300.100.1.3|
|`mailAddress`|SMTP-Mail-Address|1.2.840.113556.1.4.786|
|`managedBy`|Managed-By|1.2.840.113556.1.4.653|
|`managedObjects`|Managed-Objects|1.2.840.113556.1.4.654|
|`manager`|Manager|0.9.2342.19200300.100.1.10|
|`mAPIID`|MAPI-ID|1.2.840.113556.1.2.49|
|`marshalledInterface`|Marshalled-Interface|1.2.840.113556.1.4.72|
|`masteredBy`|Mastered-By|1.2.840.113556.1.4.1409|
|`maxPwdAge`|Max-Pwd-Age|1.2.840.113556.1.4.74|
|`maxRenewAge`|Max-Renew-Age|1.2.840.113556.1.4.75|
|`maxStorage`|Max-Storage|1.2.840.113556.1.4.76|
|`maxTicketAge`|Max-Ticket-Age|1.2.840.113556.1.4.77|
|`mayContain`|May-Contain|1.2.840.113556.1.2.25|
|`meetingAdvertiseScope`|meetingAdvertiseScope|1.2.840.113556.1.4.582|
|`meetingApplication`|meetingApplication|1.2.840.113556.1.4.573|
|`meetingBandwidth`|meetingBandwidth|1.2.840.113556.1.4.589|
|`meetingBlob`|meetingBlob|1.2.840.113556.1.4.590|
|`meetingContactInfo`|meetingContactInfo|1.2.840.113556.1.4.578|
|`meetingDescription`|meetingDescription|1.2.840.113556.1.4.567|
|`meetingEndTime`|meetingEndTime|1.2.840.113556.1.4.588|
|`meetingID`|meetingID|1.2.840.113556.1.4.565|
|`meetingIP`|meetingIP|1.2.840.113556.1.4.580|
|`meetingIsEncrypted`|meetingIsEncrypted|1.2.840.113556.1.4.585|
|`meetingKeyword`|meetingKeyword|1.2.840.113556.1.4.568|
|`meetingLanguage`|meetingLanguage|1.2.840.113556.1.4.574|
|`meetingLocation`|meetingLocation|1.2.840.113556.1.4.569|
|`meetingMaxParticipants`|meetingMaxParticipants|1.2.840.113556.1.4.576|
|`meetingName`|meetingName|1.2.840.113556.1.4.566|
|`meetingOriginator`|meetingOriginator|1.2.840.113556.1.4.577|
|`meetingOwner`|meetingOwner|1.2.840.113556.1.4.579|
|`meetingProtocol`|meetingProtocol|1.2.840.113556.1.4.570|
|`meetingRating`|meetingRating|1.2.840.113556.1.4.584|
|`meetingRecurrence`|meetingRecurrence|1.2.840.113556.1.4.586|
|`meetingScope`|meetingScope|1.2.840.113556.1.4.581|
|`meetingStartTime`|meetingStartTime|1.2.840.113556.1.4.587|
|`meetingType`|meetingType|1.2.840.113556.1.4.571|
|`meetingURL`|meetingURL|1.2.840.113556.1.4.583|
|`member`|Member|2.5.4.31|
|`memberOf`|Is-Member-Of-DL|1.2.840.113556.1.2.102|
|`mhsORAddress`|MHS-OR-Address|1.2.840.113556.1.4.650|
|`middleName`|Other-Name|2.16.840.1.113730.3.1.34|
|`minPwdAge`|Min-Pwd-Age|1.2.840.113556.1.4.78|
|`minPwdLength`|Min-Pwd-Length|1.2.840.113556.1.4.79|
|`minTicketAge`|Min-Ticket-Age|1.2.840.113556.1.4.80|
|`mobile`|Phone-Mobile-Primary|0.9.2342.19200300.100.1.41|
|`modifiedCount`|Modified-Count|1.2.840.113556.1.4.168|
|`modifiedCountAtLastProm`|Modified-Count-At-Last-Prom|1.2.840.113556.1.4.81|
|`modifyTimeStamp`|Modify-Time-Stamp|2.5.18.2|
|`moniker`|Moniker|1.2.840.113556.1.4.82|
|`monikerDisplayName`|Moniker-Display-Name|1.2.840.113556.1.4.83|
|`moveTreeState`|Move-Tree-State|1.2.840.113556.1.4.1305|
|`mscopeId`|Mscope-Id|1.2.840.113556.1.4.716|
|`mS-DS-ConsistencyChildCount`|MS-DS-Consistency-Child-Count|1.2.840.113556.1.4.1361|
|`mS-DS-ConsistencyGuid`|MS-DS-Consistency-Guid|1.2.840.113556.1.4.1360|
|`mS-DS-CreatorSID`|MS-DS-Creator-SID|1.2.840.113556.1.4.1410|
|`ms-DS-MachineAccountQuota`|MS-DS-Machine-Account-Quota|1.2.840.113556.1.4.1411|
|`mS-DS-ReplicatesNCReason`|MS-DS-Replicates-NC-Reason|1.2.840.113556.1.4.1408|
|`msiFileList`|Msi-File-List|1.2.840.113556.1.4.671|
|`msiScript`|Msi-Script|1.2.840.113556.1.4.814|
|`msiScriptName`|Msi-Script-Name|1.2.840.113556.1.4.845|
|`msiScriptPath`|Msi-Script-Path|1.2.840.113556.1.4.15|
|`msiScriptSize`|Msi-Script-Size|1.2.840.113556.1.4.846|
|`mSMQAuthenticate`|MSMQ-Authenticate|1.2.840.113556.1.4.923|
|`mSMQBasePriority`|MSMQ-Base-Priority|1.2.840.113556.1.4.920|
|`mSMQComputerType`|MSMQ-Computer-Type|1.2.840.113556.1.4.933|
|`mSMQComputerTypeEx`|MSMQ-Computer-Type-Ex|1.2.840.113556.1.4.1417|
|`mSMQCost`|MSMQ-Cost|1.2.840.113556.1.4.946|
|`mSMQCSPName`|MSMQ-CSP-Name|1.2.840.113556.1.4.940|
|`mSMQDependentClientService`|MSMQ-Dependent-Client-Service|1.2.840.113556.1.4.1239|
|`mSMQDependentClientServices`|MSMQ-Dependent-Client-Services|1.2.840.113556.1.4.1226|
|`mSMQDigests`|MSMQ-Digests|1.2.840.113556.1.4.948|
|`mSMQDigestsMig`|MSMQ-Digests-Mig|1.2.840.113556.1.4.966|
|`mSMQDsService`|MSMQ-Ds-Service|1.2.840.113556.1.4.1238|
|`mSMQDsServices`|MSMQ-Ds-Services|1.2.840.113556.1.4.1228|
|`mSMQEncryptKey`|MSMQ-Encrypt-Key|1.2.840.113556.1.4.936|
|`mSMQForeign`|MSMQ-Foreign|1.2.840.113556.1.4.934|
|`mSMQInRoutingServers`|MSMQ-In-Routing-Servers|1.2.840.113556.1.4.929|
|`mSMQInterval1`|MSMQ-Interval1|1.2.840.113556.1.4.1308|
|`mSMQInterval2`|MSMQ-Interval2|1.2.840.113556.1.4.1309|
|`mSMQJournal`|MSMQ-Journal|1.2.840.113556.1.4.918|
|`mSMQJournalQuota`|MSMQ-Journal-Quota|1.2.840.113556.1.4.921|
|`mSMQLabel`|MSMQ-Label|1.2.840.113556.1.4.922|
|`mSMQLabelEx`|MSMQ-Label-Ex|1.2.840.113556.1.4.1415|
|`mSMQLongLived`|MSMQ-Long-Lived|1.2.840.113556.1.4.941|
|`mSMQMigrated`|MSMQ-Migrated|1.2.840.113556.1.4.952|
|`mSMQNameStyle`|MSMQ-Name-Style|1.2.840.113556.1.4.939|
|`mSMQNt4Flags`|MSMQ-Nt4-Flags|1.2.840.113556.1.4.964|
|`mSMQNt4Stub`|MSMQ-Nt4-Stub|1.2.840.113556.1.4.960|
|`mSMQOSType`|MSMQ-OS-Type|1.2.840.113556.1.4.935|
|`mSMQOutRoutingServers`|MSMQ-Out-Routing-Servers|1.2.840.113556.1.4.928|
|`mSMQOwnerID`|MSMQ-Owner-ID|1.2.840.113556.1.4.925|
|`mSMQPrevSiteGates`|MSMQ-Prev-Site-Gates|1.2.840.113556.1.4.1225|
|`mSMQPrivacyLevel`|MSMQ-Privacy-Level|1.2.840.113556.1.4.924|
|`mSMQQMID`|MSMQ-QM-ID|1.2.840.113556.1.4.951|
|`mSMQQueueJournalQuota`|MSMQ-Queue-Journal-Quota|1.2.840.113556.1.4.963|
|`mSMQQueueNameExt`|MSMQ-Queue-Name-Ext|1.2.840.113556.1.4.1243|
|`mSMQQueueQuota`|MSMQ-Queue-Quota|1.2.840.113556.1.4.962|
|`mSMQQueueType`|MSMQ-Queue-Type|1.2.840.113556.1.4.917|
|`mSMQQuota`|MSMQ-Quota|1.2.840.113556.1.4.919|
|`mSMQRoutingService`|MSMQ-Routing-Service|1.2.840.113556.1.4.1237|
|`mSMQRoutingServices`|MSMQ-Routing-Services|1.2.840.113556.1.4.1227|
|`mSMQServices`|MSMQ-Services|1.2.840.113556.1.4.950|
|`mSMQServiceType`|MSMQ-Service-Type|1.2.840.113556.1.4.930|
|`mSMQSignCertificates`|MSMQ-Sign-Certificates|1.2.840.113556.1.4.947|
|`mSMQSignCertificatesMig`|MSMQ-Sign-Certificates-Mig|1.2.840.113556.1.4.967|
|`mSMQSignKey`|MSMQ-Sign-Key|1.2.840.113556.1.4.937|
|`mSMQSite1`|MSMQ-Site-1|1.2.840.113556.1.4.943|
|`mSMQSite2`|MSMQ-Site-2|1.2.840.113556.1.4.944|
|`mSMQSiteForeign`|MSMQ-Site-Foreign|1.2.840.113556.1.4.961|
|`mSMQSiteGates`|MSMQ-Site-Gates|1.2.840.113556.1.4.945|
|`mSMQSiteGatesMig`|MSMQ-Site-Gates-Mig|1.2.840.113556.1.4.1310|
|`mSMQSiteID`|MSMQ-Site-ID|1.2.840.113556.1.4.953|
|`mSMQSiteName`|MSMQ-Site-Name|1.2.840.113556.1.4.965|
|`mSMQSiteNameEx`|MSMQ-Site-Name-Ex|1.2.840.113556.1.4.1416|
|`mSMQSites`|MSMQ-Sites|1.2.840.113556.1.4.927|
|`mSMQTransactional`|MSMQ-Transactional|1.2.840.113556.1.4.926|
|`mSMQUserSid`|MSMQ-User-Sid|1.2.840.113556.1.4.1337|
|`mSMQVersion`|MSMQ-Version|1.2.840.113556.1.4.942|
|`msNPAllowDialin`|msNPAllowDialin|1.2.840.113556.1.4.1119|
|`msNPCalledStationID`|msNPCalledStationID|1.2.840.113556.1.4.1123|
|`msNPCallingStationID`|msNPCallingStationID|1.2.840.113556.1.4.1124|
|`msNPSavedCallingStationID`|msNPSavedCallingStationID|1.2.840.113556.1.4.1130|
|`msRADIUSCallbackNumber`|msRADIUSCallbackNumber|1.2.840.113556.1.4.1145|
|`msRADIUSFramedIPAddress`|msRADIUSFramedIPAddress|1.2.840.113556.1.4.1153|
|`msRADIUSFramedRoute`|msRADIUSFramedRoute|1.2.840.113556.1.4.1158|
|`msRADIUSServiceType`|msRADIUSServiceType|1.2.840.113556.1.4.1171|
|`msRASSavedCallbackNumber`|msRASSavedCallbackNumber|1.2.840.113556.1.4.1189|
|`msRASSavedFramedIPAddress`|msRASSavedFramedIPAddress|1.2.840.113556.1.4.1190|
|`msRASSavedFramedRoute`|msRASSavedFramedRoute|1.2.840.113556.1.4.1191|
|`msRRASAttribute`|ms-RRAS-Attribute|1.2.840.113556.1.4.884|
|`msRRASVendorAttributeEntry`|ms-RRAS-Vendor-Attribute-Entry|1.2.840.113556.1.4.883|
|`mS-SQL-Alias`|MS-SQL-Alias|1.2.840.113556.1.4.1395|
|`mS-SQL-AllowAnonymousSubscription`|MS-SQL-AllowAnonymousSubscription|1.2.840.113556.1.4.1394|
|`mS-SQL-AllowImmediateUpdatingSubscription`|MS-SQL-AllowImmediateUpdatingSubscription|1.2.840.113556.1.4.1404|
|`mS-SQL-AllowKnownPullSubscription`|MS-SQL-AllowKnownPullSubscription|1.2.840.113556.1.4.1403|
|`mS-SQL-AllowQueuedUpdatingSubscription`|MS-SQL-AllowQueuedUpdatingSubscription|1.2.840.113556.1.4.1405|
|`mS-SQL-AllowSnapshotFilesFTPDownloading`|MS-SQL-AllowSnapshotFilesFTPDownloading|1.2.840.113556.1.4.1406|
|`mS-SQL-AppleTalk`|MS-SQL-AppleTalk|1.2.840.113556.1.4.1378|
|`mS-SQL-Applications`|MS-SQL-Applications|1.2.840.113556.1.4.1400|
|`mS-SQL-Build`|MS-SQL-Build|1.2.840.113556.1.4.1368|
|`mS-SQL-CharacterSet`|MS-SQL-CharacterSet|1.2.840.113556.1.4.1370|
|`mS-SQL-Clustered`|MS-SQL-Clustered|1.2.840.113556.1.4.1373|
|`mS-SQL-ConnectionURL`|MS-SQL-ConnectionURL|1.2.840.113556.1.4.1383|
|`mS-SQL-Contact`|MS-SQL-Contact|1.2.840.113556.1.4.1365|
|`mS-SQL-CreationDate`|MS-SQL-CreationDate|1.2.840.113556.1.4.1397|
|`mS-SQL-Database`|MS-SQL-Database|1.2.840.113556.1.4.1393|
|`mS-SQL-Description`|MS-SQL-Description|1.2.840.113556.1.4.1390|
|`mS-SQL-GPSHeight`|MS-SQL-GPSHeight|1.2.840.113556.1.4.1387|
|`mS-SQL-GPSLatitude`|MS-SQL-GPSLatitude|1.2.840.113556.1.4.1385|
|`mS-SQL-GPSLongitude`|MS-SQL-GPSLongitude|1.2.840.113556.1.4.1386|
|`mS-SQL-InformationDirectory`|MS-SQL-InformationDirectory|1.2.840.113556.1.4.1392|
|`mS-SQL-InformationURL`|MS-SQL-InformationURL|1.2.840.113556.1.4.1382|
|`mS-SQL-Keywords`|MS-SQL-Keywords|1.2.840.113556.1.4.1401|
|`mS-SQL-Language`|MS-SQL-Language|1.2.840.113556.1.4.1389|
|`mS-SQL-LastBackupDate`|MS-SQL-LastBackupDate|1.2.840.113556.1.4.1398|
|`mS-SQL-LastDiagnosticDate`|MS-SQL-LastDiagnosticDate|1.2.840.113556.1.4.1399|
|`mS-SQL-LastUpdatedDate`|MS-SQL-LastUpdatedDate|1.2.840.113556.1.4.1381|
|`mS-SQL-Location`|MS-SQL-Location|1.2.840.113556.1.4.1366|
|`mS-SQL-Memory`|MS-SQL-Memory|1.2.840.113556.1.4.1367|
|`mS-SQL-MultiProtocol`|MS-SQL-MultiProtocol|1.2.840.113556.1.4.1375|
|`mS-SQL-Name`|MS-SQL-Name|1.2.840.113556.1.4.1363|
|`mS-SQL-NamedPipe`|MS-SQL-NamedPipe|1.2.840.113556.1.4.1374|
|`mS-SQL-PublicationURL`|MS-SQL-PublicationURL|1.2.840.113556.1.4.1384|
|`mS-SQL-Publisher`|MS-SQL-Publisher|1.2.840.113556.1.4.1402|
|`mS-SQL-RegisteredOwner`|MS-SQL-RegisteredOwner|1.2.840.113556.1.4.1364|
|`mS-SQL-ServiceAccount`|MS-SQL-ServiceAccount|1.2.840.113556.1.4.1369|
|`mS-SQL-Size`|MS-SQL-Size|1.2.840.113556.1.4.1396|
|`mS-SQL-SortOrder`|MS-SQL-SortOrder|1.2.840.113556.1.4.1371|
|`mS-SQL-SPX`|MS-SQL-SPX|1.2.840.113556.1.4.1376|
|`mS-SQL-Status`|MS-SQL-Status|1.2.840.113556.1.4.1380|
|`mS-SQL-TCPIP`|MS-SQL-TCPIP|1.2.840.113556.1.4.1377|
|`mS-SQL-ThirdParty`|MS-SQL-ThirdParty|1.2.840.113556.1.4.1407|
|`mS-SQL-Type`|MS-SQL-Type|1.2.840.113556.1.4.1391|
|`mS-SQL-UnicodeSortOrder`|MS-SQL-UnicodeSortOrder|1.2.840.113556.1.4.1372|
|`mS-SQL-Version`|MS-SQL-Version|1.2.840.113556.1.4.1388|
|`mS-SQL-Vines`|MS-SQL-Vines|1.2.840.113556.1.4.1379|
|`mustContain`|Must-Contain|1.2.840.113556.1.2.24|
|`name`|RDN|1.2.840.113556.1.4.1|
|`nameServiceFlags`|Name-Service-Flags|1.2.840.113556.1.4.753|
|`nCName`|NC-Name|1.2.840.113556.1.2.16|
|`nETBIOSName`|NETBIOS-Name|1.2.840.113556.1.4.87|
|`netbootAllowNewClients`|netboot-Allow-New-Clients|1.2.840.113556.1.4.849|
|`netbootAnswerOnlyValidClients`|netboot-Answer-Only-Valid-Clients|1.2.840.113556.1.4.854|
|`netbootAnswerRequests`|netboot-Answer-Requests|1.2.840.113556.1.4.853|
|`netbootCurrentClientCount`|netboot-Current-Client-Count|1.2.840.113556.1.4.852|
|`netbootGUID`|Netboot-GUID|1.2.840.113556.1.4.359|
|`netbootInitialization`|Netboot-Initialization|1.2.840.113556.1.4.358|
|`netbootIntelliMirrorOSes`|netboot-IntelliMirror-OSes|1.2.840.113556.1.4.857|
|`netbootLimitClients`|netboot-Limit-Clients|1.2.840.113556.1.4.850|
|`netbootLocallyInstalledOSes`|netboot-Locally-Installed-OSes|1.2.840.113556.1.4.859|
|`netbootMachineFilePath`|Netboot-Machine-File-Path|1.2.840.113556.1.4.361|
|`netbootMaxClients`|netboot-Max-Clients|1.2.840.113556.1.4.851|
|`netbootMirrorDataFile`|Netboot-Mirror-Data-File|1.2.840.113556.1.4.1241|
|`netbootNewMachineNamingPolicy`|netboot-New-Machine-Naming-Policy|1.2.840.113556.1.4.855|
|`netbootNewMachineOU`|netboot-New-Machine-OU|1.2.840.113556.1.4.856|
|`netbootSCPBL`|netboot-SCP-BL|1.2.840.113556.1.4.864|
|`netbootServer`|netboot-Server|1.2.840.113556.1.4.860|
|`netbootSIFFile`|Netboot-SIF-File|1.2.840.113556.1.4.1240|
|`netbootTools`|netboot-Tools|1.2.840.113556.1.4.858|
|`networkAddress`|Network-Address|1.2.840.113556.1.2.459|
|`nextLevelStore`|Next-Level-Store|1.2.840.113556.1.4.214|
|`nextRid`|Next-Rid|1.2.840.113556.1.4.88|
|`nonSecurityMember`|Non-Security-Member|1.2.840.113556.1.4.530|
|`nonSecurityMemberBL`|Non-Security-Member-BL|1.2.840.113556.1.4.531|
|`notes`|Additional-Information|1.2.840.113556.1.4.265|
|`notificationList`|Notification-List|1.2.840.113556.1.4.303|
|`nTGroupMembers`|NT-Group-Members|1.2.840.113556.1.4.89|
|`nTMixedDomain`|NT-Mixed-Domain|1.2.840.113556.1.4.357|
|`ntPwdHistory`|Nt-Pwd-History|1.2.840.113556.1.4.94|
|`nTSecurityDescriptor`|NT-Security-Descriptor|1.2.840.113556.1.2.281|
|`o`|Organization-Name|2.5.4.10|
|`objectCategory`|Object-Category|1.2.840.113556.1.4.782|
|`objectClass`|Object-Class|2.5.4.0|
|`objectClassCategory`|Object-Class-Category|1.2.840.113556.1.2.370|
|`objectClasses`|Object-Classes|2.5.21.6|
|`objectCount`|Object-Count|1.2.840.113556.1.4.506|
|`objectGUID`|Object-Guid|1.2.840.113556.1.4.2|
|`objectSid`|Object-Sid|1.2.840.113556.1.4.146|
|`objectVersion`|Object-Version|1.2.840.113556.1.2.76|
|`oEMInformation`|OEM-Information|1.2.840.113556.1.4.151|
|`oMObjectClass`|OM-Object-Class|1.2.840.113556.1.2.218|
|`oMSyntax`|OM-Syntax|1.2.840.113556.1.2.231|
|`oMTGuid`|OMT-Guid|1.2.840.113556.1.4.505|
|`oMTIndxGuid`|OMT-Indx-Guid|1.2.840.113556.1.4.333|
|`operatingSystem`|Operating-System|1.2.840.113556.1.4.363|
|`operatingSystemHotfix`|Operating-System-Hotfix|1.2.840.113556.1.4.415|
|`operatingSystemServicePack`|Operating-System-Service-Pack|1.2.840.113556.1.4.365|
|`operatingSystemVersion`|Operating-System-Version|1.2.840.113556.1.4.364|
|`operatorCount`|Operator-Count|1.2.840.113556.1.4.144|
|`optionDescription`|Option-Description|1.2.840.113556.1.4.712|
|`options`|Options|1.2.840.113556.1.4.307|
|`optionsLocation`|Options-Location|1.2.840.113556.1.4.713|
|`originalDisplayTable`|Original-Display-Table|1.2.840.113556.1.2.445|
|`originalDisplayTableMSDOS`|Original-Display-Table-MSDOS|1.2.840.113556.1.2.214|
|`otherFacsimileTelephoneNumber`|Phone-Fax-Other|1.2.840.113556.1.4.646|
|`otherHomePhone`|Phone-Home-Other|1.2.840.113556.1.2.277|
|`otherIpPhone`|Phone-Ip-Other|1.2.840.113556.1.4.722|
|`otherLoginWorkstations`|Other-Login-Workstations|1.2.840.113556.1.4.91|
|`otherMailbox`|Other-Mailbox|1.2.840.113556.1.4.651|
|`otherMobile`|Phone-Mobile-Other|1.2.840.113556.1.4.647|
|`otherPager`|Phone-Pager-Other|1.2.840.113556.1.2.118|
|`otherTelephone`|Phone-Office-Other|1.2.840.113556.1.2.18|
|`otherWellKnownObjects`|Other-Well-Known-Objects|1.2.840.113556.1.4.1359|
|`ou`|Organizational-Unit-Name|2.5.4.11|
|`owner`|Owner|2.5.4.32|
|`packageFlags`|Package-Flags|1.2.840.113556.1.4.327|
|`packageName`|Package-Name|1.2.840.113556.1.4.326|
|`packageType`|Package-Type|1.2.840.113556.1.4.324|
|`pager`|Phone-Pager-Primary|0.9.2342.19200300.100.1.42|
|`parentCA`|Parent-CA|1.2.840.113556.1.4.557|
|`parentCACertificateChain`|Parent-CA-Certificate-Chain|1.2.840.113556.1.4.685|
|`parentGUID`|Parent-GUID|1.2.840.113556.1.4.1224|
|`partialAttributeDeletionList`|Partial-Attribute-Deletion-List|1.2.840.113556.1.4.663|
|`partialAttributeSet`|Partial-Attribute-Set|1.2.840.113556.1.4.640|
|`pekKeyChangeInterval`|Pek-Key-Change-Interval|1.2.840.113556.1.4.866|
|`pekList`|Pek-List|1.2.840.113556.1.4.865|
|`pendingCACertificates`|Pending-CA-Certificates|1.2.840.113556.1.4.693|
|`pendingParentCA`|Pending-Parent-CA|1.2.840.113556.1.4.695|
|`perMsgDialogDisplayTable`|Per-Msg-Dialog-Display-Table|1.2.840.113556.1.2.325|
|`perRecipDialogDisplayTable`|Per-Recip-Dialog-Display-Table|1.2.840.113556.1.2.326|
|`personalTitle`|Personal-Title|1.2.840.113556.1.2.615|
|`physicalDeliveryOfficeName`|Physical-Delivery-Office-Name|2.5.4.19|
|`physicalLocationObject`|Physical-Location-Object|1.2.840.113556.1.4.514|
|`pKICriticalExtensions`|PKI-Critical-Extensions|1.2.840.113556.1.4.1330|
|`pKIDefaultCSPs`|PKI-Default-CSPs|1.2.840.113556.1.4.1334|
|`pKIDefaultKeySpec`|PKI-Default-Key-Spec|1.2.840.113556.1.4.1327|
|`pKIEnrollmentAccess`|PKI-Enrollment-Access|1.2.840.113556.1.4.1335|
|`pKIExpirationPeriod`|PKI-Expiration-Period|1.2.840.113556.1.4.1331|
|`pKIExtendedKeyUsage`|PKI-Extended-Key-Usage|1.2.840.113556.1.4.1333|
|`pKIKeyUsage`|PKI-Key-Usage|1.2.840.113556.1.4.1328|
|`pKIMaxIssuingDepth`|PKI-Max-Issuing-Depth|1.2.840.113556.1.4.1329|
|`pKIOverlapPeriod`|PKI-Overlap-Period|1.2.840.113556.1.4.1332|
|`pKT`|PKT|1.2.840.113556.1.4.206|
|`pKTGuid`|PKT-Guid|1.2.840.113556.1.4.205|
|`policyReplicationFlags`|Policy-Replication-Flags|1.2.840.113556.1.4.633|
|`portName`|Port-Name|1.2.840.113556.1.4.228|
|`possibleInferiors`|Possible-Inferiors|1.2.840.113556.1.4.915|
|`possSuperiors`|Poss-Superiors|1.2.840.113556.1.2.8|
|`postalAddress`|Postal-Address|2.5.4.16|
|`postalCode`|Postal-Code|2.5.4.17|
|`postOfficeBox`|Post-Office-Box|2.5.4.18|
|`preferredDeliveryMethod`|Preferred-Delivery-Method|2.5.4.28|
|`preferredOU`|Preferred-OU|1.2.840.113556.1.4.97|
|`prefixMap`|Prefix-Map|1.2.840.113556.1.4.538|
|`presentationAddress`|Presentation-Address|2.5.4.29|
|`previousCACertificates`|Previous-CA-Certificates|1.2.840.113556.1.4.692|
|`previousParentCA`|Previous-Parent-CA|1.2.840.113556.1.4.694|
|`primaryGroupID`|Primary-Group-ID|1.2.840.113556.1.4.98|
|`primaryGroupToken`|Primary-Group-Token|1.2.840.113556.1.4.1412|
|`primaryInternationalISDNNumber`|Phone-ISDN-Primary|1.2.840.113556.1.4.649|
|`primaryTelexNumber`|Telex-Primary|1.2.840.113556.1.4.648|
|`printAttributes`|Print-Attributes|1.2.840.113556.1.4.247|
|`printBinNames`|Print-Bin-Names|1.2.840.113556.1.4.237|
|`printCollate`|Print-Collate|1.2.840.113556.1.4.242|
|`printColor`|Print-Color|1.2.840.113556.1.4.243|
|`printDuplexSupported`|Print-Duplex-Supported|1.2.840.113556.1.4.1311|
|`printEndTime`|Print-End-Time|1.2.840.113556.1.4.234|
|`printerName`|Printer-Name|1.2.840.113556.1.4.300|
|`printFormName`|Print-Form-Name|1.2.840.113556.1.4.235|
|`printKeepPrintedJobs`|Print-Keep-Printed-Jobs|1.2.840.113556.1.4.275|
|`printLanguage`|Print-Language|1.2.840.113556.1.4.246|
|`printMACAddress`|Print-MAC-Address|1.2.840.113556.1.4.288|
|`printMaxCopies`|Print-Max-Copies|1.2.840.113556.1.4.241|
|`printMaxResolutionSupported`|Print-Max-Resolution-Supported|1.2.840.113556.1.4.238|
|`printMaxXExtent`|Print-Max-X-Extent|1.2.840.113556.1.4.277|
|`printMaxYExtent`|Print-Max-Y-Extent|1.2.840.113556.1.4.278|
|`printMediaReady`|Print-Media-Ready|1.2.840.113556.1.4.289|
|`printMediaSupported`|Print-Media-Supported|1.2.840.113556.1.4.299|
|`printMemory`|Print-Memory|1.2.840.113556.1.4.282|
|`printMinXExtent`|Print-Min-X-Extent|1.2.840.113556.1.4.279|
|`printMinYExtent`|Print-Min-Y-Extent|1.2.840.113556.1.4.280|
|`printNetworkAddress`|Print-Network-Address|1.2.840.113556.1.4.287|
|`printNotify`|Print-Notify|1.2.840.113556.1.4.272|
|`printNumberUp`|Print-Number-Up|1.2.840.113556.1.4.290|
|`printOrientationsSupported`|Print-Orientations-Supported|1.2.840.113556.1.4.240|
|`printOwner`|Print-Owner|1.2.840.113556.1.4.271|
|`printPagesPerMinute`|Print-Pages-Per-Minute|1.2.840.113556.1.4.631|
|`printRate`|Print-Rate|1.2.840.113556.1.4.285|
|`printRateUnit`|Print-Rate-Unit|1.2.840.113556.1.4.286|
|`printSeparatorFile`|Print-Separator-File|1.2.840.113556.1.4.230|
|`printShareName`|Print-Share-Name|1.2.840.113556.1.4.270|
|`printSpooling`|Print-Spooling|1.2.840.113556.1.4.274|
|`printStaplingSupported`|Print-Stapling-Supported|1.2.840.113556.1.4.281|
|`printStartTime`|Print-Start-Time|1.2.840.113556.1.4.233|
|`printStatus`|Print-Status|1.2.840.113556.1.4.273|
|`priority`|Priority|1.2.840.113556.1.4.231|
|`priorSetTime`|Prior-Set-Time|1.2.840.113556.1.4.99|
|`priorValue`|Prior-Value|1.2.840.113556.1.4.100|

---

## Object Identifiers (OIDs)
We can also use matching rule [Object Identifiers (OIDs)](https://ldapwiki.com/wiki/Wiki.jsp?page=OID) with LDAP filters as listed in this [Search Filter Syntax](https://docs.microsoft.com/en-us/windows/win32/adsi/search-filter-syntax) document from Microsoft:

|**Matching rule OID**|**String identifier**|**Description**|
|---|---|---|
|[1.2.840.113556.1.4.803](https://ldapwiki.com/wiki/Wiki.jsp?page=1.2.840.113556.1.4.803)|LDAP_MATCHING_RULE_BIT_AND|A match is found only if all bits from the attribute match the value. This rule is equivalent to a bitwise **AND** operator.|
|[1.2.840.113556.1.4.804](https://ldapwiki.com/wiki/Wiki.jsp?page=1.2.840.113556.1.4.804)|LDAP_MATCHING_RULE_BIT_OR|A match is found if any bits from the attribute match the value. This rule is equivalent to a bitwise **OR** operator.|
|[1.2.840.113556.1.4.1941](https://ldapwiki.com/wiki/Wiki.jsp?page=1.2.840.113556.1.4.1941)|LDAP_MATCHING_RULE_IN_CHAIN|This rule is limited to filters that apply to the DN. This is a special "extended" match operator that walks the chain of ancestry in objects all the way to the root until it finds a match.|

We can clarify the above OIDs with some examples. Let's take the following LDAP query:

```powershell-session
(&(objectCategory=person)(objectClass=user)(userAccountControl:1.2.840.113556.1.4.803:=2)) 
```

This query will return all administratively disabled user accounts, or [ACCOUNTDISABLE (2)](https://ldapwiki.com/wiki/Wiki.jsp?page=ACCOUNTDISABLE). We can combine this query as an LDAP search filter with the "`Get-ADUser`" cmdlet against our target domain. The LDAP query can be shortened as follows:

#### LDAP Query - Filter Disabled User Accounts

```powershell-session
PS C:\htb> Get-ADUser -LDAPFilter '(userAccountControl:1.2.840.113556.1.4.803:=2)' | select name

name
----
Guest
DefaultAccount
krbtgt
Exchange Online-ApplicationAccount
SystemMailbox{1f05a927-35b9-4cc9-bbe1-11e28cddb180}
SystemMailbox{bb558c35-97f1-4cb9-8ff7-d53741dc928c}
SystemMailbox{e0dc1c29-89c3-4034-b678-e6c29d823ed9}
DiscoverySearchMailbox {D919BA05-46A6-415f-80AD-7E09334BB852}
Migration.8f3e7716-2011-43e4-96b1-aba62d229136
FederatedEmail.4c1f4d8b-8179-4148-93bf-00a95fa1e042
SystemMailbox{D0E409A0-AF9B-4720-92FE-AAC869B0D201}
SystemMailbox{2CE34405-31BE-455D-89D7-A7C7DA7A0DAA}
SystemMailbox{8cc370d3-822a-4ab8-a926-bb94bd0641a9}
```

Now let's look at an example of the extensible match rule "`1.2.840.113556.1.4.1941`". Consider the following query:

```powershell-session
(member:1.2.840.113556.1.4.1941:=CN=Harry Jones,OU=Network Ops,OU=IT,OU=Employees,DC=INLANEFREIGHT,DC=LOCAL)
```

Now let's look at an example of the extensible match rule "`1.2.840.113556.1.4.1941`". Consider the following query:

```powershell-session
(member:1.2.840.113556.1.4.1941:=CN=Harry Jones,OU=Network Ops,OU=IT,OU=Employees,DC=INLANEFREIGHT,DC=LOCAL)
```

This matching rule will find all groups that the user `Harry Jones` ("`CN=Harry Jones,OU=Network Ops,OU=IT,OU=Employees,DC=INLANEFREIGHT,DC=LOCAL`") is a member of. Using this filter with the "`Get-ADGroup`" cmdlet gives us the following output:

#### LDAP Query - Find All Groups

```powershell-session
PS C:\htb> Get-ADGroup -LDAPFilter '(member:1.2.840.113556.1.4.1941:=CN=Harry Jones,OU=Network Ops,OU=IT,OU=Employees,DC=INLANEFREIGHT,DC=LOCAL)' | select Name

Name
----
Administrators
Backup Operators
Domain Admins
Denied RODC Password Replication Group
LAPS Admins
Security Operations
Help Desk
Network Team
```

---

## Filter Types, Item Types & Escaped Characters
With LDAP search [filters](https://ldapwiki.com/wiki/Wiki.jsp?page=LDAP%20SearchFilters), we have the following four filter types:

|**Operator**|**Meaning**|
|---|---|
|=|Equal to|
|~=|Approximately equal to|
|>=|Greater than or equal to|
|<=|Less than or equal to|

---

And we have four item types:

|**Type**|**Meaning**|
|---|---|
|=|Simple|
|=*|Present|
|=something*|Substring|
|Extensible|varies depending on type|

---

Finally, the following characters must be escaped if used in an LDAP filter:

|**Character**|**Represented as Hex**|
|---|---|
|*|\2a|
|(|\28|
|)|\29|
|\|\5c|
|NUL|\00|

---

## Example LDAP Filters

Let's build a few more LDAP filters to use against our test domain.

We can use the filter "`(&(objectCategory=user)(description=*))`" to find all user accounts that do not have a blank `description` field. This is a useful search that should be performed on every internal network assessment as it not uncommon to find passwords for users stored in the user description attribute in AD (which can be read by all AD users).

Combining this with the "`Get-ADUser`" cmdlet, we can search for all domain users that do not have a blank description field and, in this case, find a service account password!

#### LDAP Query - Description Field

```powershell-session
PS C:\htb> Get-ADUser -Properties * -LDAPFilter '(&(objectCategory=user)(description=*))' | select samaccountname,description

samaccountname description
-------------- -----------
Administrator  Built-in account for administering the computer/domain
Guest          Built-in account for guest access to the computer/domain
DefaultAccount A user account managed by the system.
krbtgt         Key Distribution Center Service Account
svc-sccm       **Do not change password** 03/04/2015 N3ssu$_svc2014!
```

This filter "`(userAccountControl:1.2.840.113556.1.4.803:=524288)`" can be used to find all users or computers marked as `trusted for delegation`, or unconstrained delegation, which will be covered in a later module on Kerberos Attacks. We can enumerate users with the help of this LDAP filter:

#### LDAP Query - Find Trusted Users

```powershell-session
PS C:\htb> Get-ADUser -Properties * -LDAPFilter '(userAccountControl:1.2.840.113556.1.4.803:=524288)' | select Name,memberof, servicePrincipalName,TrustedForDelegation | fl

Name                 : sqldev
memberof             : {CN=Protected Users,CN=Users,DC=INLANEFREIGHT,DC=LOCAL}
servicePrincipalName : {MSSQL_svc_dev/inlanefreight.local:1443}
TrustedForDelegation : True
```

We can enumerate computers with this setting as well:

#### LDAP Query - Find Trusted Computers

```powershell-session
PS C:\htb> Get-ADComputer -Properties * -LDAPFilter '(userAccountControl:1.2.840.113556.1.4.803:=524288)' | select DistinguishedName,servicePrincipalName,TrustedForDelegation | fl

DistinguishedName    : CN=DC01,OU=Domain Controllers,DC=INLANEFREIGHT,DC=LOCAL
servicePrincipalName : {exchangeAB/DC01, exchangeAB/DC01.INLANEFREIGHT.LOCAL, TERMSRV/DC01,
                       TERMSRV/DC01.INLANEFREIGHT.LOCAL...}
TrustedForDelegation : True

DistinguishedName    : CN=SQL01,OU=SQL Servers,OU=Servers,DC=INLANEFREIGHT,DC=LOCAL
servicePrincipalName : {MSSQLsvc/SQL01.INLANEFREIGHT.LOCAL:1433, TERMSRV/SQL01, TERMSRV/SQL01.INLANEFREIGHT.LOCAL,
                       RestrictedKrbHost/SQL01...}
TrustedForDelegation : True
```

Lastly, let's search for all users with the "`adminCount`" attribute set to `1` whose "`useraccountcontrol`" attribute is set with the flag "`PASSWD_NOTREQD`," meaning that the account can have a blank password set. To do this, we must combine two LDAP search filters as follows:

```powershell-session
(&(objectCategory=person)(objectClass=user)(userAccountControl:1.2.840.113556.1.4.803:=32))(adminCount=1)
```

#### LDAP Query - Users With Blank Password

```powershell-session
PS C:\htb> Get-AdUser -LDAPFilter '(&(objectCategory=person)(objectClass=user)(userAccountControl:1.2.840.113556.1.4.803:=32))(adminCount=1)' -Properties * | select name,memberof | fl

name     : Jenna Smith
memberof : CN=Schema Admins,CN=Users,DC=INLANEFREIGHT,DC=LOCAL

name     : Harry Jones
memberof : {CN=Network Team,CN=Users,DC=INLANEFREIGHT,DC=LOCAL, CN=Help Desk,OU=Microsoft Exchange Security
           Groups,DC=INLANEFREIGHT,DC=LOCAL, CN=Security Operations,CN=Users,DC=INLANEFREIGHT,DC=LOCAL, CN=LAPS
           Admins,CN=Users,DC=INLANEFREIGHT,DC=LOCAL...}
```

While uncommon, we find accounts without a password set from time to time, so it is always important to enumerate accounts with the `PASSWD_NOTREQD` flag set and check to see if they indeed do not have a password set. This could happen intentionally (perhaps as a timesaver) or accidentally if a user with this flag set changes their password via command line and accidentally presses enter before typing in a password. All organizations should perform periodic account audits and remove this flag from any accounts that have no valid business reason to have it set.

Try out building some filters of your own. This guide [Active Directory: LDAP Syntax Filters](https://social.technet.microsoft.com/wiki/contents/articles/5392.active-directory-ldap-syntax-filters.aspx) is a great starting point.

---

## Recursive Match
We can use the "`RecursiveMatch`" parameter in a similar way that we use the matching rule OID "`1.2.840.113556.1.4.1941`". A good example of this is to find all of the groups that an AD user is a part of, both directly and indirectly. This is also known as "nested group membership." For example, the user `bob.smith` may not be a direct member of the `Domain Admins` group but has `derivative` Domain Admin rights because the group `Security Operations` is a member of the `Domain Admins` group. We can see this graphically by looking at `Active Directory Computers and Users`.

![image](https://academy.hackthebox.com/storage/modules/22/secops_win.png)

We can enumerate this with PowerShell several ways, one way being the "`Get-ADGroupMember`" cmdlet.

#### PowerShell - Members Of Security Operations

```powershell-session
PS C:\htb> Get-ADGroupMember -Identity "Security Operations"

distinguishedName : CN=Harry Jones,OU=Network Ops,OU=IT,OU=Employees,DC=INLANEFREIGHT,DC=LOCAL
name              : Harry Jones
objectClass       : user
objectGUID        : f6d9b03e-7056-478b-a737-6c3298d18b9d
SamAccountName    : harry.jones
SID               : S-1-5-21-2974783224-3764228556-2640795941-2040
```

As we can see above, the `Security Operations` group is indeed "nested" within the `Domain Admins` group. Therefore any of its members are effectively Domain Admins.

Searching for a user's group membership using `Get-ADUser` focusing on the property `memberof` will not directly show this information.

#### PowerShell - User's Group Membership

```powershell-session
PS C:\htb> Get-ADUser -Identity harry.jones -Properties * | select memberof | ft -Wrap

memberof
--------
{CN=Network Team,CN=Users,DC=INLANEFREIGHT,DC=LOCAL, CN=Help Desk,OU=Microsoft Exchange Security
Groups,DC=INLANEFREIGHT,DC=LOCAL, CN=Security Operations,CN=Users,DC=INLANEFREIGHT,DC=LOCAL, CN=LAPS
Admins,CN=Users,DC=INLANEFREIGHT,DC=LOCAL...}
```

We can find nested group membership with the matching rule OID and the `RecursiveMatch` parameter, as seen in the following examples. The first example shows an AD filter and the `RecursiveMatch` to recursively query for all groups that the user `harry.jones` is a member of.

#### PowerShell - All Groups of User

```powershell-session
PS C:\htb> Get-ADGroup -Filter 'member -RecursiveMatch "CN=Harry Jones,OU=Network Ops,OU=IT,OU=Employees,DC=INLANEFREIGHT,DC=LOCAL"' | select name

name
----
Administrators
Backup Operators
Domain Admins
Denied RODC Password Replication Group
LAPS Admins
Security Operations
Help Desk
Network Team
```

Another way to return this same information is by using an `LDAPFilter` and the matching rule OID.

#### LDAP Query - All Groups of User

```powershell-session
PS C:\htb> Get-ADGroup -LDAPFilter '(member:1.2.840.113556.1.4.1941:=CN=Harry Jones,OU=Network Ops,OU=IT,OU=Employees,DC=INLANEFREIGHT,DC=LOCAL)' |select Name

Name
----
Administrators
Backup Operators
Domain Admins
Denied RODC Password Replication Group
LAPS Admins
Security Operations
Help Desk
Network Team
```

As shown in the above examples, searching recursively in AD can help us enumerate information that standard search queries do not show. Enumerating nested group membership is very important. We may uncover serious misconfigurations within the target AD environment that would otherwise go unnoticed, especially in large organizations with thousands of objects in AD. We will see other ways to enumerate this information and even ways of presenting it in a graphical format, but `RecursiveMatch` is a powerful search parameter that should not be overlooked.

---

## SearchBase and SearchScope Parameters

Even small Active Directory environments can contain hundreds if not thousands of objects. Active Directory can grow very quickly as users, groups, computers, OUs, etc., are added, and ACLs are set up, which creates an increasingly complex web of relationships. We may also find ourselves in a vast environment, 10-20 years old, with 10s of thousands of objects. Enumerating these environments can become an unwieldy task, so we need to refine our searches.

We can improve the performance of our enumeration commands and scripts and reduce the volume of objects returned by scoping our searches using the "`SearchBase`" parameter. This parameter specifies an Active Directory path to search under and allows us to begin searching for a user account in a specific OU. The "`SearchBase`" parameter accepts an OUs distinguished name (DN) such as `"OU=Employees,DC=INLANEFREIGHT,DC=LOCAL"`.

"`SearchScope`" allows us to define how deep into the OU hierarchy we would like to search. This parameter has three levels:

|**Name**|**Level**|**Description**|
|---|---|---|
|Base|0|The object is specified as the `SearchBase`. For example, if we ask for all users in an OU defining a base scope, we get no results. If we specify a user or use `Get-ADObject` we get just that user or object returned.|
|OneLevel|1|Searches for objects in the container defined by the `SearchBase` but not in any sub-containers.|
|SubTree|2|Searches for objects contained by the `SearchBase` and all child containers, including their children, recursively all the way down the AD hierarchy.|

When querying AD using "`SearchScope`" we can specify the name or the number (i.e., `SearchScope Onelevel` is interpreted the same as "`SearchScope 1`".)

![image](https://academy.hackthebox.com/storage/modules/22/OU_structures.png)

In the above example, with the SearchBase set to OU=Employees,DC=INLANEFREIGHT,DC=LOCAL, a `SearchScope` set to `Base` would attempt to query the OU object (`Employees`) itself. A `SearchScope` set to `OneLevel` would search within the `Employees` OU only. Finally, a `SearchScope` set to `SubTree` would query the `Employees` OU and all of the OUs underneath it, such as `Accounting`, `Contractors`, etc. OUs under those OUs (child containers).

---

## SearchBase and Search Scope Parameters Examples

Let's look at some examples to illustrate the difference between `Base`, `OneLevel`, and `Subtree`. For these examples, we will focus on the `Employees` OU. In the screenshot of `Active Directory Users and Computers` below `Employees` is the `Base`, and specifying it with `Get-ADUser` will return nothing. `OneLevel` will return just the user `Amelia Matthews`, and `SubTree` will return all users in all child containers under the `Employees` container.

![image](https://academy.hackthebox.com/storage/modules/22/ou_base.png)

We can confirm these results using PowerShell. For reference purposes, let's get a count of all AD users under the `Employees` OU, which shows 970 users.

#### PowerShell - Count of All AD Users

```powershell-session
PS C:\htb> (Get-ADUser -SearchBase "OU=Employees,DC=INLANEFREIGHT,DC=LOCAL" -Filter *).count

970
```

As expected, specifying a SearchScope of `Base` will return nothing.

#### PowerShell - SearchScope Base

```powershell-session
PS C:\htb> Get-ADUser -SearchBase "OU=Employees,DC=INLANEFREIGHT,DC=LOCAL" -SearchScope Base -Filter *
PS C:\htb>
```

However, if we specify "`Base`" with "`Get-ADObject`" we will get just the object (Employees OU) returned to us.

#### PowerShell - SearchScope Base OU Object

```powershell-session
PS C:\htb> Get-ADObject -SearchBase "OU=Employees,DC=INLANEFREIGHT,DC=LOCAL" -SearchScope Base -Filter *

DistinguishedName                      Name      ObjectClass        ObjectGUID
-----------------                      ----      -----------        ----------
OU=Employees,DC=INLANEFREIGHT,DC=LOCAL Employees organizationalUnit 34f42767-8a2e-493f-afc6-556bdc0b1087
```

If we specify `OneLevel` as the SearchScope, we get one user returned to us, as expected per the image above.

#### PowerShell - Searchscope OneLevel

```powershell-session
PS C:\htb> Get-ADUser -SearchBase "OU=Employees,DC=INLANEFREIGHT,DC=LOCAL" -SearchScope OneLevel -Filter *

DistinguishedName : CN=Amelia Matthews,OU=Employees,DC=INLANEFREIGHT,DC=LOCAL
Enabled           : True
GivenName         : amelia
Name              : Amelia Matthews
ObjectClass       : user
ObjectGUID        : 3f04328f-eb2e-487c-85fe-58dd598159c0
SamAccountName    : amelia.matthews
SID               : S-1-5-21-2974783224-3764228556-2640795941-1412
Surname           : matthews
UserPrincipalName : amelia.matthews@inlanefreight
```

As stated above, the `SearchScope` values are interchangeable, so the same result is returned when specifying `1` as the `SearchScope` value.

#### PowerShell - Searchscope 1

```powershell-session
PS C:\htb> Get-ADUser -SearchBase "OU=Employees,DC=INLANEFREIGHT,DC=LOCAL" -SearchScope 1 -Filter *

DistinguishedName : CN=Amelia Matthews,OU=Employees,DC=INLANEFREIGHT,DC=LOCAL
Enabled           : True
GivenName         : amelia
Name              : Amelia Matthews
ObjectClass       : user
ObjectGUID        : 3f04328f-eb2e-487c-85fe-58dd598159c0
SamAccountName    : amelia.matthews
SID               : S-1-5-21-2974783224-3764228556-2640795941-1412
Surname           : matthews
UserPrincipalName : amelia.matthews@inlanefreight
```

Finally, if we specify `Subtree` as the SearchBase, we will get all objects within all child containers, which matches the user count we established above.

#### PowerShell - Searchscope Subtree

```powershell-session
PS C:\htb> (Get-ADUser -SearchBase "OU=Employees,DC=INLANEFREIGHT,DC=LOCAL" -SearchScope Subtree -Filter *).count

970
```

---
