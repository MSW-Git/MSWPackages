# 📜 WorldShop System

Provides a shop system that allows players to purchase items through the `WorldShopService`.

---

## Key Components

| Component | Description |
|-----------|-------------|
| `PlayerWorldShop` | Load and save purchase count to DB; handle item purchases with custom logic callback |
| `WorldShopCategoryType` | Defines shop categories |
| `WorldShopDataSet` | The main shop data table |
| `WorldShopData` | Defines the structure of WorldShopData |
| `WorldShopDataSetLogic` | Loads the shop dataset; handles shop data-related operations |
| `WorldShopLogic` | Validates and processes purchases through WorldShopService |
| `WorldShopPurchaseResultEvent` | Client-side event for purchase results from the WorldShop |

---

## Methods & Events

### PlayerWorldShop

- `onPurchase(productId)`: boolean
- `onPurchaseSuccess(productId)`: void
- `onPurchaseFailed(productId, reason)`: void
- `TryPurchaseItem(productId)`: boolean

### WorldShopLogic

- `ProcessPurchase(purchaseInfo)`: boolean
- `ProcessItemPurchase(userEntity, productId)`: boolean
- `ProcessPassPurchase(userEntity, productId)`: boolean

---

## Data Structure

```
WorldShopData {
  ProductId        : string,  -- Unique product identifier based on the world ID
  Name             : string,  -- Product name
  Desc             : string,  -- Product description
  IconRUID         : string,  -- Unique resource ID for the product icon
  Price            : integer, -- Product price
  RewardFunction   : string,  -- Name of the function called when the reward is granted
  RewardArgs       : table,   -- Arguments passed to the reward function
  PurchaseLimit    : integer, -- Maximum number of purchases allowed
  StartDate        : integer, -- Product availability start time (elapsed seconds)
  EndDate          : integer, -- Product availability end time (elapsed seconds)
  Order            : integer, -- Display order of the product
  Category         : integer  -- Product category
}
```

---

## Requirements

- The `PlayerWorldShop` component must be added to the `DefaultPlayer` entity.
- Product information must be registered in the `WorldShopDataSet`.
  - `ProductId` must be an item ID registered in the internal WorldShop management.
  - The column name used in the dataset must match the world ID where the item will be used.
  - `RewardFunction` should point to a function defined in `WorldShopLogic` that grants the reward.
  - `Category` should be defined in `WorldShopCategoryType`.

---

## Recommendation

It is recommended to set `PlayerEntityAuthorityCheck` to `true` in `WorldConfig`.
If this setting is false, the server function is exposed to all clients, which may lead to security vulnerabilities.

---

## Examples & Samples

You can test the WorldShop system using the provided samples:

- Press **K** to open the sample WorldShop UI.
- Press **L** to open the sample Admin UI.
- In the admin tool, press **Enter** after editing to apply the changes.
- The `PlayerDBManager` component must be added to the player entity.

For implementation details, refer to the following sample scripts:

- `WorldShop/Sample/WorldShopSampleLogic`
- `WorldShop/Sample/PlayerDBManager`
- `WorldShop/Sample/UI/Model_UIWorldShop`
- `WorldShop/Sample/UI/UIWorldShop`
- `WorldShop/Sample/UI/UIWorldShopItem`
- `WorldShop/Sample/UI/UIWorldShopItemPanel`
- `WorldShop/Sample/UI/UIWorldShopPassPanel`
- `WorldShop/Sample/UI/Model_UIGMWorldShopTool`
- `WorldShop/Sample/UI/UIGMWorldShopTool`
- `WorldShop/Sample/UI/UIGMWorldShopTool_Page`
- `WorldShop/Sample/UI/UIGMWorldShopTool_Item`
