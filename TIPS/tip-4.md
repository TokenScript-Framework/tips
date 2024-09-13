
# TIP-4: Multi-channel rendering

## Simple summary

Introduce a non-iframe presentation layer, to decouple card rendering from action logic in order to support 
multi-channel rendering where iframes are not available.

## Abstract

TokenScript cards rely on an iframe or webview to be securely embedded in other applications and allow rendering of the UI as well 
as orchestrating transactions and other off-chain processes. 
Without an iframe or webview, interactive TokenScript cannot be achieved at this time. 

A good example of this is Farcaster frames or Metamask snaps.

## Motivation



## Specification