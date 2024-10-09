# FastInvNewGen

Lightweight and easy-to-use inventory API for PaperMC plugins with Adventure support.

**FastInvNewGen** is a fork of the original [FastInv](https://github.com/MrMicky-FR/FastInv), adapted for use with PaperMC and Adventure Components.

## Features

- Lightweight and easy-to-use (less than 400 lines of code with JavaDoc).
- **Supports Adventure Components** for inventory titles and item descriptions.
- Compatible with Minecraft versions starting from **1.21.1**.
- Supports custom inventories (size, title, and type).
- Option to prevent players from closing the inventory.
- Custom pagination support for inventories.
- Direct access to Bukkit inventories for advanced usage.

## Installation

FastInvNewGen is currently not available through a package manager. You can copy the necessary classes directly into your project or use a local dependency.

### Manual Installation

1. Copy the `FastInv.java`, `FastInvManager.java`, and `PaginatedFastInv.java` classes into your plugin project.
2. Optionally, copy the `ItemBuilder.java` if you need easy item manipulation.

## Usage

### Register FastInv

Before creating inventories, you need to register your plugin by adding `FastInvManager.register(this);` in the `onEnable()` method of your plugin:

```java
@Override
public void onEnable() {
    FastInvManager.register(this);
}
```

### Creating an Inventory Class

Now you can create an inventory by creating a class that extends `FastInv`, and adding items in the constructor. You can also override `onClick`, `onClose`, and `onOpen` if needed.

Here's an example inventory using **Adventure Components**:

```java
import fr.mrmicky.fastinv.FastInv;
import fr.mrmicky.fastinv.ItemBuilder;
import net.kyori.adventure.text.Component;
import net.kyori.adventure.text.format.NamedTextColor;
import org.bukkit.Material;
import org.bukkit.event.inventory.InventoryClickEvent;
import org.bukkit.event.inventory.InventoryCloseEvent;
import org.bukkit.event.inventory.InventoryOpenEvent;
import org.bukkit.inventory.ItemStack;

public class ExampleInventory extends FastInv {

    private boolean preventClose = false;

    public ExampleInventory() {
        super(45, Component.text("Example inventory").color(NamedTextColor.GOLD));

        // Add an item using Adventure Components
        setItem(22, new ItemBuilder(Material.IRON_SWORD)
                .name(Component.text("Click me").color(NamedTextColor.RED))
                .build(), e -> e.getWhoClicked().sendMessage(Component.text("You clicked the sword").color(NamedTextColor.GREEN)));

        // Add borders
        setItems(getBorders(), new ItemBuilder(Material.LAPIS_BLOCK).name(Component.empty()).build());

        // Add an item to toggle inventory closing
        setItem(34, new ItemBuilder(Material.BARRIER)
                .name(Component.text("Prevent close").color(NamedTextColor.RED))
                .build(), e -> this.preventClose = !this.preventClose);

        // Prevent the player from closing the inventory if `preventClose` is true
        setCloseFilter(p -> this.preventClose);
    }

    @Override
    public void onOpen(InventoryOpenEvent event) {
        event.getPlayer().sendMessage(Component.text("Inventory opened").color(NamedTextColor.GOLD));
    }

    @Override
    public void onClose(InventoryCloseEvent event) {
        event.getPlayer().sendMessage(Component.text("Inventory closed").color(NamedTextColor.GOLD));
    }

    @Override
    public void onClick(InventoryClickEvent event) {
        // Handle click events
    }
}
```

The inventory can be opened with the `open(player)` method:

```java
new ExampleInventory().open(player);
```

### Paginated Inventory

FastInvNewGen also supports paginated inventories using `PaginatedFastInv`.

```java
import org.bukkit.Material;
import org.bukkit.inventory.ItemStack;
import net.kyori.adventure.text.Component;
import net.kyori.adventure.text.format.NamedTextColor;

public class ExamplePaginatedInventory extends PaginatedFastInv {

    public ExamplePaginatedInventory() {
        super(27, Component.text("Paginated inventory").color(NamedTextColor.GOLD));

        previousPageItem(20, p -> new ItemBuilder(Material.ARROW)
            .name(Component.text("Page " + p).color(NamedTextColor.WHITE)).build());
        nextPageItem(24, p -> new ItemBuilder(Material.ARROW)
            .name(Component.text("Page " + p).color(NamedTextColor.WHITE)).build());

        for (int i = 1; i < 64; i++) {
            addContent(new ItemStack(Material.GOLDEN_APPLE, i), 
                e -> e.getWhoClicked().sendMessage(Component.text("Clicked item " + i).color(NamedTextColor.YELLOW)));
        }
    }

    @Override
    protected void onPageChange(int page) {
        setItem(18, new ItemBuilder(Material.PAPER)
            .name(Component.text("Current page " + page).color(NamedTextColor.AQUA)).build());
    }
}
```

The paginated inventory can be opened with:

```java
new ExamplePaginatedInventory().open(player);
```

### Compact Inventory Example

For small, simple inventories, you can create a "compact" inventory directly:

```java
FastInv inv = new FastInv(InventoryType.DISPENSER, Component.text("Compact inventory"));

inv.setItem(4, new ItemStack(Material.NAME_TAG), e -> e.getWhoClicked().sendMessage(Component.text("Name tag clicked").color(NamedTextColor.GREEN)));
inv.addClickHandler(e -> e.getWhoClicked().sendMessage(Component.text("Clicked slot " + e.getSlot())));
inv.addCloseHandler(e -> e.getWhoClicked().sendMessage(Component.text("Inventory closed").color(NamedTextColor.YELLOW)));
inv.open(player);
```

### Adventure Components Support

FastInvNewGen fully supports [Adventure Components](https://github.com/KyoriPowered/adventure) for inventory titles and item names. This allows for rich text formatting and translations.

Here’s how to use Adventure Components for an inventory title:

```java
Component title = Component.text("Hello World").color(NamedTextColor.GOLD);
FastInv inv = new FastInv(owner -> Bukkit.createInventory(owner, 27, title));
```

## Notes

- **Deprecated features**: String-based item names and `ChatColor` usage are discouraged. Please use Adventure Components instead.
- FastInvNewGen is compatible with **PaperMC** versions starting from 1.21.1.
