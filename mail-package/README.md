# 📬 MailPackage

This project is a package for managing the in-game mail system.
Game Masters (GMs) can send mail to all players or specific players.
Players can receive and manage their own mail.

---

## Main Features

### GMMailTool (GM Functions)

Provides the following features for GMs to manage mail:

- **Send Mail**
  - Send mail to a specific player or all players.
  - ※ If sending to a specific player, they must reconnect to reflect the mail.

- **Attach Items**
  - Items can be attached to the mail.

- **Recall Mail**
  - Sent mail can be recalled.
  - ※ If recalling from a specific player, they must reconnect to reflect the changes.

- **View Sent Mail**
  - GMs can view a list of sent mail.

### PlayerMail (Player Functions)

Provides the following features for players to manage their mail:

- **Receive Mail**
  - Players can receive mail sent to them.

- **Manage Mail**
  - Players can read and process their mail data.
  - ※ `PlayerMail` must be added to the Default Player to work correctly.
  - ※ There is a maximum number of mail slots per player. If exceeded, mails with the nearest expiration date will be automatically deleted.

- **Item Rewards**
  - Use the `handleReceiveMailItems` handler and connect a callback function to process item rewards.

---

## Mail Data Format

### Required Data When Sending Mail

| Field | Type | Description |
|-------|------|-------------|
| `Title` | String | Mail title |
| `Message` | String | Mail body |
| `Sender` | String | Sender name |
| `RetentionDays` | Integer | Number of days the mail will be retained |
| `StartDate` | Integer | Start date (UTC) |
| `EndDate` | Integer | Expiration date (UTC) |
| `Items` | Table | List of attached items |

### Data After Receiving Mail

| Field | Type | Description |
|-------|------|-------------|
| `Title` | String | Mail title |
| `Message` | String | Mail body |
| `Sender` | String | Sender name |
| `DueDate` | Integer | Expiration date (UTC) |
| `IsRead` | Bool | Read status (`true` / `false`) |
| `Items` | Table | List of attached items |

---

## Recommendation

It is recommended to set `PlayerEntityAuthorityCheck` to `true` in `WorldConfig`.
If this setting is false, the server function is exposed to all clients, which may lead to security vulnerabilities.

---

## Sample Project Guide

The sample project is designed to test GM and player mail functions using UI.

- **Press F9** → Open GM Mail UI
  - Use the TimeOffset option at the top-left of the UI to adjust time.
  - ※ Time is based on UTC.

- **Press F10** → Open Player Mailbox UI
  - Check received mail and handle attached items.

- Sample code for item rewards is written in `PlayerInventory`.
