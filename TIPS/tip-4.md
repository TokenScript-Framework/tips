
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

### Defining an image renderered card

To define a mustache rendered card, we introduce the renderer attribute to the `ts:view` element. 
`renderer="image"` declares that we want to render this view as an image and `templateFormat="mustache"` indicates 
that we will use a mustache template. 

```xml
<ts:view renderer="image" templateFormat="mustache">
    <ts:include type="mustache" src="./src/templates/token_info.mustache"/>
</ts:view>
```

Note: the templateFormat attribute could default to mustache, but allows other template languages in the future.

Just like normal views, they can define `ts:include` tags allowing the CLI to inline content from other files at build-time.
We use a new type, `type="mustache"` as it will allow us to do any mustache based post-processing to the content, 
such as wrapping it with CDATA tags. 

The output of a content would look something like this: 

```xml
<ts:view renderer="mustache">
    <![CDATA[
        <style>
            .my-container {
                background: #000;
            }
        </style>
        <div class="my-container">
        {#myAttribute}
            <p>Hello {myAttribute}</p>
        {/myAttribute}
        </div>
    ]]>
</ts:view>
```

### Engine integration

The existing ViewController model of the engine is changed to enable the image based rendering model. 
It is based on the current IViewBinding interface, which leaves the actual implementation to the user-agent (TS viewer, API server, etc.). 
Where possible the same controller APIs are reused and modified to make cross-compatible with the new rendering model. 

### View rendering

A popular way to render images from HTML is through canvas. 
Although canvas is not available natively in node.js, many libraries including [node-canvas](https://github.com/Automattic/node-canvas) 
can provide support. 

There are many alternatives for rendering and this [helpful resource](https://stackoverflow.com/questions/10721884/render-html-to-an-image) 
can provide some further guidance.

## Considerations

Once this new rendering model is implemented, it would be possible to have some support for action cards that use 
TokenScript's declarative transactions. This would require the correct implementation of 
[IWalletAdapter](https://github.com/SmartTokenLabs/tokenscript-engine/blob/master/javascript/engine-js/src/wallet/EthersAdapter.ts) 
for the specific channel and would be limited to a single ethereum transaction.

### Support for more complex actions

Whilst this EIP does not cover complex actions (such as chained transactions) it's important to think about how this 
could be achieved in the future without an iframe. 

Once such solution would be a lightweight scripting language to orchestrate transactions in the same way as the 
card SDK is used in traditional iframe cards. The language APIs could even share the same interface.
An example of card SDK invoking multiple transactions:

```javascript
if (!await tokenscript.action.executeTransaction("approve")){
  return; // User aborted transaction
}

await tokenscript.action.executeTransaction("levelUp");
```

These scripts can be run in a secure Javascript sandbox on the server, which can allow access to HTTP data sources as 
well as the token context data provided by the engine.

This would be more flexible than a declarative solution, such as allowing `ts:transaction` to encapsulate multiple 
`ethereum:transaction` declarations like demonstrated here:

```xml
<ts:transaction name="approveSUT">
    <ethereum:transaction as="uint" contract="SUTToken" function="approve">
        <ts:data>
            <ts:address ref="contractAddress_MorchiScore"/>
            <ts:uint256 local-ref="approveAmt"/>
        </ts:data>
    </ethereum:transaction>
    <ethereum:transaction as="uint" contract="MorchiGame" function="levelUp">
        <ts:data>
            <ts:address ref="contractAddress_MorchiScore"/>
            <ts:uint256 local-ref="to"/>
        </ts:data>
    </ethereum:transaction>
</ts:transaction>
```

Without some additional mechanism to provide control, usage is limited to chaining transactions, without any conditional 
checks and inter-transaction data sharing. 

The clear trade-off is between flexibility and ease of implementation. 

## Summary

Providing an image-based rendering model for TokenScript will allow a more flexible interaction model that can be 
used for channels that don't support iframes. This will bring the ability to do simple interactions via the current declarative 
transaction support. A lightweight transaction scripting language or enhancements to declarative transactions can later 
be introduced to provide more flexibility to action cards that use the image renderer model. 
