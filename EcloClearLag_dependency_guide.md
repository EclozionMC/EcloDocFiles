# EcloClearLag Dependency Guide

> **Version**: 1.0.0
> **Type**: Performance Optimization
> **Importance**: STANDARD

## 1. Introduction
**EcloClearLag** manages entity clearing and stack limits.

## 2. Integration
This plugin functions primarily as a standalone service.

### Maven
```xml
<dependency>
    <groupId>fr.eclozion.clearlag</groupId>
    <artifactId>EcloClearLag</artifactId>
    <version>1.0.0</version>
    <scope>provided</scope>
</dependency>
```

### Usage
- Ensure your custom entities are not accidentally cleared by checking if `EcloClearLag` has an exclusion list in its config.
