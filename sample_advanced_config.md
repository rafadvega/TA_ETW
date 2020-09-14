# Annotated Advanced YAML Configuration
Because sometimes seeing the entire picture together can be helpful, here is a sample YAML config file that does the following:

1. Track eventIds 256 and 279 in the Microsoft-Windows-DNSServer ETW provider
2. Track eventId 11 in the Microsoft-Windows-WMI-Activity ETW provider    

Each option in the configration schema is annotated with a comment the first time that it is used.

###### Annotated config demonstrating all features of TA_ETW
```yaml
###################################################################################################################
# The config file below shows two example traces                                                                  #
# The first trace will track eventIds 256 and 279 in the Microsoft-Windows-DNSServer ETW provider                 #
# The second trace will track eventId 11 in the Microsoft-Windows-WMI-Activity ETW provider                       #
# New information in the configuration scheme will be annotated in the examples below the first time they appear  #
###################################################################################################################
Traces: # Collection of traces to run [required]
  - Name: TA_ETW_DNS # Friendly name for this trace entry [required]
    Provider: Microsoft-Windows-DNSServer # Name of the ETW provider to leverage [required]
    Index: etw_dns # Splunk index that data should be sent to [optional, if omitted the index defined in inputs.conf will be used]
    SourceType: etw:dns #Splunk sourcetype that data should be sent to [optional, if omitted the sourcetype defined in inputs.conf will be used, or none if one isn't in inputs.conf]
    Events: # Collection of events to track [required]
    - Id: 256 # EventId to capture [required]
      Fields: # Collection of fields to output [required]
      # There are several "built-in" fields that you can leverage:
      # Timestamp fields will output the current UTC time in the format YYYY-MM-DDTHH:MM:SS.sssssssZ
      # Literal fields will output the value defined in LiteralValue
      # Property fields (indicated by specifying IsField: false) will attempt to find the named field as a property of the event object rather than a field
      # This is useful for getting information like the EventId since it is not a field in the event data itself
      - Name: timestamp # Name of the field in both input (if IsField: true) and output (if no translation name is defined) [required]
        IsTimestamp: true # Indicates that this field's value should be a generated timestamp [optional]
      - Name: query_type
        IsLiteral: true # Indicates that this field's value should be the contents of LiteralValue [optional]
        LiteralValue: Query # The value for the ouput of a literal field [required when IsLiteral is true for a field]
      - Name: message_type
        IsLiteral: true
        LiteralValue: Query
      - Name: vendor_product
        IsLiteral: true
        LiteralValue: Microsoft DNS Server ETW
      - Name: Source
        ExtractionMethod: GetAnsiString # Indicates which krabsetw method to use to get the field value.
                                        # Options are:
                                        #   GetBinary
                                        #   GetAnsiString
                                        #   GetInt16
                                        #   GetInt32
                                        #   GetInt64
                                        #   GetInt8
                                        #   GetIPAddress
                                        #   GetUInt16
                                        #   GetUInt32
                                        #   GetUInt64
                                        #   GetUInt8
                                        #   GetUnicodeString
        TranslationName: src # Indicates that this field's name should be changed to the provided value when the event is output [optional]
      - Name: InterfaceIP
        ExtractionMethod: GetAnsiString
        TranslationName: dest
      - Name: QNAME
        ExtractionMethod: GetAnsiString
        Filters: # Collection of filters to apply to the field, if a field matches a filter the entire event that it belongs to is discarded [optional]
                 # The filter options are StartsWith, Contains, and EndsWith
          EndsWith: # Indicates that the EndsWith filter method should be used [optional]
          - microsoft.com. # Indicates that if the value of the QNAME field ends with microsoft.com. the event should be discarded [required if using a filter]
        TransformationMethod: # Indicates that the value of the field should be modified prior to being output
                              # Options for modification include:
                              #  Replace - replace a string with another string in the value of the field
                              #     Replace expects arguments ReplaceOld and ReplaceNew to be defined in the config
                              #  RegexReplace - replace a string with another string in the value of the field using regular expressions
                              #     RegexReplace expects arguments Regex and RegexReplacement to be defined in the config
                              #  TrimNChars - trim off the last N characters from the value of the field
                              #     TrimNCharts expects the argument TrimNumber to be defined in the config
                              #  TrimChar - trim off the last character from the value of the field if it is this character
                              #     TrimChar expects the argument TrimChar to be defined in the config
          TransformationType: TrimChar
          TrimChar: .
        TranslationName: query
      - Name: Id
        IsField: false # The Id field being defined here is going to look for an Id property of the event object
        ExtractionMethod: GetInt32
        TranslationName: name
        FieldEnumeration: # Collection of number: Literal pairs that will be used to translate a numeric field value to the literal string provided
          0: Unknown      # pairs from https://docs.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2012-r2-and-2012/dn800669(v%3Dws.11)#analytic-events
          256: Query_Received
          257: Response_success
          258: Response_failure
          259: Ignored_query
          260: Query_out
          261: Response_in
          262: Recursive_query_timeout
          263: Update_in
          264: Update_response
          265: IXFR_request_out
          266: IXFR_request_in
          267: IXFR_response_out
          268: IXFR_response_in
          269: AXFR_request_out
          270: AXFR_request_in
          271: AXFR_response_out
          272: AXFR_response_in
          273: XFR_notification_in
          274: XFR_notification_out
          275: XFR_notify_ACK_in
          276: XFR_notify_ACK_out
          277: Update_forward
          278: Update_response_in
          279: Internal_lookup_CNAME
          280: Internal_lookup_additional
      - Name: QTYPE
        ExtractionMethod: GetInt32
        TranslationName: record_type
        FieldEnumeration: # pairs from https://en.wikipedia.org/wiki/List_of_DNS_record_types
          0: Uknown
          1: A
          28: AAAA
          18: AFSDB
          42: APL
          257: CAA
          60: CDNSKEY
          59: CDS
          37: CERT
          5: CNAME
          49: DHCID
          32769: DLV
          39: DNAME
          48: DNSKEY
          43: DS
          55: HIP
          45: IPSECKEY
          25: KEY
          36: KX
          29: LOC
          15: MX
          35: NAPTR
          2: NS
          47: NSEC
          50: NESC3
          51: NSEC3PARAM
          61: OPENPGPKEY
          12: PTR
          46: RRSIG
          17: RP
          24: SIG
          53: SMIMEA
          6: SOA
          33: SRV
          44: SSHFP
          32768: TA
          249: TKEY
          52: TLSA
          250: TSIG
          16: TXT
          256: URI
          255: STAR
          252: AXFR
          251: IXFR
          41: OPT
    - Id: 279
      Fields:
      - Name: timestamp
        IsTimestamp: true
      - Name: query_type
        IsLiteral: true
        LiteralValue: Query
      - Name: message_type
        IsLiteral: true
        LiteralValue: Query
      - Name: vendor_product
        IsLiteral: true
        LiteralValue: Microsoft DNS Server ETW
      - Name: Source
        ExtractionMethod: GetAnsiString
        TranslationName: src
      - Name: InterfaceIP
        ExtractionMethod: GetAnsiString
        TranslationName: dest
      - Name: QNAME
        ExtractionMethod: GetAnsiString
        Filters:
          EndsWith:
          - microsoft.com.
        TransformationMethod:
          TransformationType: TrimChar
          TrimChar: .
        TranslationName: query
      - Name: Id
        IsField: false
        ExtractionMethod: GetInt32
        TranslationName: name
        FieldEnumeration:
          0: Unknown
          256: Query_Received
          257: Response_success
          258: Response_failure
          259: Ignored_query
          260: Query_out
          261: Response_in
          262: Recursive_query_timeout
          263: Update_in
          264: Update_response
          265: IXFR_request_out
          266: IXFR_request_in
          267: IXFR_response_out
          268: IXFR_response_in
          269: AXFR_request_out
          270: AXFR_request_in
          271: AXFR_response_out
          272: AXFR_response_in
          273: XFR_notification_in
          274: XFR_notification_out
          275: XFR_notify_ACK_in
          276: XFR_notify_ACK_out
          277: Update_forward
          278: Update_response_in
          279: Internal_lookup_CNAME
          280: Internal_lookup_additional
      - Name: QTYPE
        ExtractionMethod: GetInt32
        TranslationName: record_type
        FieldEnumeration:
          0: Uknown
          1: A
          28: AAAA
          18: AFSDB
          42: APL
          257: CAA
          60: CDNSKEY
          59: CDS
          37: CERT
          5: CNAME
          49: DHCID
          32769: DLV
          39: DNAME
          48: DNSKEY
          43: DS
          55: HIP
          45: IPSECKEY
          25: KEY
          36: KX
          29: LOC
          15: MX
          35: NAPTR
          2: NS
          47: NSEC
          50: NESC3
          51: NSEC3PARAM
          61: OPENPGPKEY
          12: PTR
          46: RRSIG
          17: RP
          24: SIG
          53: SMIMEA
          6: SOA
          33: SRV
          44: SSHFP
          32768: TA
          249: TKEY
          52: TLSA
          250: TSIG
          16: TXT
          256: URI
          255: STAR
          252: AXFR
          251: IXFR
          41: OPT
  - Name: TA_ETW_WMI
    Provider: Microsoft-Windows-WMI-Activity
    Index: etw_wmi
    Events:
    - Id: 11
      Fields:
      - Name: Timestamp
        IsTimestamp: true
      - Name: vendor_product
        IsLiteral: true
        LiteralValue: WMI Activity ETW
      - Name: Operation
        ExtractionMethod: GetUnicodeString
        TranslationName: operation
      - Name: ClientMachine
        ExtractionMethod: GetUnicodeString
        TranslationName: src
      - Name: User
        ExtractionMethod: GetUnicodeString
        TranslationName: username
      - Name: ClientProcessId
        ExtractionMethod: GetUInt32
        TranslationName: processid
```
