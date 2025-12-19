# EcloTP Dependency Guide

> **Version**: 1.0.0
> **Type**: Teleportation
> **Importance**: STANDARD

## 1. Introduction
**EcloTP** handles warps, homes, and spawn teleportation.

## 2. Integration

### Maven
```xml
<dependency>
    <groupId>com.eclozion.tp</groupId>
    <artifactId>EcloTP</artifactId>
    <version>1.20</version>
    <scope>provided</scope>
</dependency>
```

### API
Hook into `WarpManager` (via main instance) to get list of warps for custom menus.
