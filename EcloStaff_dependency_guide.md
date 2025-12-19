# EcloStaff Dependency Guide

> **Version**: 1.0.0
> **Type**: Moderation
> **Importance**: STANDARD

## 1. Introduction
**EcloStaff** provides tools for moderation (Freeze, Vanish, InventorySee).

## 2. Integration

### Maven
```xml
<dependency>
    <groupId>com.eclozion.staff</groupId>
    <artifactId>EcloStaff</artifactId>
    <version>1.20</version>
    <scope>provided</scope>
</dependency>
```

### Checking Staff Mode
Use this to check if a player is in Vanish or ModMode before interacting with them.

```java
import com.eclozion.staff.EcloStaff;

public boolean isVanished(Player player) {
    // Basic check (if VanishManager exposed)
    return player.hasMetadata("vanished"); // Common pattern
}
```

### Permissions
- `eclostaff.mod`: Access /mod mode.
