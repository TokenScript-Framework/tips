# TIP-3: Intrinsic cards

## Simple summary

Add built-in cards that are bundled with the TokenScript engine, 
and can be enabled/disabled by TokenScript developers through simple feature flags.

## Abstract

Tokens using standards like erc-20, erc-721, erc-1155 all have similar functionality, for instance the transfer action. 
Similarly, many TokenScript applications have the potential to work across multiple token collections or even all 
collections belonging to specific token standards. Some examples of this are our token-to-token messaging system and the 
DOOAR DEX.

## Motivation

We have previously used TokenScript templates via a launchpad API, however these are limited to providing simple functions 
for tokens that do not have a TokenScript associated with them. In many circumstances, it makes sense to have these cards 
enabled for all tokens by default. Transfer functions are the perfect example of this. In other scenarios these cards 
should be disabled by default and enabled by TokenScript developers that want to add some pre-bundled cards to their 
project.

## Specification

To support this EIP we introduce two main concepts into the TokenScript engine: 

- Bundled Cards
- Virtual TokenScripts

### Bundled Cards

Bundled cards are based on TokenScript templates. They are pre-compiled .tsml files with cards that can be combined  
with any tokenscript. Due to name collisions, these cards should be self-contained, and only reference the host TokenScripts 
origin contracts. This imposes additional requirements, including tokenscript XML schema changes to allow `ts:selections` and named `ts:transactions` to be placed inside `ts:card`.

A set of bundled TokenScript cards might look like this:
```xml
<ts:cards>
    <ts:webContent>
        <ts:include type="html" src="./dist/index.html"/>
        <ts:include type="css" src="./src/styles.css"/>
    </ts:webContent>
    <ts:card name="inbuilt_erc20Transfer" type="action" origins="Token" exclude="notERC20">
        <ts:label>
            <ts:string xml:lang="en">Play Morchi</ts:string>
        </ts:label>
        <ts:selections name="notERC20" filter="tokenType!=erc20">
            <ts:label>
                <ts:string xml:lang="en">Cat already adopted</ts:string>
            </ts:label>
        </ts:selections>
        <ts:attribute name="toAddress">
            <ts:label>
                <ts:string xml:lang="en">To</ts:string>
            </ts:label>
            <ts:origins>
                <ts:user-entry as="address"/>
            </ts:origins>
        </ts:attribute>
        <ts:attribute name="redeemAmt">
            <ts:label>
                <ts:string xml:lang="en">Amount</ts:string>
            </ts:label>
            <ts:origins>
                <ts:user-entry as="uint"/>
            </ts:origins>
        </ts:attribute>
        <ts:view xmlns="http://www.w3.org/1999/xhtml" urlFragment="erc20Transfer">
            <ts:viewContent name="common"/>
        </ts:view>
    </ts:card>
    <!-- Other cards for er721 & erc1155 transfers -->
</ts:cards>
```

An alternative pattern would be to treat a these additional cards as a completely different TokenScript instance.  
i.e. `tokenScript.getBundledTokenScriptInstance() => TokenScript`  
The bundled cards from this separate instance can be merged with the current instance and displayed in the UI with 
minimal changes to the code and no changes to the XML schema.

This may be a better approach as it requires additional research to determine the best method.

Some transformation of the bundled cards .tsml is required to set the correct contract, token origins and the correct 
origin attribute for the `ts:card` element.

#### Feature flags

The host TokenScript developer can enable OR disable specific cards based on a simple syntax:

```xml
<ts:features>
    <ts:feature name="erc20Transfer" enable="false"/>
    <ts:feature name="erc20Messaging" enable="true"/>
</ts:features>
```

In this scenario the developer chooses to disable the bundled transfer card and implement his own.
He also enables the erc-20 messaging card which is disabled by default.

In this way the TokenScript can contain no cards within itself but still use these flags to enable built-in 
cards. 

### Virtual TokenScripts

Since we now have a way to provide a small amount of TokenScript XML markup to enable built-in cards, we can create 
the concept of "virtual TokenScripts". This enables us to use bundles cards for tokens that don't even have a 
TokenScript published for them yet. 

When a token fails to resolve a TokenScript for itself, we provide a small XML snippet that enables bundled cards. 

## Considerations

The main consideration mentioned above is how to implement these features in a way that:

- Limits the changes to TS Engine logic and the XML schema
- Does not create additional requirements for external tools such as TokenScript CLI that will be used to build these bundled cards.
- Info & Transfer card should be used as a PoC in order to drive the implementation, as they are already available as TokenScript templates.

## Summary

This EIP represents an easy way for us to enable common token features for every single token, on every single chain  
in a way that is transparent to the user and customisable for the TokenScript developer.