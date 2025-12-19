# EcloCore Dependency Guide

> **Version**: 1.0.0
> **Type**: Core Infrastructure / Library
> **Importance**: CRITICAL (Required by all Eclo plugins)

## 1. Introduction
**EcloCore** is the foundational library and infrastructure plugin for the EclozionMC network. It provides a centralized API for database connections (SQL & Redis), command handling, task scheduling, and configuration management.

**Primary Purpose:**
- Centralize `HikariCP` (SQL) and `Jedis` (Redis) connection pools.
- Provide a standardized `@CommandInfo` annotation-based command framework.
- simplify asynchronous task management.
- Expose a singleton `CoreAPI` for easy access to all services.

---

## 2. Integration & Setup

### Maven Dependency
To use EcloCore in your plugin, add it as a provided dependency in your `pom.xml`. Since EcloCore runs on the server, you do not need to shade it.

```xml
<dependency>
    <groupId>com.eclozion.core</groupId>
    <artifactId>EcloCore</artifactId>
    <version>1.0-SNAPSHOT</version>
    <scope>provided</scope>
</dependency>
```

### plugin.yml
You must declare `EcloCore` as a dependency in your `plugin.yml` to ensure it loads before your plugin.

```yaml
name: YourPlugin
version: 1.0.0
main: com.yourname.yourplugin.YourPlugin
depend: [EcloCore] # CRITICAL: EcloCore must be loaded first
```

### Accessing the API
The entry point for all EcloCore functionality is the `CoreAPI` class.

```java
import com.eclozion.core.api.CoreAPI;
import com.eclozion.core.EcloCore;

public class YourPlugin extends JavaPlugin {
    @Override
    public void onEnable() {
        // Verify CoreAPI is available (optional, as 'depend' handles this)
        if (CoreAPI.getPlugin() == null) {
            getLogger().severe("EcloCore not found!");
            getServer().getPluginManager().disablePlugin(this);
            return;
        }
        
        getLogger().info("Connected to EcloCore successfully.");
    }
}
```

---

## 3. Database Management (SQL & Redis)

EcloCore manages global connection pools. **DO NOT** create your own HikariDataSource or JedisPool. Use the ones provided by EcloCore to save resources and prevent connection leaks.

### SQL Connection (MySQL / SQLite)
EcloCore handles the switch between MySQL and SQLite transparently based on its `config.yml`.

**Usage Pattern:**
```java
import com.eclozion.core.api.CoreAPI;
import java.sql.Connection;
import java.sql.PreparedStatement;
import java.sql.ResultSet;
import java.sql.SQLException;

public void fetchPlayerData(String playerName) {
    // Runnable for async execution (ALWAYS run DB operations async)
    CoreAPI.getTaskManager().runAsync(() -> {
        String query = "SELECT * FROM player_data WHERE name = ?";
        
        try (Connection conn = CoreAPI.getDatabaseManager().getSQLConnection();
             PreparedStatement ps = conn.prepareStatement(query)) {
             
            ps.setString(1, playerName);
            try (ResultSet rs = ps.executeQuery()) {
                if (rs.next()) {
                    int coins = rs.getInt("coins");
                    System.out.println(playerName + " has " + coins + " coins.");
                }
            }
            
        } catch (SQLException e) {
            e.printStackTrace();
        }
    });
}
```

### Redis Connection
Redis is optional in EcloCore. Always check if it's enabled before using it.

**Usage Pattern:**
```java
import com.eclozion.core.api.CoreAPI;
import redis.clients.jedis.Jedis;

public void sendCrossServerMessage(String channel, String message) {
    if (!CoreAPI.getDatabaseManager().isRedisEnabled()) {
        System.out.println("Redis is not enabled!");
        return;
    }

    CoreAPI.getTaskManager().runAsync(() -> {
        try (Jedis jedis = CoreAPI.getDatabaseManager().getRedisResource()) {
            jedis.publish(channel, message);
        } catch (Exception e) {
            e.printStackTrace();
        }
    });
}
```

---

## 4. Command Framework

EcloCore provides a powerful annotation-based command manager. You do not need to register commands in `plugin.yml` or implement `CommandExecutor` manually if you use this system correctly.

### 1. Create a Command Class
Annotate your method with `@CommandInfo`.

```java
import com.eclozion.core.command.annotation.CommandInfo;
import com.eclozion.core.command.CommandContext;
import org.bukkit.entity.Player;

public class MyCommands {

    @CommandInfo(
        name = "heal",
        usage = "/heal <player>",
        description = "Heals a player",
        permission = "myplugin.heal",
        aliases = {"h", "doctor"},
        playerOnly = true, // If true, console cannot run this
        minArgs = 0
    )
    public void onHeal(CommandContext context) {
        Player sender = (Player) context.getSender();
        
        // Handling args using helper methods in CommandContext
        // (Assuming CommandContext has args access)
        if (context.getArgs().length > 0) {
            Player target = Bukkit.getPlayer(context.getArgs()[0]);
            if (target != null) {
                target.setHealth(20);
                sender.sendMessage("Healed " + target.getName());
            } else {
                sender.sendMessage("Player not found.");
            }
        } else {
            sender.setHealth(20);
            sender.sendMessage("You have been healed.");
        }
    }
}
```

### 2. Register the Command
You must register your command class instance with EcloCore's `CommandManager`.

```java
@Override
public void onEnable() {
    // Register commands
    CoreAPI.getCommandManager().registerCommand(new MyCommands());
}
```

**Key Features of `@CommandInfo`:**
- `name`: The main command name.
- `permission`: Auto-checks permission. Returns "No permission" message if failed.
- `playerOnly`: Auto-checks if sender is a player.
- `minArgs`: Auto-checks argument usage info.

---

## 5. Task Management

Avoid using `Bukkit.getScheduler()` directly. `TaskManager` provides a simplified wrapper.

**API Methods:**
- `runAsync(Runnable)`: Run immediately on a worker thread.
- `runSync(Runnable)`: Run immediately on main thread.
- `runLater(Runnable, long delayTicks)`: Run after X ticks.
- `runTimer(Runnable, long delay, long period)`: Repeating task.

**Example:**
```java
// Good practice: Define task logic separately
Runnable logic = () -> {
    System.out.println("Saving data...");
    // DB save logic here
};

// Run it async every 5 minutes (6000 ticks)
int taskId = CoreAPI.getTaskManager().runTimerAsync(logic, 0L, 6000L);
```

---

## 6. Event Management

While you can use `Bukkit.getPluginManager().registerEvents()`, EcloCore offers `EventManager` for potential future expansions (custom event buses). Currently, it acts as a wrapper.

```java
// Standard Listener
public class JoinListener implements Listener {
    @EventHandler
    public void onJoin(PlayerJoinEvent event) {
        // logic
    }
}

// Registering
// Note: Currently CoreAPI.getEventManager().registerListeners() is internal
// For your own plugins, continue using Bukkit's standard registration 
// coupled with CoreAPI services.
getServer().getPluginManager().registerEvents(new JoinListener(), this);
```

**Custom Events:**
You can fire events using the Manager:
```java
MyCustomEvent event = new MyCustomEvent(player);
CoreAPI.getEventManager().callEvent(event);
```

---

## 7. Configuration Options

EcloCore exposes its `ConfigManager` via `CoreAPI.getConfigManager()`.
This is useful if you need to read global settings, like the server name or network formatting standards defined in Core's config.

```java
String globalDateFormat = CoreAPI.getConfigManager().getConfig().getString("settings.date-format", "dd/MM/yyyy");
```

---

## 8. Common Integration Scenarios

### Scenario A: Shared Economy Transaction with Redis Notification
1. Update SQL database to deduct coins.
2. Publish message to Redis to update other servers (proxy-wide sync).

```java
public void payUser(String fromUser, String toUser, double amount) {
    CoreAPI.getTaskManager().runAsync(() -> {
        try (Connection conn = CoreAPI.getDatabaseManager().getSQLConnection()) {
            // ... SQL Transaction logic ...
            
            // Notify network
            if (CoreAPI.getDatabaseManager().isRedisEnabled()) {
                try (Jedis jedis = CoreAPI.getDatabaseManager().getRedisResource()) {
                    jedis.publish("economy_update", fromUser + ":" + toUser + ":" + amount);
                }
            }
        } catch (SQLException e) {
            e.printStackTrace();
        }
    });
}
```

### Scenario B: Global Admin Chat
1. Create command `/adminchat`.
2. Use Redis to broadcast.

```java
@CommandInfo(name = "staffchat", permission = "core.staff")
public void onStaffChat(CommandContext ctx) {
    String msg = String.join(" ", ctx.getArgs());
    // Send to Redis
    CoreAPI.getTaskManager().runAsync(() -> {
        try (Jedis j = CoreAPI.getDatabaseManager().getRedisResource()) {
            j.publish("staff_chat", ctx.getSender().getName() + ":" + msg);
        }
    });
}
```

---

## 9. Error Handling

| Error | Cause | Solution |
|-------|-------|----------|
| `SQLException: DataSource is not initialized` | EcloCore failed to connect to DB on startup or `onEnable` order is wrong. | Check `depend: [EcloCore]` and EcloCore `config.yml`. |
| `IllegalStateException: CoreAPI already initialized!` | You are trying to set the instance. | Never call `CoreAPI.setInstance()`. |
| `CommandException` inside `onCommand` | Uncaught exception in your logic. | Wrap command logic in try/catch to prevent console spam. |

---

> **Note**: EcloCore is an internal API. Breaking changes may occur between major versions (1.x -> 2.x). Always check the CHANGELOG before updating.
