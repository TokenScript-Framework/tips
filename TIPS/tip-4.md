
# TIP-4: Multi-channel rendering

## Simple summary

Introduce a non-iframe presentation layer, to decouple card rendering from action logic in order to support 
multi-channel rendering where iframes are not available.

## Abstract

TokenScript cards rely on an iframe or webview to be securely embedded in other applications and allow rendering of the 
UI as well as orchestrating transactions and other off-chain processes. 
Without an iframe or webview, interactive TokenScripts cannot be achieved at this time. 

A good example of this is Farcaster frames or Metamask snaps, where the execution logic must be completely decoupled 
from UI rendering. 

## Motivation

Since its conception, TokenScript has relied heavily on iframes or webviews for the execution of Javascript logic. 
As discussed in the abstract, this imposes framework limitations where normal TokenScript cards cannot be used 
in user-agents that do not support iframes or use server-side APIs to invoke wallet interactions. 

## Specification

The EIP will focus only on UI rendering, but will touch on how complex actions could be enabled in the future. 
It will introduce an alternative rendering model that uses HTML & CSS templates that do not contain any Javascript code 
but provide a high level of flexibility. These templates can easily be parsed and rendered into an image for use in 
channels that cannot make use of iframes. 

Triggering any actions for this card will go through the same flow as an iframe based card, with one key change:

1. Invalidate attributes
2. Reload token metadata
3. Reload attributes and ~~push new card data to the iframe~~ rerender the iframe using the updated card data

This allows image frames to be reactive to changes made by the user.
It also allows for server-side rendering support for TokenScript cards in a secure manner that does not require the 
use of a headless browser or other resource intensive runtime. 

### Dependencies

The flexibility of this implementation will draw on improvements made in [TIP-1](./tip-1.md) & [TIP-2](./tip-2.md).

### Mustache templates

[Mustache](https://github.com/janl/mustache.js) or [Handlebars](https://handlebarsjs.com/) are two good candidates as a template language. 
They call themselves logic-less templates however they do provide conditional and iterable support, just not in the way you may expect. 

One advantage of a templating languages like this is that they do not impose any sandboxing requirements, unlike some 
other general programming language based solution.

### Defining an image renderer card

To define an image-rendered card, we introduce the renderer attribute to the `ts:view` element.

```xml

```

### Engine integration

The existing ViewController model of the engine is changed to enable the image based rendering model. 
It is based on the current IViewBinding interface, which leaves the actual implementation to the user-agent (TS viewer, API server, etc.). 
Where possible the same controller APIs are reused and modified to make cross-compatible with the new rendering model. 

## Considerations

