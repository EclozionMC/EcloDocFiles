# EcloCombat Dependency Guide

> **Version**: 1.20
> **Type**: PVP / Combat Tag
> **Importance**: STANDARD

## 1. Introduction
**EcloCombat** handles PVP tagging, preventing players from logging out or using commands while fighting.

## 2. Integration

### Maven
```xml
<dependency>
    <groupId>com.eclozion.combat</groupId>
    <artifactId>EcloCombat</artifactId>
    <version>1.20</version>
    <scope>provided</scope>
</dependency>
```

### Checking Combat Tag
Use this to prevent an action (e.g., opening a menu) if the player is fighting.

```java
import com.eclozion.combat.EcloCombat;

public void openMenu(Player player) {
    EcloCombat combat = EcloCombat.getInstance();
    
    if (combat.getCombatManager().isInCombat(player)) {
        player.sendMessage("§cYou combat tagged! Wait " + 
            combat.getCombatManager().getRemainingTime(player) + "s.");
        return;
    }
    
    // Open menu
}
```
