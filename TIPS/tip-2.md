# TIP-2: Allow dot notation for all attribute references

## Simple summary

Allow Lodash path notation for referencing nested values of attributes, anywhere where attributes are currently referenced 
within the tokenscript.xml. 

## Abstract

[Lodash path notation](https://lodash.com/docs/4.17.15#get) has previously been used for 
[Reference Attributes](https://sln-doc.vercel.app/framework/tokenscript-syntax/attributes#referenceAttributes). 
These were implemented for SmartCats to allow the display of nested values of the catInfo attribute which 
contains complex data. Additionally, this allows other attributes & transactions to reference this nested data.

```xml
<ts:token>
    <ts:attribute name="catInfo">
        <ts:type><ts:syntax>1.3.6.1.4.1.1466.115.121.1.27</ts:syntax></ts:type>
        <ts:label>
            <ts:string xml:lang="en">Cat Info</ts:string>
        </ts:label>
        <ts:origins>
            <ethereum:call function="getCatInfo3" contract="SmartCatScore" as="abi">
                <ts:data>
                    <ts:uint256 ref="tokenId"/>
                </ts:data>
            </ethereum:call>
        </ts:origins>
    </ts:attribute>
    <ts:attribute name="level">
        <ts:label>
            <ts:string xml:lang="en">Level</ts:string>
        </ts:label>
        <ts:origins>
            <ts:data ref="catInfo.state.level" />
        </ts:origins>
    </ts:attribute>
</ts:token>
```

## Motivation

Although reference attributes allow the use of nested data within transactions, filters and other attributes, 
it's cumbersome for the developer to declare an attribute for each nested value that he intends to use. 

By expanding Lodash notation to all references, we can simplify the developer experience. 

## Specification

The engine will be modified to allow Lodash path notation in the following areas:
- ref & local-ref attributes, including `ts:transaction` && `ts:call` elements.
- filter queries used in `ts:selections`.

These changes may require a small amount of refactoring in the existing argument & attribute code.

## Considerations

Access to user-entry attributes that are not declared in the XML must be handled appropriately. 
Metadata access may also be considered in the future. 

## Summary

This simple change will make it easier to work with complex data types and minimise the XML needed to do so.