# EcloHDV Dependency Guide

> **Version**: 1.0.0
> **Type**: Auction House
> **Importance**: STANDARD

## 1. Introduction
**EcloHDV** is the player-to-player market system.

## 2. Integration
Integrates with `EcloEco` for transactions.

### Maven
```xml
<dependency>
    <groupId>com.eclozion.hdv</groupId>
    <artifactId>EcloHDV</artifactId>
    <version>1.20</version>
    <scope>provided</scope>
</dependency>
```

### Automation
To add items programmatically to the HDV (e.g., from a plugin event), you would need to access the internal `HDVManager` (Check source if available).
