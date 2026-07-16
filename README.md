# BotPenguin_Connector (Magento 2)

Pushes signed Magento **store events** — orders, new customers, abandoned carts — to a configured webhook for WhatsApp automation.

- Supported: Magento Open Source & Adobe Commerce **2.4.4+**, PHP **8.1+**.
- Events: `order.created`, `order.updated`, `order.canceled`, `order.refunded`, `customer.created`, `cart.abandoned`.

## Installation

### Composer (recommended)
```bash
composer require botpenguin/module-connector
bin/magento module:enable BotPenguin_Connector
bin/magento setup:upgrade
bin/magento setup:di:compile
bin/magento cache:flush
```

### Manual (zip / app/code)
Extract to `app/code/BotPenguin/Connector`, then run the same `bin/magento` commands as above.

## Configuration
`Stores → Configuration → BotPenguin → Connector → General`:
- **Enable** — turn the connector on.
- **Webhook URL** — paste the URL from your dashboard (`…/custom-app/magento/{botApiKey}`).
- **Signing Secret** — paste the one-time secret from your dashboard (stored encrypted).

On **Save Config**, the module runs a one-time activation handshake with the webhook and shows a success/error message confirming the connection.

## How it works
- Observers on `sales_order_place_after` (created) and `sales_order_save_after` (status change → updated/canceled/refunded) build a normalized, versioned envelope and POST it to the Webhook URL.
- `customer.created` fires when the customer first becomes reachable: at registration if a phone is present, otherwise the moment their first address with a telephone is saved (exactly once per customer).
- A cron job (dedicated `botpenguin_connector` cron group, every 15 minutes) scans stale active quotes and emits `cart.abandoned` once per cart.
- Each request is signed: `X-Signature: v1=HMAC_SHA256(secret, "v1:{X-Timestamp}:{botApiKey}")`, with `X-Timestamp` (unix seconds) and `X-Delivery-Id`.
- Delivery is fail-safe (never disrupts checkout, order save, or registration).

## Uninstall
```bash
bin/magento module:disable BotPenguin_Connector
```
Disabling stops all event delivery. Configuration values remain until removed from `core_config_data`.

## License

Open Software License v3.0 (OSL-3.0) — see [LICENSE.txt](LICENSE.txt).
