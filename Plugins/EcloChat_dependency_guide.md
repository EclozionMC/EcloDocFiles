# EcloChat Dependency Guide

> **Version**: 1.20
> **Type**: Chat Management
> **Importance**: STANDARD

## 1. Introduction
**EcloChat** manages chat channels (Global, Local, Staff, Commerce, etc.) and formatting.

## 2. Integration

### Maven
```xml
<dependency>
    <groupId>fr.eclozion.chat</groupId>
    <artifactId>EcloChat</artifactId>
    <version>1.20</version>
    <scope>provided</scope>
</dependency>
```

### Accessing API
EcloChat does not have a static API class. You must access the main class.

```java
import fr.eclozion.chat.EcloChat;

public void setChannel(Player player) {
    EcloChat chat = EcloChat.getInstance();
    chat.getChannelManager().setChannel(player, ChannelType.GLOBAL);
}
```

## 3. Key Permissions
- `eclochat.admin`: Admin prefix.
- `eclochat.mod`: Mod prefix.
