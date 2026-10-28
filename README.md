# Inventory Desk — MySQL Trigger Lab

A small PHP and MySQL inventory application with a public customer store and a separate admin desk, as well as exactly three database triggers:

1. `AFTER INSERT` records new products in `activity_logs`.
2. `AFTER UPDATE` records old and new prices in `price_history` when the price changes.
3. `BEFORE DELETE` copies a product into `deleted_products` before removal.

Customers can pay for purchases manually through QR Ph or borrow available stock with a selected return date. Admins manage the product catalog, monitor purchases and loans, and record returns. The PHP application does **not** write to the trigger-owned log, price-history, or archive tables.

## Files

- `schema.sql` — creates the database tables, sample categories, customer order/loan records, and the three triggers.
- `bootstrap.php` — applies secure session-cookie settings and provides shared CSRF and HTML-escaping helpers.
- `config.php` — database connection settings.
- `index.php` — public customer storefront and buy/borrow request handling.
- `admin.php` — separate admin sign-in handler and inventory/order/loan dashboard.
- `login.php` — demo admin sign-in form.
- `template.php` — admin dashboard template and trigger results.
- `qr_checkout.php` — displays order totals, QR Ph demo branding, and a decorative non-scannable sample QR graphic.
- `script.js` — sign-in demo interactions and the animated background.
- `styles.css` — website styles.

## Run with XAMPP

1. Install and open XAMPP. Start **Apache** and **MySQL** from the XAMPP Control Panel.
2. Copy the project folder into XAMPP's web root:
   - macOS: `/Applications/XAMPP/xamppfiles/htdocs/inventory-trigger-lab`
   - Windows: `C:\xampp\htdocs\inventory-trigger-lab`
   - Linux: commonly `/opt/lampp/htdocs/inventory-trigger-lab`
3. Open `http://localhost/phpmyadmin` in a browser.
4. In phpMyAdmin, open the **Import** tab, choose this project's `schema.sql`, and select **Import** / **Go**. The script creates and selects `inventory_db`, creates its tables, seeds the categories, and installs the three triggers.
5. Check `config.php`. Its defaults are for a typical local XAMPP MySQL setup: host `127.0.0.1`, port `3306`, database `inventory_db`, user `root`, and an empty password. If your MySQL credentials differ, update those defaults or set the `DB_HOST`, `DB_PORT`, `DB_NAME`, `DB_USER`, and `DB_PASSWORD` environment variables.
6. Visit `http://localhost/inventory-trigger-lab/` for the public customer store.
7. Visit `http://localhost/inventory-trigger-lab/login.php` for the separate admin sign-in. The demo accepts any valid email and non-empty password.

For WAMP, put the folder in its `www` directory and open `http://localhost/inventory-trigger-lab/`. Ensure PHP has the PDO MySQL extension enabled.

To run this workspace directly on macOS with XAMPP's PHP, start MySQL in XAMPP, open Terminal, and run:

```bash
/Applications/XAMPP/xamppfiles/bin/php -S 127.0.0.1:8088 -t /Users/khryzz/palmero.proj
```

Keep the Terminal window open and visit `http://127.0.0.1:8088/`. Press **Control+C** in that Terminal window to stop the PHP server.

## QR Ph payment demo

The checkout uses a local QR Ph-style demo wordmark (`qrph-logo.svg`) and a decorative sample pattern (`qrph-demo-code.svg`). The sample pattern is intentionally not a real, scannable payment QR code and cannot accept payments. Customers are redirected to `qr_checkout.php` after placing an order. This is a visual school-project demo only; do not mark demo orders as real payments.

Borrowing immediately reduces available stock and creates a loan record. From the admin desk, mark a loan returned to restore its quantity. Products cannot be deleted while loans or reserved checkouts are active.

## How the page files are separated

- `index.php` is the public customer catalog and handles purchases and borrowing.
- `admin.php` protects product and customer-activity management with the demo session and validates admin form input.
- `login.php` submits the CSRF-protected demo admin sign-in form. Sign-in is for demonstration only; it accepts any valid email and non-empty password, so it is not production authentication.
- `template.php` contains the admin dashboard and displays values provided by `admin.php`.
- `qr_checkout.php` displays the order total and QR Ph demo branding with a non-scannable sample graphic.
- `styles.css` contains shared storefront, admin, and sign-in styles.
- `script.js` powers the demo form interactions. Social sign-in and password reset are informational placeholders and are not connected to external services.

## Step-by-step trigger demonstration

1. **INSERT trigger:** Sign in to the admin desk, then add a product, for example `Wireless Mouse`, SKU `DEMO-001`, category `Electronics`, price `599.00`, and stock `25`. Submit the form. The product appears in the customer store and the activity log shows the database-generated entry.
2. **UPDATE trigger:** Edit that product's price to `649.00` and click **Save changes**. The price-history section shows the old price (`599.00`) and new price (`649.00`). To confirm that it only logs a price change, edit the description without changing the price; no additional price-history row should appear.
3. **DELETE trigger:** Click **Delete product** for the product and confirm. It disappears from the current product list and appears in the deleted-product archive.
4. **CRUD check:** Adding is Create, the product cards and history sections are Read, **Save changes** is Update, and **Delete product** is Delete. All three trigger demonstrations are initiated from the website.

## Verify in phpMyAdmin

Select the `inventory_db` database and open the **SQL** tab. To confirm the three triggers:

```sql
SHOW TRIGGERS FROM inventory_db;
```

To inspect what the website actions caused:

```sql
SELECT * FROM activity_logs ORDER BY log_id DESC;
SELECT * FROM price_history ORDER BY history_id DESC;
SELECT * FROM deleted_products ORDER BY archive_id DESC;
```

The website does not issue inserts to these three tables. MySQL runs the corresponding trigger as part of the product `INSERT`, `UPDATE`, or `DELETE`.

## Explaining the trigger code

- `NEW` refers to the row being inserted or the new version of an updated row.
- `OLD` refers to the previous version of an updated row or the row about to be deleted.
- The insert trigger runs `AFTER INSERT`, so MySQL has assigned the product ID before it logs it.
- The update trigger uses MySQL's null-safe comparison operator `<=>` and writes a history row only if `OLD.price` and `NEW.price` differ.
- The delete trigger runs `BEFORE DELETE`, so it can copy the product details before MySQL removes the product.
- The trigger inserts and product change are part of the same database operation. If a trigger insert fails, the product change fails too because the tables use InnoDB.

The schema is safe to import again for this exercise: categories are inserted only if their unique names are not present, and the three named triggers are replaced. Existing table definitions and product data are not reset.


The checkout includes a locally bundled QR Ph-style logo graphic based on the supplied reference, so the logo does not depend on an external image host. This is a recreated graphic, not the original downloaded image file. The QR code remains a non-scannable demo.
