# EcloSpawners Dependency Guide

> **Version**: 1.0.0
> **Type**: Spawners & Stacking
> **Importance**: STANDARD

## 1. Introduction
**EcloSpawners** allows players to mine spawners and potentially stack them.

## 2. Integration

### Maven
```xml
<dependency>
    <groupId>com.eclozion.spawners</groupId>
    <artifactId>EcloSpawners</artifactId>
    <version>1.20</version>
    <scope>provided</scope>
</dependency>
```

### Permissions
- `eclospawners.admin`: Bypass protections and give spawners.
- `eclospawners.mine`: Ability to mine items with silk touch (if configured).
