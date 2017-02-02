---
layout: presentation
title: FpML Product Defs
---

## FpML Product Definitions
https://github.com/mrlifetime/fpml_prod_defs/


---

## Objectives

- Create a view for sharing supported products between execution venues and their clients.
- Encode common conventions associated with products.
- Ability to represent majority of electronically traded derivatives (currently modelled for IRS only)
- Ability to represent package products that comprise of multiple components

---

## Basic design

- Modelled as a schema on top of pre-trade view v5.9
- Avoid changing existing FpML types
- Provide new structures to use existing FpML types

---

## High level structure

```xml
<ProductDefinitionDocument>
  <!-- header of the document contains useful meta data -->
  <header>
    <platformDate>2016-10-12</platformDate>
  </header>

  <!-- multiple definitions go here -->
  <definition><!-- body --></definition>
  <definition><!-- body --></definition>
  <!-- ... -->
 
  <!-- PartiesAndAccounts.model defines parties that are used to encode trade sides -->
  <party id="party1">
      <partyId>MarketMaker</partyId>
  </party>
  <party id="party2">
      <partyId>PriceTaker</partyId>
  </party>
</ProductDefinitionDocument>

```
---

## High level structure 2 - Product Definition

Product definititon consists of:

- Description - "friendly name" for a product
- Conventions which are observed by the platform
- Product definition itself
- Trade side reference - href to an id that defines the main "side" for the product (optional)

### Example - Fixed-Float vanilla swap

```xml
    <definition id="BPSW5">
        <description>GBP SWAP VS 6M 5YR</description>
        <conventions>
            <maximumQuotePrecisionDecimal>5</maximumQuotePrecisionDecimal>
            <quoteType>FixedRate</quoteType>
            <quoteUnit>Percent</quoteUnit>
        </conventions>
        <swap>
            <productId productIdScheme="http://www.bloomberg.com/ticker-scheme">BPSW5</productId>
            <swapStream id="BPSW5_floatingLeg">
              <!-- swap stream definition -->
            </swapStream>
            <swapStream id="BPSW5_fixedLeg">
              <!-- swap stream definition -->
            </swapStream>
        </swap>
        <tradeSideRelativeToSwapStream href="BPSW5_fixedLeg" />
    </definition>
```

---

### Package structure

Package structure allows to describe products that consist of 2 or more outright (single) products.

User can refer to components by:

- Providing href to product definition elsewhere in the document
- Providing a standard identifier (such as ISIN) for a product NOT defined in the document

## Additional package data

User can also encode following package details

- Package hedge type
- Risk relationship between legs
- Trade sides relationship for each leg
- Main component that determines the side of the "trade"
---

## Example - Butterfly swap

```xml
    <definition id="EU020305">
        <description>EUR Butterfly 2Y 3Y 5Y</description>
        <conventions>
            <maximumQuotePrecisionDecimal>3</maximumQuotePrecisionDecimal>
            <quoteType>Spread</quoteType>
            <quoteUnit>BasisPoint</quoteUnit>
        </conventions>
        <package>
            <productId productIdScheme="http://www.bloomberg.com/ticker-scheme">EU020305</productId>
            <packageName>Butterfly</packageName>
            <hedgeType>DeltaNeutral</hedgeType>
            <component componentId="EU020305_EUSA2">
                <productDefinitionReference href="EUSA2" />
                <payerPartyReference href="party1" />
                <receiverPartyReference href="party2" />
                <hedgeRiskWeight>0.5</hedgeRiskWeight>
            </component>
            <component componentId="EU020305_EUSA3">
                <productDefinitionReference href="EUSA3" />
                <payerPartyReference href="party2" />
                <receiverPartyReference href="party1" />
                <hedgeRiskWeight>1.0</hedgeRiskWeight>
            </component>
            <component componentId="EU020305_EUSA5">
                <productDefinitionReference href="EUSA5" />
                <payerPartyReference href="party1" />
                <receiverPartyReference href="party2" />
                <hedgeRiskWeight>0.5</hedgeRiskWeight>
            </component>
        </package>
        <tradeSideRelativeToComponent href="EU020305_EUSA5" />
    </definition>
```
---

## Example - Swap vs Bond

```xml
    <definition id="USSP5">
        <description>USD 5Y US TREASURY SPREAD</description>
        <conventions>
            <maximumQuotePrecisionDecimal>3</maximumQuotePrecisionDecimal>
            <quoteType>Spread</quoteType>
            <quoteUnit>BasisPoint</quoteUnit>
        </conventions>
        <package>
            <productId productIdScheme="http://www.bloomberg.com/ticker-scheme">USSP5</productId>
            <packageName>SwapVsBond</packageName>
            <hedgeType>DeltaNeutral</hedgeType>
            <component componentId="USSP5_USSWAP5">
                <productDefinitionReference href="USSWAP5" />
                <payerPartyReference href="party1" />
                <receiverPartyReference href="party2" />
                <hedgeRiskWeight>1.0</hedgeRiskWeight>
            </component>
            <component componentId="USSP5_BOND">
                 <!-- not in the document -->
                <productId productIdScheme="http://www.fpml.org/spec/2002/instrument-id-ISIN-1-0/">US912828T677</productId>
                <buyerPartyReference href="party2" />
                <sellerPartyReference href="party1" />
                <hedgeRiskWeight>1.0</hedgeRiskWeight>
            </component>
        </package>
        <tradeSideRelativeToComponent href="USSP5_USSWAP5" />
    </definition>
```
---
