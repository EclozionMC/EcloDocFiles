# EcloSync Dependency Guide

> **Version**: 2.0.1
> **Type**: Data Synchronization / Middleware
> **Importance**: CRITICAL (Required for cross-server data)

## 1. Introduction
**EcloSync** is the "Universal Data Hub" for the EclozionMC network. It acts as the nervous system, allowing plugins to communicate across different servers in real-time, validate data structures, and discover available services.

**Key Features:**
- **Universal Pub/Sub**: Real-time messaging between servers.
- **Service Discovery**: Register your plugin as a service so others knows it's online.
- **Schema Registry**: Enforce data contracts using JSON Schemas.
- **Data Persistence**: Key-Value storage abstraction (Database/Redis/Hybrid).

---

## 2. Integration & Setup

### Maven Dependency
```xml
<dependency>
    <groupId>com.eclozion.sync</groupId>
    <artifactId>EcloSync</artifactId>
    <version>2.0.1</version>
    <scope>provided</scope>
</dependency>
```

### plugin.yml
```yaml
name: YourPlugin
main: com.your.plugin.Main
depend: [EcloCore, EcloSync] # EcloSync requires EcloCore
```

---

## 3. Pub/Sub Messaging (Channel API)

The most common use case is sending real-time updates to other servers (e.g., "Player X found a legendary item").

### Publishing a Message
Use a namespaced channel format: `plugin_name:action`.

```java
import com.eclozion.sync.api.SyncAPI;

public void broadcastDrop(String playerName, String itemName) {
    String payload = playerName + ":" + itemName;
    SyncAPI.publish("myrpg:legendary_drop", payload);
}
```

### Subscribing to Messages
Register your subscription in `onEnable`.

```java
@Override
public void onEnable() {
    SyncAPI.subscribe("myrpg:legendary_drop", (channel, message) -> {
        // This runs on an async thread usually
        String[] parts = message.split(":");
        String player = parts[0];
        String item = parts[1];
        
        getLogger().info("[Network] " + player + " found " + item);
    });
}
```

> **Note**: Wildcard subscriptions (e.g., `myrpg:*`) are supported by `ChannelManager` but depend on the underlying adapter implementation. Stick to exact channels for maximum reliability.

---

## 4. Service Discovery

If your plugin provides a service (e.g., an Economy provider) that other plugins might need to verify is active on the network, register it.

### Registering a Service
```java
import com.eclozion.sync.service.ServiceRegistry;
import com.eclozion.sync.api.SyncAPI;

public void onEnable() {
    ServiceRegistry.ServiceInfo info = new ServiceRegistry.ServiceInfo(
        "MyEconomy",          // Service Name
        "1.2.0",              // Version
        List.of("economy", "bank") // Capabilities
    );
    
    SyncAPI.registerService(info);
}
```

### Discovering Services
Check if a dependent service is running network-wide.

```java
public void checkForEconomy() {
    if (SyncAPI.isServiceAvailable("MyEconomy")) {
        getLogger().info("Economy service is online on the network.");
    }
}
```

---

## 5. Schema Registry & Validation

EcloSync 2.0 introduces strict data validation. If you send complex JSON objects over Channels, you should register a schema.

### 1. Register Schema (JSON Format)
```java
String itemSchema = """
{
  "type": "object",
  "properties": {
    "itemId": { "type": "string" },
    "amount": { "type": "integer", "minimum": 1 }
  },
  "required": ["itemId", "amount"]
}
""";

SyncAPI.registerSchema("myrpg:item_transfer", itemSchema);
```

### 2. Validate Data
Before processing (or sending) data, you can validate it.

```java
import com.eclozion.sync.api.ValidationResult;

public void processTransfer(String json) {
    ValidationResult result = SyncAPI.validateAgainstSchema("myrpg:item_transfer", json);
    
    if (result.isValid()) {
        // Process
    } else {
        getLogger().warning("Invalid data received: " + result.getErrors());
    }
}
```

---

## 6. Key-Value Storage (V1 Compatibility)

For simple persistent data that needs to be accessible everywhere (like player global settings).

```java
// Set
SyncAPI.set(player.getUniqueId(), "language", "en_US");

// Get (Blocking - run async)
String lang = SyncAPI.get(player.getUniqueId(), "language");
```

---

## 7. Common Use Cases

### Case 1: Cross-Server Chat
**Sender:**
```java
SyncAPI.publish("global_chat", senderName + "|" + message);
```

**Receiver:**
```java
SyncAPI.subscribe("global_chat", (ch, msg) -> {
    String[] parts = msg.split("\\|");
    Bukkit.broadcastMessage(ChatColor.BLUE + "[Global] " + parts[0] + ": " + parts[1]);
});
```

### Case 2: Inventory Sync
1. Define Schema `inventory_schema`.
2. On `PlayerQuitEvent`: Serialize inventory -> JSON -> `SyncAPI.set(uuid, "inv_data", json)`.
3. On `PlayerJoinEvent`: `SyncAPI.get(uuid, "inv_data")` -> Validate vs Schema -> Deserialize -> Restore.

---

## 8. Best Practices
1. **Namespace Channels**: Always use `plugin:channel`. Do not use generic names like `update` or `test`.
2. **Async Handling**: Subscriptions run on IO threads. If you touch the Bukkit API (e.g., `player.sendMessage`), wrap it in a Sync Task using `CoreAPI`.
   ```java
   SyncAPI.subscribe("ch", (c, m) -> {
       CoreAPI.getTaskManager().runSync(() -> {
           // Safe Bukkit API usage
       });
   });
   ```
3. **Minimize Traffic**: Don't send huge payloads every tick. Use delta updates where possible.

