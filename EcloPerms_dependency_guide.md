# EcloPerms Dependency Guide

> **Version**: 1.0.0
> **Type**: Permissions Management
> **Importance**: MEDIUM

## 1. Introduction
**EcloPerms** is the custom permissions engine for EclozionMC. It handles permission assignment, inheritance, and chat meta (prefixes/suffixes).

## 2. Integration & Setup

### Maven Dependency
```xml
<dependency>
    <groupId>com.eclozion.perms</groupId>
    <artifactId>EcloPerms</artifactId>
    <version>1.0.0</version>
    <scope>provided</scope>
</dependency>
```

### plugin.yml
```yaml
depend: [EcloPerms]
```

---

## 3. Checking Permissions

**Best Practice:** Do NOT use EcloPerms API to check permissions. Use the standard Bukkit API. EcloPerms injects permissions directly into the Bukkit attachment system.

```java
if (player.hasPermission("myplugin.admin")) {
    // allow access
}
```

---

## 4. Accessing Meta (Prefixes/Suffixes)

If you need to display a player's rank prefix in your plugin (e.g., scoreboard, custom chat channel), use the `PermissionManager`.

```java
import com.eclozion.perms.EcloPerms;
import com.eclozion.perms.data.User;
import com.eclozion.perms.manager.PermissionManager;

public String getPlayerPrefix(Player player) {
    PermissionManager pm = EcloPerms.getInstance().getPermissionManager();
    User user = pm.getUser(player.getUniqueId());
    
    if (user != null) {
        return pm.getUserPrefix(user);
    }
    return "";
}
```

---

## 5. Group Management API

You can inspect a player's groups or specific group data.

```java
public String getPrimaryGroup(Player player) {
    User user = EcloPerms.getInstance().getPermissionManager().getUser(player.getUniqueId());
    if (user != null) {
        return user.getPrimaryGroup(); // e.g., "admin"
    }
    return "default";
}
```

### Checking Group Weight
Useful for determining if one player outranks another.

```java
public boolean outranks(Player p1, Player p2) {
    PermissionManager pm = EcloPerms.getInstance().getPermissionManager();
    Group g1 = pm.getGroup(pm.getUser(p1.getUniqueId()).getPrimaryGroup());
    Group g2 = pm.getGroup(pm.getUser(p2.getUniqueId()).getPrimaryGroup());
    
    return g1.getWeight() > g2.getWeight();
}
```

---

## 6. Events

EcloPerms fires standard Bukkit events when players change state, but it is recommended to rely on the data available in the manager which is kept in sync.
