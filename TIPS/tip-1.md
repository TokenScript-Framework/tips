
# TIP-1: REST API attribute source

## Simple summary

User defined attributes are helpful for developers to declaratively define the data they want to use in their TApp. 
However, attributes have limited data sources (ethereum function calls and event) and cannot access off-chain data. 
This restriction necessitates the need to use Javascript within a card to access any off-chain data, making TokenScript 
applications heavily rely on the iframe/webview model to function

## Abstract

Implement a new API attribute source that can be used to fetch data from REST APIs. 
GraphQL can also be implemented in the future but is out of scope of this proposal.
This TIP focuses on attribute sources, but there is no reason that this declarative API format couldn't be used for 
transactions in the future (i.e. declaratively chaining a transaction with an off-chain API call).

## Motivation

Enhancing attribute functionality will pave the way for advanced multi-channel functionality. 
Once more data can be declared within the XML and accessed as inputs to transactions and other processes, 
we can reduce the need for card Javascript logic and decouple rendering from actions.

Additionally, straight out of the box we bring `ts:selection` support. Allowing card to be disabled based on off-chain data.

## Specification

A new XML element `ts:api` is added to the schema, and is allowed within the `ts:origin` element of a `ts:attribute`.
This element defines the REST API call that is used to retrieve the data for the attribute.

```xml

<ts:attribute name="myApiData">
    <ts:label>
        <ts:string xml:lang="en">API Data</ts:string>
    </ts:label>
    <ts:origins>
        <ts:api method="GET" 
                url="https://my.api/some/${path}?params=${canComeFromAttributes}"
                as="json"
                bodyFormat="json">
            <ts:body ref="myAttribute.level"/>
        </ts:api>
    </ts:origins>
</ts:attribute>
```

### ts:api

The `ts:api` element contains a few attributes that control its behavior:

- method: The HTTP method to use.
- url: The URL for the API request. This can contain tokens that reference attribute values.
- as: How to handle the response. json & text are supported.
- bodyFormat: The format of the body. This is either `json` or `form` (for HTTP form encoding).

### ts:body

Each `ts:api` element can optionally contain a single `ts:body` element.
The design of this element is borrowed from [Referential Attributes](https://sln-doc.vercel.app/framework/tokenscript-syntax/attributes#referenceAttributes) 
which can reference nested values of complex attributes using a dot notation. You can see this in the previous example.

In this example, we see how we can draw on values from multiple attributes in a single request. 
Rather than referencing a single attribute with `ts:body`, we can include multiple `ts:data` elements within `ts:body`. 
These each have their own `ref` to an attribute and can access nested values. 
They can also optionally contain a `name` attribute that is used as the JSON property name for the request. 
If the name is not supplied, the last portion of the `ref` (everything after the last dot) is used as a name. 

```xml
<ts:api method="GET" 
        url="https://my.api/some/${path}?params=${canComeFromAttributes}"
        as="json"
        bodyFormat="json">
    <ts:body>
        <ts:data name="myParam1" ref="myAttribute"/>
        <ts:data ref="mySecondAttribute.myParam2"/>
    </ts:body>
</ts:api>
```

## Summary

Adding HTTP APIs as an attribute source is an easy win to enhance attribute functionality and bring declarative 
off-chain data to TokenScript. Uses of this are discussed further in [TIP-4](tip-4.md).
