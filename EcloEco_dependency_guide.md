# EcloEco Dependency Guide

> **Version**: 1.0.0
> **Type**: Economy / Currency System
> **Importance**: HIGH (Central Economy)

## 1. Introduction
**EcloEco** manages the server's economy, including the primary currency (Coins) and the secondary PVP currency (Blood Money). It supports MySQL/SQLite and integrates with EcloSync for cross-server consistency.

## 2. Integration & Setup

### Maven Dependency
```xml
<dependency>
    <groupId>com.eclozion.eco</groupId>
    <artifactId>EcloEco</artifactId>
    <version>1.0.0</version>
    <scope>provided</scope>
</dependency>
```

### plugin.yml
```yaml
depend: [EcloCore, EcloEco]
```

---

## 3. Economy API Usage

The main entry point is `EcloEcoAPI`.

### Accessing the API
```java
import com.eclozion.eco.api.EcloEcoAPI;

EcloEcoAPI eco = EcloEcoAPI.getInstance();
```

### Checking Balance
```java
public double getBalance(Player player) {
    return EcloEcoAPI.getInstance().getBalance(player);
}

public boolean canAfford(Player player, double cost) {
    return EcloEcoAPI.getInstance().has(player, cost);
}
```
*Note: This checks a local cache and is very fast.*

### Transactions (Deposit/Withdraw)
Transactions are processed immediately in the cache and saved asynchronously.

```java
public void buyItem(Player player, double cost) {
    EcloEcoAPI eco = EcloEcoAPI.getInstance();
    
    if (eco.has(player, cost)) {
        eco.withdraw(player, cost);
        player.sendMessage("You bought the item!");
    } else {
        player.sendMessage("Insufficient funds.");
    }
}

public void giveReward(Player player, double reward) {
    EcloEcoAPI.getInstance().deposit(player, reward);
}
```

---

## 4. Blood Money API
Blood Money is a specialized currency earned via PVP.

```java
// Get
int bm = EcloEcoAPI.getInstance().getBloodMoney(player);

// Add
EcloEcoAPI.getInstance().addBloodMoney(player, 10);

// Remove
EcloEcoAPI.getInstance().removeBloodMoney(player, 5);
```

---

## 5. Event Listening
EcloEco currently does not expose a custom TransactionEvent in the public API (v1.0.0). To detect changes, you currently need to poll or rely on EcloSync channel updates if you are on a different server.

Future versions will include `EconomyTransactionEvent`.

---

## 6. Implementation Notes
- **Thread Safety**: API methods are safe to call from the main thread.
- **Persistence**: Data is saved to `SQL` via `EcloCore`.
- **Offline Players**: `getBalance(UUID)` works if the player is in the cache (online). Fetching offline balances currently returns 0.0 unless the player data is loaded. Use with caution for offline players.
