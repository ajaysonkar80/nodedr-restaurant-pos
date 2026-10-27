# OrderRestro — the complete restaurant tutorial

This tutorial walks through every screen in Nodedr OrderRestro using the seeded demo restaurant. Each section starts with the click path, explains what each state means, and includes a flowchart for the longer flows. Screenshots are from the running demo, so your amounts, guests and order numbers will differ.

> **Demo access**
>
> - App: **http://localhost:3000**
> - Email: **owner@demo.local**
> - Password: **Password123!**
> - Restaurant: **Nodedr Bistro (Demo)** · Branch: **Main Branch**
> - API docs: **http://localhost:4001/api/docs**
>
> Screenshots live in [`docs/tutorial/`](./docs/tutorial/). The seeded data includes menu items, eight tables, customers, staff, suppliers and a few open orders so every screen has something to show.

**Contents**

1. [What the app does](#1-what-the-app-does)
2. [Run it and sign in](#2-run-it-and-sign-in)
3. [Navigation and permissions](#3-navigation-and-permissions)
4. [Core concepts](#4-core-concepts)
5. [A day of service, end to end](#5-a-day-of-service-end-to-end)
6. [Point of Sale and billing](#6-point-of-sale-and-billing)
7. [UPI QR payments and the customer display](#7-upi-qr-payments-and-the-customer-display)
8. [Orders](#8-orders)
9. [Tables, waitlist and QR ordering](#9-tables-waitlist-and-qr-ordering)
10. [Reservations](#10-reservations)
11. [Menu, combos and stock tracking](#11-menu-combos-and-stock-tracking)
12. [Kitchen Display](#12-kitchen-display)
13. [Customers and loyalty](#13-customers-and-loyalty)
14. [Inventory and procurement](#14-inventory-and-procurement)
15. [Delivery](#15-delivery)
16. [Team, attendance and payroll](#16-team-attendance-and-payroll)
17. [Accounting](#17-accounting)
18. [Reports, exports and scheduled emails](#18-reports-exports-and-scheduled-emails)
19. [Marketing](#19-marketing)
20. [Operations](#20-operations)
21. [Settings](#21-settings)
22. [Guest ordering, kiosk and customer display](#22-guest-ordering-kiosk-and-customer-display)
23. [Roles and permissions reference](#23-roles-and-permissions-reference)
24. [Use-case playbook](#24-use-case-playbook)
25. [Troubleshooting and backups](#25-troubleshooting-and-backups)

---

## 1. What the app does

OrderRestro is a self-hosted restaurant management system. It covers the whole service loop, not just billing:

| Module | What it is for | Route | Sidebar permission |
| --- | --- | --- | --- |
| Dashboard | Today's revenue, orders, tables, kitchen queue, trends | `/dashboard` | `dashboard.sales.view` |
| Point of Sale | Build orders, send to kitchen, take payment | `/pos` | `orders.create` |
| Orders | Accept online orders, edit or cancel open bills, refunds | `/orders` | `orders.create` |
| Tables | Visual floor plan, table status, merge, QR codes, waitlist | `/tables` | `tables.view` |
| Reservations | Bookings, arrivals, cancellations | `/reservations` | `reservations.manage` |
| Menu | Categories, items, prices, combos, stock tracking | `/menu` | `menu.view` |
| Kitchen Display | Live kitchen tickets by station | `/kds` | `kds.manage` |
| Customers | Profiles, loyalty, credit, gift cards, history | `/customers` | `customers.manage` |
| Inventory | Ingredients, suppliers, POs, procurement, waste, transfers | `/inventory` | `inventory.manage` |
| Delivery | Driver assignment and delivery status | `/delivery` | `delivery.manage` |
| Team | Shifts, leave, payroll, performance | `/team` | `staff.manage` |
| Accounting | Expenses, cash close, P&L, balance sheet, budgets, bank, GST | `/accounting` | `accounting.manage` |
| Reports | 18 reports with print/export/email schedules | `/reports` | `reports.access` |
| Marketing | Coupons, happy hours, campaigns | `/marketing` | `marketing.manage` |
| Operations | Equipment servicing and documents | `/operations` | `operations.manage` |
| Attendance | Clock in/out, own hours, leave requests | `/attendance` | none (always visible) |
| Settings | Restaurant, branch, staff, roles, security, backups | `/settings` | `settings.manage` |

Guest-facing surfaces sit outside the sidebar:

- `/order/<qrToken>` — the menu a guest opens by scanning a table QR code.
- `/kiosk/<token>` — the self-service kiosk.
- `/customer-display` — the cashier's second screen.

Anything not listed in the sidebar is still reachable by URL, but the API enforces the same permissions.

![Landing page](docs/tutorial/landing.png)

The public landing page links to sign-in and to the built-in manual at `/docs`:

![Built-in documentation](docs/tutorial/docs.png)

---

## 2. Run it and sign in

### Production install (Docker)

On the machine that will stay in the restaurant:

```bash
git clone https://github.com/Raktim94/nodedr-restaurant-pos.git
cd nodedr-restaurant-pos
./install.sh            # add --demo for sample data
```

Open `http://<server-ip>:1995`. `./install.sh --demo` seeds the same demo login shown at the top of this page. For CasaOS/ZimaOS use the manifest in [`casaos/`](./casaos/README.md). If you put the app behind HTTPS, set `COOKIE_SECURE=true` in `.env`; otherwise the browser drops the session cookie and login loops with "Unauthorized".

### Local development (the setup used for these screenshots)

Requires Node 22+, pnpm and PostgreSQL.

```bash
pnpm install
pnpm --filter @nodedr-restaurant/types build

# apps/backend/.env
# DATABASE_URL="postgresql://user:pass@localhost:5432/nodedr_restaurant"
# JWT_SECRET="<64 random characters>"
# PORT=4001
# CORS_ORIGIN="http://localhost:3000"

pnpm --filter @nodedr-restaurant/backend exec prisma generate
pnpm --filter @nodedr-restaurant/backend exec prisma migrate deploy
pnpm --filter @nodedr-restaurant/backend exec prisma db seed

pnpm --filter @nodedr-restaurant/backend run dev
pnpm --filter web exec next dev -p 3000
```

The web app proxies `/api` and `/socket.io` to the backend, so the browser only needs port 3000. Swagger is at `http://localhost:4001/api/docs`.

### Sign in

![Sign in](docs/tutorial/login.png)

1. Open the app URL.
2. Enter **Email** and **Password**.
3. Press **Sign in**.
4. If the account has two-factor authentication, the **Verification code** field appears; enter the 6-digit code (or a recovery code).

**First run / new restaurant:** choose **Create an account** on the sign-in page.

![Create your restaurant](docs/tutorial/signup.png)

Fill **Restaurant name**, **First branch name**, **Your name**, **Email** and **Password**, then submit. This creates the restaurant, the first branch and the owner account. The owner gets every permission.

> Locked out? Another owner/administrator can reset 2FA under **Settings → Staff**, or disable it under **Settings → Security** with the password and a current code. There is no email-based "forgot password" flow in this build; a server administrator resets the password in the database.

### Where you land

![Dashboard](docs/tutorial/dashboard.png)

The dashboard is the service cockpit:

- **Today's Revenue**, **Today's Orders**, **Occupied Tables**, **Kitchen Queue**.
- **Revenue - last 14 days** and **Food waste cost - last 14 days** charts.
- **Tables** by status (Available, Occupied, Reserved, Cleaning) and the kitchen queue by ticket status.
- **Recent transactions** with a refund action on each row.
- **Today, self-serve vs staff-entered**: **Online (QR / kiosk)** and **Offline (dine-in / takeaway / phone)**.
- A banner at the top whenever online orders are waiting for acceptance.

It is responsive; the same screens work on a phone:

<table>
<tr>
<td><img src="docs/tutorial/dashboard-dark.png" alt="Dashboard in dark mode"></td>
<td><img src="docs/tutorial/dashboard-mobile.png" alt="Dashboard on a phone" width="260"></td>
</tr>
</table>

Useful dashboard shortcuts: **New order**, **Tables**, **New reservation**, **Kitchen display**, **Menu**.

---

## 3. Navigation and permissions

The left sidebar is generated from the current user's permissions. If a module is missing, the account does not hold its permission — the route is protected too, so typing the URL does not help.

- **Branch switcher** at the top (for multi-branch restaurants). Every screen is scoped to the selected branch. Tables, menu, orders, stock and staff are per branch.
- **Language** menu in the header. The staff UI and guest QR page support English, Spanish and French; the kiosk offers Spanish and English.
- **Notification bell** for online-order alerts; a dot appears on **Orders** when new online orders are waiting.
- **Theme** toggle for light/dark.
- **User menu**: profile, security and sign out.
- **Mobile**: the sidebar collapses into a bottom navigation bar; the POS shows the menu full screen with the cart as an overlay.

The demo owner sees every module. A waiter sees Point of Sale, Orders, Tables, Menu (read-only) and Attendance; a kitchen-staff account sees Kitchen Display and Attendance, and a chef adds the editable Menu. This is intentional — see [section 23](#23-roles-and-permissions-reference).

### Permission gates you will meet

- Buttons disappear when the account lacks the permission: **Discount %**, **Cancel order**, **Edit bill**, report exports, **Refund**, settings saves.
- The server checks again. Hiding a button is convenience, not security.
- Custom roles can mix any permissions, including read-only vs manage pairs.

---

## 4. Core concepts

### Channels and order types

| Term | Meaning |
| --- | --- |
| **Channel: Staff / Offline** | An order keyed in by staff at the POS. |
| **Channel: Online** | An order placed by a guest from the table QR page, the kiosk, or a website/API integration. |
| **Dine-in** | Attached to a table; can be billed later. |
| **Takeaway** | No table; usually paid when ordered. |
| **Delivery** | Needs an address, pincode and phone; appears in **Delivery** for driver assignment. |
| **QR_ORDER / KIOSK** | The two self-service online order types; website/API orders keep their fulfilment type with channel Online. |

### Order lifecycle

An order is `OPEN` until it is paid or cancelled. Website/API orders can carry an acceptance state (`PENDING → ACCEPTED` or `PENDING → REJECTED`); a rejected order is never cooked and nothing is charged. QR and kiosk orders skip that gate (`NOT_REQUIRED`) and go straight to the kitchen; staff accept the KDS ticket instead.

```mermaid
flowchart TD
  A["Guest or staff starts an order"] --> B{"Source"}
  B -->|Staff POS| C["Staff adds items in POS"]
  B -->|Website/API| D["Order arrives PENDING"]
  B -->|Table QR or kiosk| E["Accepted automatically"]
  D --> F{"Accept?"}
  F -->|Accept| G["Order OPEN, KOT created"]
  F -->|Decline| R["Rejected, nothing charged"]
  E --> G
  C --> G
  G --> H["Kitchen prepares"]
  H --> I["Guest is served"]
  I --> J{"Paid?"}
  J -->|No| K["Bill stays open on the table or in Orders"]
  K --> J
  J -->|Yes| L["Order PAID, stock deducted, receipt"]
  G --> M["Cancel before payment"]
  M --> N["Order CANCELLED, tickets withdrawn, table freed"]
```

### Kitchen tickets (KOTs)

Sending an order to the kitchen creates one or more tickets, routed to the kitchen station configured on each menu item. Ticket states:

`NEW → ACCEPTED → PREPARING → READY → SERVED`

From the KDS, **Mark served** closes the ticket after it leaves the pass. A ticket can be **rejected** before or during preparation with a reason (out of stock, item unavailable, kitchen too busy, equipment problem). Rejection does not cancel the bill; it tells the floor that an item cannot be made.

### Tables

| Status | Meaning |
| --- | --- |
| **Available** | Free to seat a party. |
| **Occupied** | Has at least one open bill. |
| **Reserved** | Held for a booking. |
| **Cleaning** | Just vacated; not yet ready. |

A table can hold more than one open bill (for example two unrelated parties); the POS asks which bill new items belong to when that happens.

### Bill and payment

- Menu prices are either tax-inclusive or tax-exclusive per branch. The checkout preview always shows the authoritative total.
- **Subtotal**, **discount**, **coupon**, **happy hour**, **tip**, **loyalty redemption**, **gift card**, **tax**, **delivery fee** and **total** are computed on the server.
- A bill can take **several payments** (split tender). Each payment is recorded; the order becomes `PAID` only when the balance reaches zero.
- **Split between N guests** is a calculator: it shows the equal share, then you take each guest's share as a part payment, or collect the whole amount once.
- Paid bills are locked. Corrections go through a **refund**, not an edit.
- Cancelling an order after the kitchen has started requires the cancel permission; the UI warns and asks for confirmation.

### Stock

Menu items can be **Not tracked**, tracked by **Recipe** (ingredients deducted per sale), or as a **Stock item** (one unit of a bottled drink or packaged good per sale). A sale is never blocked by a stock shortfall — stock can go negative so service never stops — but the shortfall is visible in inventory.

---

## 5. A day of service, end to end

```mermaid
flowchart TD
  A["Open the app: Dashboard"] --> B["Check kitchen queue, low stock and today's bookings"]
  B --> C{"How is the guest arriving?"}
  C -->|Walk-in| D["Tables: pick a free table"]
  C -->|Reservation| E["Reservations: mark Arrived"]
  C -->|QR order| F["Orders: accept the incoming order"]
  C -->|Takeaway or delivery| G["POS: choose Takeaway or Delivery"]
  D --> H["POS: add items"]
  E --> H
  F --> I
  G --> H
  H --> I["Send to kitchen"]
  I --> J["KDS: Accept, Start preparing, Mark ready, Mark served"]
  J --> K["Serve the guest"]
  K --> L["POS or Tables: take payment"]
  L --> M["Print bill or receipt"]
  M --> N["Tables: optional Cleaning, then Available"]
  N --> O["End of day: Accounting cash close"]
  O --> P["Reports: sales, tax, staff, stock"]
```

1. **Before opening:** Dashboard → check the kitchen queue is empty and low-stock items are known. Reservations → check the day's book. Tables → confirm the floor is clean and available.
2. **During service:** take orders in POS, accept online orders in Orders or KDS, watch tickets in KDS, and settle bills from POS, Tables (**Bill this table**) or Orders.
3. **After each table:** the last paid bill returns the table to **Available** automatically. If you track bussing, set **Cleaning** and then **Available** once reset.
4. **Close:** log waste, receive deliveries, count the drawer in **Accounting → Cash close**, then review **Reports**.
5. **Weekly/monthly:** run payroll, reconcile the bank, review P&L, budgets, tax and customer retention.

---

## 6. Point of Sale and billing

![Point of Sale](docs/tutorial/pos.png)

The POS has three areas:

- **Left:** categories and the menu grid. The search box filters as you type.
- **Middle/right:** the cart (before an order is sent) or the checkout panel (after it is sent).
- **Top:** the kitchen status strip (**New** / **Accepted**), and **Customer display** which opens the second screen in a new window.

### 6.1 Dine-in order

```mermaid
flowchart TD
  A["Point of Sale"] --> B["Dine-in selected"]
  B --> C["Tap Table 1 to 8"]
  C --> D{"Table has an open bill?"}
  D -->|No| E["Add items to the cart"]
  D -->|Yes| F["Choose which open bill receives the new round"]
  F --> E
  E --> G["Send to kitchen"]
  G --> H["Order OPEN, KOT printed or shown on KDS"]
  H --> I["Checkout panel: discount, tip, coupon, gift card, customer"]
  I --> J["Choose Cash, Card, UPI or Other"]
  J --> K["Complete payment"]
  K --> L["Receipt, stock deducted, table can be cleared"]
```

1. Open **Point of Sale**.
2. Keep **Dine-in** selected.
3. Tap a table tile. Green tiles are available; red are occupied; amber are reserved; grey are cleaning.
4. Tap menu items. Use the **+ / -** stepper to change quantity. Items with a half portion price ask for a portion: **Half**, **Full**, and **Custom** when custom portions are enabled.
5. Review the cart. The total follows the branch's tax mode: tax is either included in menu prices or added at checkout.
6. Press **Send to kitchen**. The button reads **Add to order & send to kitchen** when the table already has a bill and you are adding a round.
7. The order number appears in a confirmation. The right panel switches to checkout, but payment is optional now.
8. Optionally attach a customer (requires `customers.manage`), apply **Discount %**, **Tip**, **Coupon code**, **Gift card code** or **Redeem loyalty points**.
9. Pick a payment method: **Cash**, **Card**, **UPI** or **Other**.
10. For cash, enter **Cash received**; the button shows the change. For other methods enter **Amount to pay now** if it differs from the total.
11. Press **Complete payment** (or **Complete payment — change ₹X**).
12. Print the receipt, or press **New order** / **Done** to return to the floor.

![Menu grid](docs/tutorial/pos-menu.png)

![Cart before sending](docs/tutorial/pos-cart.png)

![Checkout panel](docs/tutorial/pos-checkout.png)

Every line can be removed from the cart before sending. After sending, use **Edit bill** (see [Orders](#8-orders)) for corrections.

### 6.2 Takeaway

```mermaid
flowchart TD
  A["Point of Sale"] --> B["Choose Takeaway"]
  B --> C["Add items"]
  C --> D["Send to kitchen"]
  D --> E["Attach customer if known"]
  E --> F["Take payment"]
  F --> G["Receipt"]
```

1. Open **Point of Sale**.
2. Select **Takeaway** above the menu.
3. Add items and press **Send to kitchen**.
4. Optionally attach a customer by name or phone; **New customer** creates one without leaving the till.
5. Take payment and print the receipt.

Takeaway orders are not tied to a table, so they never change floor status.

### 6.3 Delivery order

1. Open **Point of Sale** and select **Delivery**.
2. Enter the delivery details: **address**, **pincode** and **phone**. The order cannot be sent without them.
3. Add items and press **Send to kitchen**.
4. Take payment (prepaid) or leave the bill open if the driver collects cash.
5. The order now appears in **Delivery → Active** with status **Needs driver**.
6. Assign a driver and move it through **Picked up → Delivered**, or **Mark failed**.

The delivery fee comes from the matching delivery zone (configured in **Delivery → Zones**), with the zone's minimum order and ETA.

### 6.4 Adding another round to an open table

1. **Tables** → tap an occupied table → **Add another round**, or tap the occupied table directly in POS.
2. If the table has one open bill, it loads automatically. If it has several, choose the bill.
3. Add items and press **Add to order & send to kitchen**.
4. A new KOT is created for only the new items; the existing bill totals update.

Use **Bill this table** from the table tile to settle without walking back through the menu.

### 6.5 Discounts, coupons, loyalty, gift cards and tips

| Field | What it does | Permission / setup |
| --- | --- | --- |
| **Discount %** | Manual percentage discount on the bill. | `discounts.apply` |
| **Tip** | Adds a tip to the total; tips are pooled for payroll by hours. | — |
| **Coupon code** | Applies an active coupon; **Apply** validates before the bill recalculates. | Marketing → Coupons |
| **Gift card code** | **Check** looks up the balance and applies it to the bill. | Existing gift card |
| **Redeem loyalty points** | Converts the customer's points using the earn/redeem values from Settings. | Customer attached; loyalty configured |
| **Attach customer** | Links the bill to a profile for points and history. | `customers.manage` (search or create) |

A coupon never stacks with a happy-hour discount — the larger one applies. Happy hours are applied automatically during their configured window in the restaurant's timezone.

Gift cards can be redeemed at checkout, but this build has no screen to issue a new gift card. Store credit is issued by refunds and shown on the customer profile; there is also no checkout button to spend it. Use the API if you need to create gift cards programmatically.

### 6.6 Split and partial payments

```mermaid
flowchart TD
  A["Bill with balance due"] --> B{"Paying all at once?"}
  B -->|Yes| C["Enter amount and Complete payment"]
  B -->|No| D["Set Split between N guests"]
  D --> E["Enter each guest's share as a part payment"]
  E --> F["Record partial payment — ₹X"]
  F --> G{"Balance zero?"}
  G -->|No| E
  G -->|Yes| H["Complete payment for the remainder"]
  C --> I["Order PAID"]
  H --> I
```

1. With the bill open, set **Split between** to the number of guests. The per-guest amount appears.
2. Enter the first guest's share in the amount field and press **Record partial payment — ₹X**.
3. Repeat for each guest. **Payments so far** lists each payment and method.
4. When the balance is zero, press **Complete payment** for the final portion (or let the last part payment close it).

Part payments are real payments: they are recorded immediately against the order. The order stays open until fully paid.

### 6.7 Cash, card, UPI and other

- **Cash** — enter **Cash received** to calculate change. The cash drawer kicks when you print the paid receipt on the thermal printer if the bill had a cash payment; test the wiring with the drawer button under **Settings → Thermal printer**.
- **Card** — record the amount; the card terminal is separate.
- **UPI** — a QR for the exact amount is shown at the till and mirrored on the customer display. See [section 7](#7-upi-qr-payments-and-the-customer-display).
- **Other** — wallets, bank transfer, staff meal, etc.

Payment methods are only labels for reporting; no gateway is contacted. **Complete payment** is the cashier confirming the money arrived.

### 6.8 Printing

Before payment, the checkout panel offers:

- **KOT (browser)** — opens a print-friendly kitchen ticket in a new window.
- **KOT (thermal)** — sends ESC/POS directly to the configured USB or network printer.
- **Print bill** — browser print dialog for the guest bill.
- **Print bill (thermal)** — direct thermal print.

After payment the panel shows **Print receipt (browser)** and **Thermal receipt**. Configure the printer under **Settings → Thermal printer** (USB on the server host, or a network printer on port 9100) and use the test print/cash drawer buttons there.

### 6.9 Back, cancel and corrections

- **Back** returns to the floor without paying. The order stays open and is reachable from **Tables** or **Orders**.
- **Edit bill** (open bills with no payments yet) lets you remove or reduce lines before the kitchen starts them, move the bill to another free table, or fix guest details.
- **Cancel order** withdraws the kitchen tickets, frees the table and records a reason. If the kitchen already started, the confirmation warns you and requires `orders.cancel`.
- **Paid bills cannot be edited.** Use a refund from **Dashboard → Recent transactions** or the order detail.

### 6.10 A complete worked example

> Table 3 orders two Virgin Mojitos (₹220 each) and one Margherita Pizza (₹420). The bill is ₹860 including tax.

1. POS → **Dine-in** → **Table 3**.
2. Tap **Virgin Mojito** twice, then **Margherita Pizza** once.
3. **Send to kitchen** → order #20261027-0002 is open.
4. Guest asks to split three ways. Set **Split between** to 3 (₹286.67 each).
5. Take ₹300 cash from guest 1 → **Record partial payment — ₹286.67**, change ₹13.33.
6. Take the remaining two shares the same way.
7. **Complete payment** on the last share; print the receipt.
8. **Tables** → Table 3 has returned to **Available** automatically. Set **Cleaning** first if you want the bussing state, then **Available**.

---

<a id="upi-qr-payments-and-the-customer-display"></a>

## 7. UPI QR payments and the customer display

OrderRestro builds a standard UPI payment link for the exact amount due. There is no gateway fee and no amount to type into the guest's app.

### Setup

1. **Settings → Restaurant**.
2. Enter **UPI id** (for example `yourshop@okbank`) and **UPI payee name**.
3. Press **Save restaurant**.
4. Optionally open **Customer display** in a second window on the same till machine (drag it to the second monitor) and tap it once so the browser allows sound. It uses a browser-local channel, so a separate tablet will not follow the till.

### Take a UPI payment

```mermaid
flowchart TD
  A["Open bill at checkout"] --> B["Select UPI"]
  B --> C["QR for exact amount appears"]
  C --> D["QR mirrors on customer display"]
  D --> E["Guest scans with any UPI app"]
  E --> F["Cashier checks bank app or soundbox"]
  F --> G["Complete payment"]
  G --> H["Payment recorded, chime, thank-you screen"]
```

1. At checkout choose **UPI**.
2. A QR for the bill amount appears in the panel. The same QR appears full-screen on the customer display.
3. The guest scans it with GPay, PhonePe, Paytm or any UPI app.
4. Check your bank app or soundbox for the credit. The app does **not** read your bank confirmation.
5. Press **Complete payment**. The order is recorded and the customer display switches to **Thank you** and plays the payment chime (tap the display once beforehand so the browser allows audio).

![UPI QR at checkout](docs/tutorial/pos-checkout-upi.png)

![UPI QR on the customer display](docs/tutorial/customer-display-upi.png)

![Payment recorded](docs/tutorial/pos-payment-done.png)

![Thank-you screen](docs/tutorial/customer-display-paid.png)

### The customer display

1. On the second screen open `http://<server>:3000/customer-display` (production: port 1995).
2. Click or tap once anywhere; browsers block audio until the page is touched.
3. Open it in the same browser profile as the POS, in another window. It follows the till through a local channel.
4. It shows a welcome screen when idle, the running cart and total, the UPI QR at payment, and a thank-you when paid.

![Customer display idle](docs/tutorial/customer-display.png)

> Direct printing and the cash drawer need a supported printer; the browser print dialog works with any printer or **Save as PDF**.

---

## 8. Orders

**Orders** is the control room for everything after the order exists.

![Orders](docs/tutorial/orders.png)

### Tabs and filters

- **New (to accept)** — website/API orders waiting for a decision. QR and kiosk orders skip this tab and appear in **In progress**; pending orders also show in a dashboard banner and on the KDS.
- **In progress** — open bills, whether sent to the kitchen or not.
- **History** — paid and cancelled orders.
- Filters: **All**, **Online**, **Offline**, **Takeaway**, **Delivery**, **Dine-in**.
- **Scan ticket QR or type order number, then Enter** — scan the QR on a printed KOT (or type the order number) to jump straight to that order.

### Accept or decline an online order

```mermaid
flowchart TD
  A["New (to accept)"] --> B["Open the order"]
  B --> C{"Can the kitchen make it?"}
  C -->|Yes| D["Accept order"]
  D --> E["Order accepted, tickets sent to KDS"]
  C -->|No| F["Decline"]
  F --> G["Pick a reason"]
  G --> H["Order rejected, nothing charged"]
```

1. Open **Orders → New (to accept)**.
2. Tap the order row. The detail dialog shows the guest, items, total and phone.
3. Review the items, total and delivery address/phone.
4. Press **Accept order** to send it to the kitchen, or **Decline**.
5. Declining asks for a reason: **Closing soon**, **Out of stock**, **Too busy** or **Outside delivery area**. The order stays in History with the reason and your name; nothing is charged.

Requires `orders.accept`. This gate applies to website/API orders. QR and kiosk orders are already accepted when they arrive.

### Open an order and read it

![Order detail](docs/tutorial/orders-detail.png)

The detail dialog shows status, acceptance, table or delivery details, items, taxes, payments, KOT status and the guest. It offers:

- **Edit bill** - for `OPEN` orders with no payments recorded.
- **Cancel order** — with reason and kitchen warning.
- **Refund** — for paid orders (also on the dashboard).
- Print actions for bill and KOT.

![In-progress orders](docs/tutorial/orders-in-progress.png)

### Edit a bill

![Orders history](docs/tutorial/orders-history.png)

**Edit bill** is for corrections before payment:

1. Open the order → **Edit bill**.
2. Reduce quantity with **-** or remove a line. Lines the kitchen has already started cannot be removed; the dialog explains why.
3. Move the bill to another free table if the party changed seats.
4. Fix guest name/phone.
5. Save. The bill and KOTs update server-side.

### Cancel an order

1. Open the order → **Cancel order**.
2. Pick or type a **Reason**: Guest left, Wrong order, Duplicate order, Out of stock, or free text.
3. Confirm. If the kitchen has started, the action warns and needs `orders.cancel`.

Cancellation withdraws tickets, frees the table and leaves the order in History with the reason and the staff member's name. Nothing is charged or deducted. Once any payment has been recorded, cancellation is refused; settle or refund the bill instead.

### Refunds

Refunds are issued from a paid order, not from the POS:

1. **Dashboard → Recent transactions** → refund icon on the row, or open a paid order and choose **Refund**.
2. Enter the **Amount** (up to the paid amount).
3. Choose the **Refund method**: Cash, Card, UPI, Other or Store credit.
4. Add an optional **Reason**.
5. Under **Returned to stock**, enter units that physically came back and can be reused. A refund alone does not change stock.
6. Press **Issue refund**.

Requires `refunds.process`.

### Delete historical transactions

Administrators with `orders.delete` can select finished transactions in **History** and delete them after entering the admin password. This is for cleaning test or duplicate records, not normal corrections; use refunds/cancels otherwise.

---

## 9. Tables, waitlist and QR ordering

![Tables](docs/tutorial/tables.png)

The floor plan is live. Drag tables in edit mode to match the room; positions save automatically. Use the **Floor** selector for multiple rooms and **Edit floor plan** to add or move tables.

### Table tile actions

Tap the **⋯** menu on a tile:

![Table actions](docs/tutorial/tables-actions.png)

| Action | When it appears | What it does |
| --- | --- | --- |
| **View QR code** | Always | Opens the printable guest menu QR. |
| **New order** | Available or Reserved | Jumps to POS with that table pre-selected. |
| **Add another round** | Occupied | Adds items to an existing bill. |
| **Bill this table** | Occupied | Opens the bill and payment panel without going through the menu. |
| **Merge another table's bill here** | Occupied | Moves another table's open bill onto this table. |
| **Available / Occupied / Reserved / Cleaning** | Always | Sets the table status directly. |

Changing status manually is how the floor communicates: seat a party → **Occupied**; guest leaves → **Cleaning**; reset → **Available**. The POS also sets **Occupied** automatically when an order is sent. Closing the last open bill on a table (paid, cancelled or merged away) returns it to **Available** automatically; set **Cleaning** manually if you want a bussing state before the next party.

### Table lifecycle

```mermaid
flowchart TD
  A["Available"] --> B["Party seated"]
  B --> C["Occupied: open bill"]
  C --> D["Last open bill paid, cancelled or merged"]
  D --> E["Auto: Available"]
  E --> A
  E --> F["Optional: set Cleaning"]
  F --> A
  A --> G["Reserved for a booking"]
  G --> H["Guest arrives: Arrived"]
  H --> C
  G --> I["Booking cancelled or guest absent"]
  I --> A
```

### Add tables and floors

1. **Tables → Edit floor plan**.
2. **Add table** for one, or turn on **Add multiple tables at once** for a run.
3. Enter **Number**, optional **Label**, **Capacity** and **Shape** (Square, Round or Rectangle). Bulk mode asks for a **Starting number** and **How many** (up to 100) and applies the same capacity and shape to every table.
4. Drag to position; each drop saves. Press **Done editing** to leave edit mode.

A table's QR token stays the same until you replace it. **View QR code → Generate QR code** creates the first one; **Generate new code** invalidates the old printed code. Print from the dialog and place it on the table.

### QR code dialog

![Table QR](docs/tutorial/tables-qr.png)

1. Tables → tile menu → **View QR code**.
2. If there is no code, press **Generate QR code**.
3. **Print** the card, or right-click the QR to save the image.
4. The URL is your app address plus `/order/<token>` — `http://localhost:3000/order/<token>` in this demo, or `http://<server>:1995/order/<token>` in the Docker install. Anyone with the link can see that table's menu and place an order; rotate the code if a card is lost or misused.

### Waitlist

![Waitlist](docs/tutorial/tables-waitlist.png)

1. **Tables → Waitlist → Add to waitlist**.
2. Enter **Name**, **Party size**, **Quoted wait (min)** and optional **Phone**.
3. When a table frees up, press **Seat** and choose the table. The party is removed from the list and the table becomes occupied.
4. **Remove** deletes an entry that left.

### Guest service requests

Guests can tap **Call waiter** or **Request bill** on the QR page. Pending requests appear in a bar at the top of **Tables** ("Table 5 needs a waiter", "wants the bill"). Press **Done** to resolve one. This is how the floor notices a raised hand without shouting across the room.

### Merge bills

Use **Merge another table's bill here** when a party expands across two tables. The source table is freed and all items move to the destination bill. Merging is only allowed for open bills; paid bills cannot be merged.

---

## 10. Reservations

![Reservations](docs/tutorial/reservations.png)

Reservations hold a table for a future time without occupying it yet.

### Create a booking

![New reservation](docs/tutorial/reservations-new.png)

1. **Reservations → New reservation**.
2. Fill **Customer name**, **Phone**, **Guests**, **Date & time**, optional **Table** and **Special requests**.
3. **Save reservation**. The default duration is 90 minutes.
4. The booking appears with status **Reserved**.

### Move a booking through its life

```mermaid
flowchart TD
  A["Reserved"] --> B["Confirmed: guest reconfirmed"]
  B --> C["Arrived: guest is here"]
  C --> D["Completed after service"]
  A --> F["Cancelled"]
  B --> F
  C --> F
```

Use the status menu on the row. Quick actions on a **Reserved** row confirm or cancel it. **Arrived** means the guest is at the door; seat them and start a table order from **Tables** or **POS**. **Completed** closes the booking after service. The UI offers Reserved, Confirmed, Arrived, Completed and Cancelled; the API also stores a No-show status, but this build does not expose a button for it, so use Cancelled with a note.

If **Settings → Online orders & table bookings → Auto-confirm table bookings** is on, online bookings skip the manual confirmation step.

### Reservations and tables

A reservation can be created without a table ("No table assigned yet"). This build has no edit screen for a booking, so to change its table, time or party size, cancel it and create a new one. Setting a table to **Reserved** manually does not create a booking; use the reservation screen so the guest details and times are recorded.

---

## 11. Menu, combos and stock tracking

![Menu](docs/tutorial/menu.png)

The menu is organized by **Categories** on the left and items in the middle. **All items** shows everything. Hover a category to reveal its **Edit** and **Delete** icons; a category can only be deleted when it has no items. Add categories before items.

### Add a category

1. **Menu → Add category**.
2. **Name**, and **Show on website** (on by default). Turning it off hides the category from the website, table QR and kiosk menus; it stays on the POS.
3. **Add category**. Categories appear on the POS and guest menu.

### Add a menu item

![New menu item](docs/tutorial/menu-add-item.png)

1. **Menu → Add item**.
2. **Photo** (optional): upload an image.
3. **Name** — for example `Margherita Pizza`.
4. **Category** — required.
5. **Price (incl. tax)** or **Price (excl. tax)** depending on the branch tax mode. This is the Full portion if you add a half price.
6. **Tax rate %** — for example 5.
7. **Half portion price (optional)** — when set, the POS offers **Half** and **Full** (plus **Custom** when enabled).
8. **Kitchen station** — routes the KOT to that station (for example Kitchen, Bar). Leave **None** for no routing.
9. **Vegetarian** — controls the green/red dot on the POS and guest menu.
10. Save.

![Edit menu item](docs/tutorial/menu-edit-item.png)

### Edit, delete and combos

- **Edit** opens the same form.
- **Delete** permanently removes an item. Past bills keep the name and price snapshotted on the order line, but the server normally refuses to delete an item that orders already reference — reserve deletion for mistakes and never-sold items.
- **Combo** opens the combo builder.
- This build has **no per-item hide or sold-out switch** on the Menu screen. You can hide a whole category with **Show on website**, but there is no one-tap "86" for a single item. If an item runs out during service, the kitchen rejects the ticket and the floor stops offering it. Do not delete a well-selling item just to mark it unavailable.

### Combos

1. Use the **Combo** action on an item.
2. Add component items and quantities with **+** and **-**.
3. Save. The POS sells the combo as one line; stock is deducted through each component's own tracking.

A combo must have at least one component. The combo price is the item's own price, not the sum of components.

### Stock tracking and recipes

Open an item, press **Edit**, and use the **Stock when sold** section at the bottom:

| Mode | Behavior |
| --- | --- |
| **Not tracked** | Selling it never changes stock. |
| **Recipe** | Deducts the listed ingredient quantities per sale. |
| **Stock item** | Deducts a quantity of one ingredient as-is — bottled drinks, packaged goods. |

For **Recipe**, add each ingredient and the quantity used per portion, then save. For **Stock item**, choose the ingredient and quantity per sale. Combos deduct through their component items; set tracking on each component.

Stock is deducted when the bill is paid. A shortfall never blocks a sale; it can make stock negative, which is visible in Inventory and reports.

### Menu setup flow

```mermaid
flowchart TD
  A["Add categories"] --> B["Add items: name, price, tax, station"]
  B --> C{"Combo?"}
  C -->|Yes| D["Add component items"]
  C -->|No| E["Set photo, veg flag, half price"]
  D --> F["Configure stock tracking"]
  E --> F
  F --> G{"Track stock?"}
  G -->|No| H["Not tracked"]
  G -->|Recipe| I["Add ingredients per portion"]
  G -->|Stock item| J["Choose ingredient and quantity"]
  H --> K["Available on POS and QR menu"]
  I --> K
  J --> K
```

### Availability and pricing rules

- Deleted items disappear from the POS and guest menu; historical bills keep the name and price captured at order time.
- There is no per-item hide/sold-out toggle in this build — only Delete, and only safely before the item has order history. To remove a whole category from online ordering, edit the category and turn **Show on website** off.
- Prices are server-authoritative; the POS never sends a price.
- Changing a price does not change historical bills.
- Tax mode is branch-wide: **Settings → This branch → Menu prices are Tax-inclusive / Tax-exclusive**. Tax-exclusive adds tax at checkout; tax-inclusive backs it out.

---

## 12. Kitchen Display

![Kitchen Display](docs/tutorial/kds.png)

Open **Kitchen Display** on a screen in the kitchen. It updates in real time through Socket.IO; no refresh is needed.

### Header and stations

- **Station** buttons: **All stations** or a specific station. The choice is remembered per device per branch, so the grill screen can stay on Grill while the pass sees everything.
- **Sound on / Sound off** and **Tap to turn on new-ticket sound**. Browsers block audio until the page is touched; the prompt handles that.
- The header shows live counts by column.
- **Avg. time to ready today** shows the average minutes per station and the number of tickets ready.

### Incoming online orders

Orders with acceptance **PENDING** appear above the ticket columns. This is the kitchen's chance to decline before cooking:

1. Review items and notes.
2. **Accept** to create the tickets and start the timer.
3. **Decline** for out of stock or too busy; the order is rejected, appears in History with the reason and your name, and nothing is charged.

### Ticket columns and actions

| Column | Primary action |
| --- | --- |
| **New** | **Accept** |
| **Accepted** | **Start preparing** |
| **Preparing** | **Mark ready** |
| **Ready** | **Mark served** |

![A ticket](docs/tutorial/kds-ticket.png)

Each ticket shows the order number, table or delivery label, elapsed timer, items, quantities, notes and station. Late tickets and priority tickets are highlighted.

- **Star / priority** flags a rush ticket.
- **Print** reprints the KOT.
- **Reject** (X) opens the reason list: **Out of stock**, **Item unavailable**, **Kitchen too busy**, **Equipment problem**. Rejecting before cooking is open to kitchen staff; rejecting after **Start preparing** needs `orders.cancel`.

### Ticket flow

```mermaid
flowchart TD
  A["Order sent to kitchen"] --> B["Ticket NEW"]
  B --> C{"Can we make it?"}
  C -->|No| D["Reject with reason"]
  C -->|Yes| E["Accept"]
  E --> F["Start preparing"]
  F --> G["Mark ready"]
  G --> H["Mark served"]
  D --> I["Floor sees rejection; bill unchanged"]
```

### Working a busy service

1. Keep the KDS on **All stations** at the pass; pin each prep screen to its station.
2. Accept new tickets as they arrive; the timer starts at creation, so don't leave them in New.
3. Use priority for a guest who is waiting or a delivery driver at the door.
4. Bump **Mark ready** only when the plate is on the pass; **Mark served** when it leaves.
5. If an item runs out, reject the ticket and tell the floor; the POS can then remove or substitute it with **Edit bill**.

---

## 13. Customers and loyalty

![Customers](docs/tutorial/customers.png)

The customer list is searchable by name, phone or email. Each row opens the profile; the icons edit or delete.

### Add and edit a customer

![New customer](docs/tutorial/customers-new.png)

1. **Customers → New customer**.
2. Enter **Name**, **Phone** and **Email**; save.
3. Open the profile and press **Edit** to add **Address**, **Allergies**, **Notes** and the marketing opt-out.
4. **Save changes**.

Allergies are shown in red on the customer profile. This build does not print them on the KOT or show them in the POS checkout, so check the profile and tell the kitchen directly when a guest has an allergy.

### What a profile shows

- **Loyalty points**, **Store credit** and the number of **Gift cards**.
- **Allergies** and notes.
- **Gift cards** with code, balance and status.
- **Order history** of paid orders, with totals and dates.

### Loyalty at the till

1. With `customers.manage`, attach the customer at checkout by name or phone, or create them with **New customer**.
2. Their points appear in the checkout panel; enter how many to redeem.
3. The bill recalculates; the remaining points are stored on the profile.
4. Points are earned on paid orders according to **Settings → Restaurant → Loyalty program** (`Earn 1 point per` amount and `1 point redeems for` value).

### Customer flow

```mermaid
flowchart TD
  A["Guest at the till"] --> B{"Known customer?"}
  B -->|Yes| C["Attach by name or phone"]
  B -->|No| D["New customer: name, phone, email"]
  D --> C
  C --> E["Order linked to profile"]
  E --> F["Pay: earn or redeem points, apply a gift card"]
  F --> G["Profile shows history, points and balances"]
```

### Delete a customer

**Delete** asks for confirmation and is permanent. Orders already linked keep their snapshot data; the profile link is removed. Prefer keeping the profile if there is purchase history.

---

## 14. Inventory and procurement

Inventory is a full stock system: ingredients, suppliers, purchase orders, goods receipts, quotations, invoices, payments, transfers and waste.

![Inventory](docs/tutorial/inventory.png)

The tabs are **Ingredients**, **Suppliers**, **Purchase orders**, **Procurement**, **Transfers** and **Waste**. The page header may read "Inventory" on every tab; the tab strip tells you where you are.

### 14.1 Ingredients

Ingredients are the stock units behind recipes and purchases. Each has a unit, current stock, reorder level and weighted-average cost.

**Add an ingredient**

![New ingredient](docs/tutorial/inventory-new-ingredient.png)

1. **Inventory → Ingredients → New ingredient**.
2. **Name**, **Unit** (KG, G, L, ML, PIECE, DOZEN, PACK, BOX) and **Reorder level**.
3. Save. The ingredient appears under **Low Stock** when stock falls to or below the reorder level.

**Adjust stock** (stocktake correction only)

1. **Adjust** on the ingredient row.
2. Enter **Quantity change** — positive to add, negative to remove, e.g. `-2` or `5`.
3. Enter a **Reason** such as "Physical count correction".
4. **Save adjustment**.

Do not use Adjust for deliveries; receive a purchase order so batch cost and expiry are recorded.

### 14.2 Suppliers

![Suppliers](docs/tutorial/inventory-suppliers.png)

**New supplier** records **Business name**, **Contact person**, **Phone** and **Email**. Suppliers are used by purchase orders, quotations and invoices.

![New supplier](docs/tutorial/inventory-new-supplier.png)

### 14.3 Purchase orders and goods receipts

![Purchase orders](docs/tutorial/inventory-purchase-orders.png)

A purchase order (PO) is what you send the supplier. Receiving it creates stock batches with lot numbers and optional expiry.

```mermaid
flowchart TD
  A["New purchase order"] --> B["Choose supplier and lines: ingredient, qty, cost/unit"]
  B --> C["PO DRAFT"]
  C --> D["Mark as sent"]
  D --> E["PO SENT"]
  E --> F["Receive items"]
  F --> G["Enter qty received, cost, expiry per line"]
  G --> H{"All lines received?"}
  H -->|No| I["PARTIALLY_RECEIVED"]
  H -->|Yes| J["RECEIVED: stock and weighted-average cost update"]
  I --> F
  C --> K["Cancel order"]
  E --> K
```

1. **Inventory → Purchase orders → New purchase order**.
2. Pick a **Supplier**.
3. Add each **Ingredient**, **Qty** and **Cost/unit**. Add rows as needed.
4. Create the PO. It starts as **DRAFT**.
5. Open the PO → **Mark as sent** when it goes to the supplier.
6. When goods arrive, open the PO → **Receive items**.
7. For each line enter **Qty received**, **Cost/unit** and optional **Expiry**. Cost can differ from the order.
8. Submit. Each line becomes its own batch with a lot number; stock and weighted-average cost update.

PO statuses: **DRAFT**, **SENT**, **PARTIALLY_RECEIVED**, **RECEIVED**, **CANCELLED**.

![New purchase order](docs/tutorial/inventory-new-po.png)

### 14.4 Procurement

![Procurement](docs/tutorial/inventory-procurement.png)

Procurement tracks the paperwork around buying:

| Section | Flow |
| --- | --- |
| **Purchase requests** | **New request** → items and notes → **DRAFT** → mark **PENDING_APPROVAL** → **APPROVED** or **REJECTED** |
| **Supplier quotations** | **New quotation** → supplier, items, qty, price/unit → **DRAFT** → mark **RECEIVED** → **ACCEPTED** or **REJECTED** |
| **Vendor invoices** | **New invoice** → supplier, invoice number, dates, total → status **UNPAID** → **Record payment** → **PARTIALLY_PAID** → **PAID** |

**Record a vendor payment**

1. On an unpaid or partly paid invoice press **Record payment**.
2. Enter **Amount**, **Method** and optional **Reference** (UTR/cheque number).
3. Save. The invoice shows the amount paid and remaining balance.

### 14.5 Transfers between branches

![Transfers](docs/tutorial/inventory-transfers.png)

Transfers need at least two branches.

1. **Inventory → Transfers**.
2. Choose the **Ingredient**, **Quantity**, **To branch** and optional **Note**.
3. Press **Transfer**. The source stock is reduced FIFO and the destination receives a new batch at the same cost.
4. The history below shows from/to branch, ingredient, quantity, cost and time.

The destination's ingredient must use the same unit; a unit mismatch is rejected.

### 14.6 Waste

![Waste](docs/tutorial/inventory-waste.png)

1. **Inventory → Waste → Log waste**.
2. Choose the **Ingredient**, **Quantity** and **Reason**: Expired, Spoiled, Damaged, Prep error, Over-production, Other.
3. Add notes if useful, then save. Stock is reduced FIFO at the real batch cost.
4. The page shows the total cost of waste on record; the dashboard charts it over 14 days.

![Log waste](docs/tutorial/inventory-waste-dialog.png)

Waste logging cannot take stock negative; if the quantity exceeds stock, reduce it or correct the stock first.

### 14.7 Low stock and costing

- **Low Stock** lists ingredients at or below the reorder level.
- Cost is **weighted average**, recomputed on every goods receipt.
- Recipes and stock-item mappings are set on the menu item, not here.
- Sales deduct stock but never fail because of a shortfall.

---

## 15. Delivery

![Delivery](docs/tutorial/delivery.png)

Delivery orders are created at the POS with **Delivery** selected. This screen manages what happens after they are sent.

### Active deliveries

Each card shows the order, guest, address, phone, total and delivery status.

1. Choose a driver from the **Assign driver** dropdown on the card. Only staff with the delivery role/branch appear.
2. When the driver leaves, press **Picked up**.
3. At the door, press **Delivered**.
4. If it cannot be delivered, press **Mark failed**. The card moves to Failed; this build has no separate failure-reason field, so note the reason in the order or with the driver.
5. **History** shows completed and failed deliveries.

Statuses: **Needs driver** (`UNASSIGNED`), **Driver assigned** (`ASSIGNED`), **Out for delivery** (`PICKED_UP`), **Delivered**, **Failed**.

```mermaid
flowchart TD
  A["Delivery order sent from POS"] --> B["Needs driver"]
  B --> C["Assign driver"]
  C --> D["Driver assigned"]
  D --> E["Picked up"]
  E --> F{"Delivered?"}
  F -->|Yes| G["Delivered"]
  F -->|No| H["Failed"]
```

### Zones

![Delivery zones](docs/tutorial/delivery-zones.png)

**Delivery → Zones** defines fees and serviceability:

1. Enter **Zone name**, **Fee**, **Min order**, **ETA (min)** and **Pincodes, comma separated**.
2. **Add zone**.
3. **Disable** a zone to stop new orders without deleting it; **Enable** restores it.

The POS matches the guest's pincode to a zone to apply the fee, minimum and ETA. With no zones configured, delivery orders are accepted from anywhere with no fee; once at least one zone exists, only its pincodes can order delivery.

### Delivery and payment

- **Prepaid:** take payment at checkout when the order is placed, before the driver leaves.
- **Cash on delivery:** leave the bill open; the driver or cashier settles it on return.
- Delivery orders appear in **Orders** with the **Delivery** filter and in the kitchen as normal tickets.

---

## 16. Team, attendance and payroll

### 16.1 Team

![Team](docs/tutorial/team.png)

**Team** has four tabs: **Schedule**, **Leave**, **Payroll**, **Performance**.

**Schedule**

1. Pick the week (the view shows seven days from the selected date).
2. Choose **Staff**, **date**, **start** and **end**.
3. **Add shift**. A clashing shift for the same person is rejected with “That person already has a shift at this time”.
4. Delete a shift with its remove action.

![Add shift](docs/tutorial/team-add-shift.png)

**Leave**

![Leave](docs/tutorial/team-leave.png)

Staff request leave from **Attendance**. Managers approve or reject here. Approved **unpaid** leave is deducted in payroll; annual and sick leave are tracked but not deducted.

**Payroll**

![Payroll](docs/tutorial/team-payroll.png)

1. Under **Pay rates**, set each person to **Monthly** or **Hourly** and enter the **Rate**; save the row.
2. Pick the **period** start and end.
3. Press **Run payroll**. The draft uses **clocked hours** from Attendance, approved unpaid leave, and tips collected in the period.
4. Tips are split by hours worked.
5. Review **Hours**, **Base**, **Leave -**, **Tips** and **Net** per person; the run total is below the table.
6. **Finalize** to lock the run, or **Discard draft** to recalculate.
7. Finalized payroll feeds profit & loss.

```mermaid
flowchart TD
  A["Set pay type and rate"] --> B["Pick payroll period"]
  B --> C["Run payroll"]
  C --> D["Draft: clocked hours, unpaid leave, tips by hours"]
  D --> E{"Correct?"}
  E -->|No| F["Discard draft, fix attendance or rates"]
  F --> C
  E -->|Yes| G["Finalize"]
  G --> H["Payroll cost appears in P and L"]
```

**Performance**

![Performance](docs/tutorial/team-performance.png)

Choose a date range to see orders, sales, tips, hours and sales-per-hour per staff member.

### 16.2 Attendance

![Attendance](docs/tutorial/attendance.png)

Everyone can open **Attendance**; the sidebar item has no permission gate.

1. **Clock in** at the start of the shift. The page shows "On shift since HH:MM" and the running duration.
2. **Clock out** at the end.
3. Managers with `attendance.manage` can pick a date and see everyone's clock-in/out times and durations.
4. **Request leave**: choose **Leave type** (Annual, Sick, Unpaid, Other), start date, end date and optional **Reason**, then **Request**.
5. Recent requests show status (pending, approved, rejected).

```mermaid
flowchart TD
  A["Start of shift"] --> B["Attendance: Clock in"]
  B --> C["Work"]
  C --> D["Clock out"]
  D --> E["Hours feed payroll"]
  C --> F["Request leave: type, dates, reason"]
  F --> G["Manager approves or rejects in Team → Leave"]
  G --> H["Approved unpaid leave deducted in payroll"]
```

This build has no punch-correction screen: a missed clock-out stays wrong and produces wrong payroll hours. Have staff clock in and out every shift; if a punch is missed, an administrator has to correct the record in the database.

---

## 17. Accounting

![Accounting](docs/tutorial/accounting.png)

Accounting has eight tabs: **Expenses**, **Cash close**, **Profit & loss**, **Balance sheet**, **Budgets**, **Bank**, **GST**, **Branches**. All amounts are branch-scoped and use the branch currency.

### 17.1 Expenses

1. **Accounting → Expenses**.
2. Set the date, **Category** (e.g. Rent, Supplies), **Amount** and **Note**.
3. Tick **Paid from till** if the cash left the drawer; this affects cash close.
4. **Add expense**.
5. Use the date range to review; delete a wrong entry with its delete action.

![Add expense](docs/tutorial/accounting-add-expense.png)

### 17.2 Cash close

![Cash close](docs/tutorial/accounting-cash-close.png)

1. Pick the **business date**.
2. Enter the **Opening float**.
3. Enter **Counted cash** from the drawer.
4. The page shows **Expected in till** (cash sales minus cash expenses plus float), the **variance**, and **Balanced / Over by / Short by**.
5. Add a **Note** if there is a variance.
6. **Close day**. The closing is stored in history with expected, counted and variance.

```mermaid
flowchart TD
  A["End of service"] --> B["Accounting: Cash close"]
  B --> C["Opening float and counted cash"]
  C --> D{"Variance?"}
  D -->|No| E["Balanced: Close day"]
  D -->|Yes| F["Investigate tips, cash expenses, refunds"]
  F --> G["Add note and Close day"]
  E --> H["Closed day in history"]
  G --> H
```

Cash close is a management record, not a bank deposit. It does not move money by itself.

### 17.3 Profit & loss

Choose a date range. The report shows net sales, expenses by category, payroll and the resulting profit. Finalized payroll runs are included; drafts are not.

![Profit and loss](docs/tutorial/accounting-profit-loss.png)

### 17.4 Balance sheet

A snapshot of assets (cash, inventory value), liabilities (unpaid supplier invoices, gift-card balances) and equity. It is a **management view**, not a statutory filing. Use your accountant for filed statements.

![Balance sheet](docs/tutorial/accounting-balance-sheet.png)

### 17.5 Budgets

![Budgets](docs/tutorial/accounting-budgets.png)

1. Choose the **Month**.
2. Enter an **Expense category** and a **Budget** amount.
3. Press **Set budget**. The table shows **Budget**, **Spent**, **Left** and a **Used** progress bar per category.
4. Remove a budget with its **Remove** action.

### 17.6 Bank reconciliation

![Bank](docs/tutorial/accounting-bank.png)

1. Export your bank statement as **CSV** with a date, a description, and either one `amount` column (negative for money out) or `debit` and `credit` columns.
2. **Accounting → Bank → Import bank statement (CSV)** and choose the file.
3. Money-out lines are auto-matched to expenses with the same amount within 5 days.
4. Press **Auto-match** to retry, **Match** to choose an expense manually, or **Ignore** for lines you do not track.
5. Re-importing the same file is safe; duplicates are skipped.

### 17.7 GST / tax summary

![GST](docs/tutorial/accounting-gst.png)

Pick a date range. The table shows tax by rate: **Rate**, **Taxable value**, **Tax**, **Total**. The regime (India GST, US Sales Tax, EU VAT, Spain IVA) and inclusive/exclusive mode come from **Settings → This branch**.

### 17.8 Branches

![Branches](docs/tutorial/accounting-branches.png)

Compare **Orders**, **Sales**, **Open** orders, **Low stock** and inventory value across all branches for a range. Use the branch switcher for branch-specific detail.

---

## 18. Reports, exports and scheduled emails

![Reports](docs/tutorial/reports.png)

Reports open with a date range (default: last 30 days) and a catalog on the left. Choosing a report loads its table or chart on the right.

### The report catalog

| Group | Reports |
| --- | --- |
| **Sales** | Daily sales, Sales by item, Sales by category, Popular items, Slow movers, Payment methods, Discounts & coupons |
| **Tax** | Tax (GST) summary |
| **Analytics** | Peak hours, Food cost & margins |
| **Customers** | Top customers, New vs returning customers |
| **Staff** | Staff sales |
| **Operations** | Table turnover, Reservations, Deliveries |
| **Inventory** | Waste, Inventory valuation |

### Print and export

- **Print** opens the browser print dialog with the sidebar and controls hidden; use **Save as PDF** for a PDF.
- **Excel**, **PDF** and **CSV** download the report. Those three buttons require `data.export`; **Print** does not.
- The date range is always applied to the export.

### Scheduled email

1. Set `SMTP_URL` and `SMTP_FROM` on the server and restart the backend.
2. Open a report and scroll to **Email this report on a schedule**.
3. Choose **Frequency** (Daily, Weekly, Monthly), **File format** (xlsx, pdf, csv) and **Recipients** (comma separated).
4. Press **Schedule**.
5. Existing schedules can be **Send now** (test) or **Delete**.

Timing is 06:00 branch time: daily covers yesterday, weekly goes out Monday for the previous seven days, monthly goes on the 1st for the previous month. If email is not configured, the panel says so instead of failing silently.

```mermaid
flowchart TD
  A["Reports"] --> B["Pick report and date range"]
  B --> C{"What next?"}
  C -->|View| D["Read table or chart"]
  C -->|Print| E["Browser print or Save as PDF"]
  C -->|Export| F["Excel, PDF or CSV"]
  C -->|Email| G["Frequency, format, recipients, Schedule"]
  G --> H["Sent at 06:00 branch time"]
```

### Report permissions

`reports.access` opens the module. Financial reports additionally use `reports.financial.view`; exports use `data.export`. A restaurant manager may see reports but not export them.

---

## 19. Marketing

![Marketing](docs/tutorial/marketing.png)

Three tabs: **Coupons**, **Happy hour**, **Campaigns**.

### 19.1 Coupons

1. Enter **CODE**, choose **Percent** or **Flat amount**, and a **Value**.
2. Optionally set **Min order** and **Max uses**.
3. **Create**.
4. The list shows the code, discount, minimum and usage. **Disable/Enable** pauses it; delete removes it.

Coupons are entered at checkout and validated server-side. A coupon never stacks with a happy hour — the bigger discount applies.

### 19.2 Happy hour

![Happy hour](docs/tutorial/marketing-happy-hour.png)

1. **Name** (for example "Weekday happy hour").
2. **Type** Percent or Flat amount, and **Value**.
3. **Start** and **End** times.
4. Toggle the days of the week.
5. **Create**.

Happy hours apply automatically at checkout during the window in the restaurant's timezone. Pause or delete from the list.

### 19.3 Campaigns

![Campaigns](docs/tutorial/marketing-campaigns.png)

1. **Campaign name**.
2. **Channel**: Email, SMS or WhatsApp.
3. **Audience**: **All customers**, **Loyal (5+ orders)**, **Lapsed (60+ days)** or **Birthday this month**.
4. Enter the **Message** (up to 600 characters).
5. **Save campaign**.
6. **Audience CSV** downloads the recipient list; the count is shown on the card.
7. **Send now** dispatches to the audience.

Delivery:

- **Email** goes through your SMTP server (`SMTP_URL`, `SMTP_FROM`).
- **SMS and WhatsApp** are handed to a webhook subscribed to `campaign.message` under **Settings → Webhooks**; your messaging service performs delivery.
- Customers marked "do not send marketing" are always excluded.

```mermaid
flowchart TD
  A["Create coupon or happy hour"] --> B["Guest order reaches checkout"]
  B --> C{"Happy hour active?"}
  C -->|Yes| D["Automatic discount"]
  B --> E["Cashier enters coupon code"]
  E --> F["Server validates min order and max uses"]
  D --> G["Larger discount applies"]
  F --> G
  G --> H["Bill recalculates"]
```

---

## 20. Operations

![Operations](docs/tutorial/operations.png)

Two tabs: **Equipment** and **Documents**.

### 20.1 Equipment

1. **Equipment name**, **Location**, **Service every (days)**, **Last serviced**.
2. **Add**.
3. The list shows the next service date and **Due soon** or **Overdue** badges.
4. **Serviced today** records today's service and moves the next due date.
5. **Retire** removes equipment from the active list after confirmation.

### 20.2 Documents

![Documents](docs/tutorial/operations-documents.png)

1. **New document**.
2. Enter **Title**, choose **Category** (SOP, Contract, Recipe, Purchase, Other) and type the **Body**.
3. **Save**. Select a document to read it; **Edit** or **Delete** as needed.
4. Use **Attach file** in the document view to upload PDFs, images, text, Word or Excel files (up to 5 MB each, 10 per document).

Use documents for opening/closing checklists, supplier contracts, recipe cards and training material. Documents are branch-scoped.

---

## 21. Settings

![Settings](docs/tutorial/settings.png)

Settings is split into tabs. The main page holds restaurant, branch, online ordering, notifications, kiosk and printer cards; the rest are separate pages.

Tab visibility is permission-based: **Staff** needs `users.manage`, **Roles** `roles.manage`, **Audit Log** `audit_log.view`, **Backup** `backup.manage`, **Updates** `system.update`, and **Server Sync**, **Webhooks** and **VERI*FACTU** need `settings.manage`. **API Keys** and **Security** have no extra tab permission, but the Settings route itself still requires `settings.manage`.

### 21.1 Restaurant

- **Logo** upload.
- **Name** and **Legal name**.
- **Currency** and **Timezone**.
- **Loyalty program**: **Earn 1 point per** amount and **1 point redeems for** value.
- **UPI id (for QR payments)** and **UPI payee name**.
- **Save restaurant**.

### 21.2 This branch

- **Branch name**, **Address**, **Phone**, **GST number (legacy)**.
- **Tax**: **Region / regime** (India GST, US Sales Tax, EU VAT, Spain IVA), **Menu prices are** Tax-inclusive or Tax-exclusive, **Country code**, tax ID and optional **Receipt label override**.
- **Save branch**.

Changing the tax mode affects new bills, not historical ones. The tax ID label changes with the regime (GSTIN, VAT number, etc.).

### 21.3 Online orders and notifications

- **Auto-confirm online orders** - website/API orders skip manual acceptance and go straight to the kitchen. QR and kiosk orders already skip it.
- **Auto-confirm table bookings** — online bookings skip manual confirmation.
- **Notification sound** — plays an alert for new online orders on this device.

### 21.4 Customer kiosk

See [section 22](#22-guest-ordering-kiosk-and-customer-display). Enable the kiosk, choose whether **Eat here** and **Take away** are offered, copy the URL, and open it on the kiosk device. The URL token is the only credential; rotate it if it leaks.

### 21.5 Thermal printer

- **Connection**: **USB (on this server)** or **Network (IP address)**.
- **Printer address** and **Port** for network printers (default 9100).
- Save, then use **Send test print**, **Test kitchen ticket** or **Open cash drawer** to verify.

Direct ESC/POS printing is independent of the browser print dialog. The printer must be reachable from the machine running the backend.

### 21.6 Staff

![Staff](docs/tutorial/settings-staff.png)

**Add staff** creates a login:

![Add staff](docs/tutorial/settings-add-staff.png)

1. **Name**, **Email**, **Phone**, **Password**.
2. **Role** — one of the seeded roles or a custom role.
3. **Branches** — at least one.
4. Save. The account can sign in immediately.

The **Active** switch enables or disables the account; the row actions also **Reset 2FA** and delete. Disabling is preferred to deleting so history keeps the person's name.

### 21.7 Roles

![Roles](docs/tutorial/settings-roles.png)

Seeded roles are listed with type and permission count. **Create role** makes a custom role:

![Create role](docs/tutorial/settings-create-role.png)

1. **Name** (internal key, e.g. `DISPLAY`) and **Display label** (what staff see).
2. Tick permissions in the grouped checklist.
3. Create. Edit later to change permissions; existing sessions pick up the new set on the next request.

A custom role with no permissions can sign in but sees almost nothing; this is useful for a locked-down account.

### 21.8 Audit log

![Audit log](docs/tutorial/settings-audit-log.png)

Filter by **From** and **To** dates. The table shows **Actor**, **Action**, **Entity**, **Timestamp** and a **Details** view. Use it to answer "who cancelled this order?" or "who changed the price?".

### 21.9 Backup

![Backup](docs/tutorial/settings-backup.png)

- **Download backup now** saves a `.zip` of the database and uploaded files to this computer.
- **Backup & restore** configures S3-compatible storage: **Endpoint URL**, **Bucket**, **Access key ID**, **Secret access key**, **Region**, **Backup every (days)** and **Keep last (backups)**.
- **History** lists backups with status, size and actions; restore from a backup with the restore dialog.
- Test a restore before you need it.

### 21.10 Updates

![Updates](docs/tutorial/settings-updates.png)

**Check for update** compares the running version with the latest release. If a new version exists, **Update now** asks for confirmation; the updater pulls images, restarts the backend and reloads the UI. The old version keeps running if the update fails. Back up first.

### 21.11 Server sync

![Server sync](docs/tutorial/settings-sync.png)

For multi-terminal or multi-site setups:

1. **Server address** and **Sync key** (the central server's `SYNC_API_KEY`).
2. **Test connection**.
3. Choose **First sync** mode:
   - **Use the server's data** — replaces this PC's data; pick this for a new terminal joining an existing server. It signs you out.
   - **Merge both** — keeps local data and adds it to the server.
4. Press **Connect & sync**. **Sync now** shows pulled/pushed counts; **Turn off** stops syncing.

Only one installation should be the sync server; others connect to it.

### 21.12 API keys

![API keys](docs/tutorial/settings-api-keys.png)

- **Personal API keys** authenticate you to the MCP server. Copy the key when created; it is not shown again. Point your MCP client at the displayed **MCP server URL** with `Authorization: Bearer <key>`.
- **Integration API keys** are for external systems; each has a name, location, permission set and revocation.
- Revoke a leaked key immediately. Keys are scoped; integration keys cannot exceed their granted permissions.

### 21.13 Security

![Security](docs/tutorial/settings-security.png)

- Set up **Two-factor authentication** by scanning the QR with an authenticator app and entering the 6-digit code.
- Save the **recovery codes** shown once.
- **Turn off** requires your password and a current code/recovery code.
- Use this page to re-enable 2FA after a reset.

### 21.14 Webhooks

![Webhooks](docs/tutorial/settings-webhooks.png)

1. Enter the endpoint **URL** and optional **Description**.
2. Select **Events** (for example `order.paid`, `campaign.message`).
3. Create; copy the **signing secret** immediately.
4. **Send test** verifies delivery; **Log** shows attempts; **Pause/Resume** and **Delete** manage it.

Each request carries `X-OrderRestro-Signature: t=<unix time>,v1=<hex>`, an HMAC-SHA256 of `{t}.{raw body}` with your secret. Reject stale timestamps and mismatched signatures.

### 21.15 Spain VERI*FACTU

![VERI*FACTU](docs/tutorial/settings-verifactu.png)

The tab appears only when the branch's country code is **ES**. If you operate in Spain, this page issues chained invoice records for sales:

- Enable **Issue VERI*FACTU invoices at this branch**.
- Choose **Environment** (Test/Production) and **Invoice series**.
- Upload the **Digital certificate** and password.
- Invoice records appear as sales are paid.

The screen says it explicitly: **implemented, not certified** — it has not been run against the live AEAT portal and NODEDR has not issued a compliance certificate. Do not treat it as certified fiscal software without your accountant's sign-off.

---

## 22. Guest ordering, kiosk and customer display

### 22.1 Table QR ordering

![Guest name prompt](docs/tutorial/guest-order.png)

1. Guest scans the table QR card and opens `/order/<token>`.
2. Enters **What's your name?** (placeholder `e.g. Priya`) → **Continue to menu**.
3. Browses categories, searches, taps **Add**; options/modifiers appear if configured.
4. Reviews the cart and presses **Place order**.
5. Sees **Thank you, Priya!**, the order number, and **Order more**.
6. On later visits, **Call waiter** and **Request bill** create a request for staff; the buttons change to "Waiter is coming" / "Bill requested".
7. The footer always says **Prices include tax. A staff member will confirm your order shortly.** That copy is fixed even if the branch uses tax-exclusive prices, so verify the guest total before payment.

The order goes straight to the kitchen as channel **Online**, type **QR_ORDER**, acceptance `NOT_REQUIRED`; it appears in **Orders → In progress**, not **New (to accept)**. The table is marked occupied. If the table already has an open bill, **Order more** adds the items to that bill instead of opening a new order.

```mermaid
flowchart TD
  A["Scan table QR"] --> B["Enter name"]
  B --> C["Menu: add items"]
  C --> D["Place order"]
  D --> E["Accepted automatically: Online / QR_ORDER"]
  E --> F["KDS tickets created; appears in Orders → In progress"]
  F --> G["Guest can Order more or Request bill"]
  G --> H["Staff resolve request on Tables"]
```

### 22.2 Customer kiosk

The kiosk is a self-service terminal for walk-in orders. Enable it in **Settings → Customer kiosk**:

1. **Enable kiosk**.
2. Choose **Offer "Eat here"** and/or **Offer "Take away"** (at least one must stay on).
3. Copy the **Kiosk address** or scan its QR; open it full-screen on the kiosk device.
4. **Create new address** invalidates the old URL; **Turn kiosk off** disables it. If only one service option is enabled, the kiosk skips the choice screen.

The kiosk opens in **Spanish** by default. Use the **ES** / **EN** buttons in the header to switch it to English; the labels below show the English copy with Spanish in parentheses.

- Welcome: **Order here** / **Touch the screen to start** (*Haz tu pedido aquí* / *Toca la pantalla para empezar*).
- **Where will you eat?** → **Eat here** or **Take away** (*¿Dónde vas a comer?* → *Comer aquí* or *Para llevar*).
- Menu → **Add** → **Your order** → **Review order** (*Añadir* → *Tu pedido* → *Revisar pedido*).
- **Check your order** → **Confirm order** → **Order sent!** with **Your order number** (*Revisa tu pedido* → *Confirmar pedido* → *¡Pedido enviado!* / *Tu número de pedido*).
- **Please pay at the counter. Your order is not paid yet.** The kiosk does not take payment. (*Pasa por caja para pagar. Tu pedido aún no está pagado.*)
- **New order** starts again; the idle warning **Are you still there?** clears the basket if ignored.

Kiosk orders are channel **Online**, type **KIOSK**, and also skip the order-accept gate: they appear in **Orders → In progress** and go straight to the kitchen. Payment is at the counter.

### 22.3 Customer display

`/customer-display` is a read-only second screen for the guest. It shows the cart live, the UPI QR with amount, and a thank-you after payment. Open it in a second window of the same browser on the same machine as the POS, and tap once to unlock sound. It is not a payment terminal; the cashier confirms payment.

---

## 23. Roles and permissions reference

Permissions are granular and individually toggleable. A role is just a bundle; the server checks every action, and the sidebar hides what the account cannot open.

### Seeded roles

| Role | Typical access |
| --- | --- |
| **Owner** | Everything, including settings, roles, backups, updates and destructive actions. |
| **Administrator** | Everything, same as Owner. |
| **Restaurant Manager** | Sales, reports (non-financial), orders, discounts, refunds, tables, menu, inventory, KDS, reservations, customers, attendance, delivery, staff, accounting, marketing, operations. No settings, roles, backups or updates by default. |
| **Cashier** | POS, orders (create/edit/accept), discounts, bills, customers, sales view, dashboard sales, menu view, tables view. |
| **Waiter** | POS, orders (create/edit/accept), tables view, menu view, bills. No revenue or editing menu. |
| **Kitchen Staff** | KDS, accept orders. |
| **Chef** | KDS, accept orders, menu manage/view. |
| **Bartender** | POS, accept orders, KDS, menu view, tables view. |
| **Delivery Staff** | Edit orders, delivery management. |
| **Accountant** | Sales view, dashboard, reports, financial reports, exports, accounting. |
| **Inventory Manager** | Inventory management, reports. |

Custom roles can combine any permissions. A **manage** permission implies its **view** counterpart (`menu.manage` includes `menu.view`; `tables.manage` includes `tables.view`).

### Permission groups

| Category | Keys |
| --- | --- |
| **Sales** | `sales.view`, `dashboard.sales.view` |
| **Orders** | `orders.create`, `orders.edit`, `orders.cancel`, `orders.accept`, `discounts.apply`, `refunds.process`, `orders.delete`, `bills.print` |
| **Tables** | `tables.view`, `tables.manage` |
| **Menu** | `menu.view`, `menu.manage` |
| **Inventory** | `inventory.manage` |
| **Kitchen** | `kds.manage` |
| **Reservations** | `reservations.manage` |
| **CRM** | `customers.manage` |
| **Delivery** | `delivery.manage` |
| **Staff** | `staff.manage`, `attendance.manage` |
| **Accounting** | `accounting.manage` |
| **Reports** | `reports.access`, `reports.financial.view`, `data.export` |
| **Marketing** | `marketing.manage` |
| **Operations** | `operations.manage` |
| **Admin** | `users.manage`, `settings.manage`, `audit_log.view`, `roles.manage`, `backup.manage`, `system.update` |

### How gating works

1. The sidebar reads the current user's permissions and hides items.
2. Buttons inside pages check the specific key (for example `discounts.apply` hides **Discount %**).
3. The API re-checks on every request and returns `403` if the permission is missing.
4. The audit log records sensitive actions with the actor.

**Attendance** is deliberately visible to everyone; managers see all records via `attendance.manage`.

---

## 24. Use-case playbook

A quick index of common tasks and where they start.

| I want to… | Go to | First action |
| --- | --- | --- |
| Take a dine-in order | POS | Dine-in → tap table → add items → **Send to kitchen** |
| Take a takeaway order | POS | Takeaway → add items → **Send to kitchen** → pay |
| Send food to a delivery address | POS | Delivery → address/pincode/phone → **Send to kitchen** |
| Add a second round | Tables or POS | Occupied table → **Add another round** |
| Settle a table without the menu | Tables | Occupied table → **Bill this table** |
| Split a bill | POS checkout | **Split between** (number) → part payments |
| Give a discount | POS checkout | **Discount %** (needs `discounts.apply`) |
| Redeem a coupon | POS checkout | Enter code → **Apply** |
| Redeem loyalty points | POS checkout | Attach customer → **Redeem loyalty points** |
| Pay by UPI | POS checkout | **UPI** → guest scans → **Complete payment** |
| Print a bill | POS checkout | **Print bill** or **Print bill (thermal)** |
| Reprint a KOT | POS/Orders/KDS | **KOT (browser)** / **KOT (thermal)** / print on ticket |
| Accept a pending online order | Orders or KDS | **New (to accept)** → **Accept order** |
| Decline a pending online order | Orders or KDS | **Decline** → reason |
| Cancel an open order | Orders detail | **Cancel order** → reason |
| Correct an open bill | Orders detail | **Edit bill** (before any payment) |
| Refund a paid bill | Dashboard or order | Refund icon → amount/method → **Issue refund** |
| Mark a table available | Tables | Tile menu → status |
| Add a table | Tables | **Edit floor plan** → add table |
| Print a table QR | Tables | Tile menu → **View QR code** |
| Seat a waiting party | Tables | **Waitlist** → **Seat** |
| Create a booking | Reservations | **New reservation** |
| Check a guest in | Reservations | **Arrived** |
| Add a menu item | Menu | **Add item** |
| Remove an item from the menu | Menu | Item trash icon (safe only before it has orders) |
| Build a combo | Menu | Item **Combo** → components |
| Track recipe stock | Menu | Item → **Edit** → **Stock when sold** → Recipe |
| Accept/advance a kitchen ticket | KDS | **Accept** → **Start preparing** → **Mark ready** → **Mark served** |
| Reject an item | KDS ticket | Reject → reason |
| Add a customer | Customers | **New customer** |
| See a guest's history | Customers | Open profile |
| Add an ingredient | Inventory | Ingredients → **New ingredient** |
| Order stock | Inventory | Purchase orders → **New purchase order** |
| Receive stock | Inventory | PO → **Receive items** |
| Record a supplier invoice | Inventory → Procurement | **New invoice** |
| Pay a supplier | Inventory → Procurement | **Record payment** |
| Log spoilage | Inventory → Waste | **Log waste** |
| Move stock to another branch | Inventory → Transfers | Choose destination → **Transfer** |
| Assign a delivery driver | Delivery | **Assign driver** dropdown |
| Add a shift | Team → Schedule | Staff, date, times → **Add shift** |
| Approve leave | Team → Leave | **Approve** |
| Run payroll | Team → Payroll | Set rates → **Run payroll** → **Finalize** |
| Clock in/out | Attendance | **Clock in** / **Clock out** |
| Add an expense | Accounting → Expenses | Fill form → **Add expense** |
| Close the cash drawer | Accounting → Cash close | Float, counted cash → **Close day** |
| Reconcile the bank | Accounting → Bank | Import CSV → **Auto-match** |
| Print/export a report | Reports | Pick report → **Print** / **Excel** / **PDF** / **CSV** |
| Email a report | Reports | Schedule panel → frequency, format, recipients |
| Create a coupon | Marketing → Coupons | Code, type, value → **Create** |
| Set happy hour | Marketing → Happy hour | Name, value, times, days → **Create** |
| Send a campaign | Marketing → Campaigns | Name, channel, audience, message → **Save campaign** → **Send now** |
| Track equipment service | Operations → Equipment | Add equipment; **Serviced today** |
| Write an SOP | Operations → Documents | **New document** → save → attach files |
| Add staff | Settings → Staff | **Add staff** |
| Make a custom role | Settings → Roles | **Create role** → permissions |
| Reset 2FA | Settings → Staff | Row action **Reset 2FA** |
| Review who changed what | Settings → Audit log | Date range |
| Back up now | Settings → Backup | **Download backup now** |
| Enable the kiosk | Settings → Customer kiosk | **Enable kiosk** |
| Configure a printer | Settings → Thermal printer | USB/network → test |
| Set tax mode | Settings → This branch | Menu prices are Tax-inclusive/Exclusive |
| Configure UPI | Settings → Restaurant | UPI id + payee → save |
| Connect an external system | Settings → API keys / Webhooks | Create key / endpoint |

---

## 25. Troubleshooting and backups

| Problem | Cause and fix |
| --- | --- |
| A sidebar item is missing | The account lacks the permission. Check **Settings → Roles** and the user's role/branches. |
| Login appears to succeed, then "Unauthorized" | HTTPS with `COOKIE_SECURE=false`, or HTTP with `COOKIE_SECURE=true`. Match the setting to the URL scheme. |
| No UPI QR at checkout | Add **UPI id** in **Settings → Restaurant**, save, and re-open the bill. |
| UPI QR is blank or wrong amount | The bill total is zero, or the UPI id is missing. UPI is unavailable for a zero balance. |
| Customer display is silent | Tap the display page once; browsers block audio until a gesture. |
| Customer display does not follow the till | Open it in the same browser profile and origin as the POS. |
| "Email is not set up" | Set `SMTP_URL` and `SMTP_FROM`, restart the backend. |
| Report schedule does not send | Check SMTP, the schedule's recipients, and the 06:00 branch-time run. |
| Printer prints nothing | Confirm USB on the server host or network IP/port 9100, then use the test print in Settings. |
| Kiosk says "This kiosk is not available" (*Este quiosco no está disponible*) | The token was rotated or the kiosk disabled. Copy the current address from Settings. |
| QR page says "This QR code isn't recognized" | The table token was regenerated; reprint the table card. |
| "Unit mismatch" on a transfer | The destination branch stocks that ingredient in a different unit. Align the unit. |
| Cannot remove an item from a bill | The kitchen already started it. Cancel/void through the proper flow or ask a manager. |
| Paid bill is read-only | Paid bills are never edited. Use a refund. |
| Stock went negative | A sale is never blocked by stock. Receive stock or adjust after a stocktake. |
| Payroll hours look wrong | Check clock in/out records and approved unpaid leave before finalizing. |
| Update failed | The old version keeps running. Check the server logs and retry when the release is available. |
| Disk full / app returns 500 | Free space on the server volume (logs, old backups, container images) and restart. |

### Backups

Data lives in Postgres; uploaded images and document files live on the server volume. Use **Settings → Backup** for scheduled S3-compatible backups and **Download backup now** before upgrades. Test a restore on a spare machine; an untested backup is only a hope.

### Resetting the demo

Re-running the seed is upsert-based and safe:

```bash
pnpm --filter @nodedr-restaurant/backend exec prisma db seed
```

It restores the demo menu, tables, customers and staff without deleting orders you created. For a completely clean demo, recreate the database and run migrations + seed again.

### Where the images live

All screenshots used here are in [`docs/tutorial/`](./docs/tutorial/), captured from the running demo at a 1440×900 viewport (some at higher pixel density; the guest page at phone size). The older `docs/screenshots/tutorial/` set is kept for the README and the published manual.

---

## Appendix: quick reference

**Order statuses:** `OPEN`, `BILLED`, `PAID`, `CANCELLED` · acceptance `NOT_REQUIRED` (staff, QR, kiosk), `PENDING` → `ACCEPTED` / `REJECTED` (website/API) · KOT `NEW`, `ACCEPTED`, `PREPARING`, `READY`, `SERVED`, `CANCELLED`, `REJECTED`

**Table statuses:** `AVAILABLE`, `OCCUPIED`, `RESERVED`, `CLEANING`

**Reservation statuses:** UI `RESERVED`, `CONFIRMED`, `ARRIVED`, `COMPLETED`, `CANCELLED` · API also accepts `NO_SHOW`

**Delivery statuses:** `UNASSIGNED`, `ASSIGNED`, `PICKED_UP`, `DELIVERED`, `FAILED`

**Purchase order statuses:** `DRAFT`, `SENT`, `PARTIALLY_RECEIVED`, `RECEIVED`, `CANCELLED`

**Payment methods:** POS buttons are **Cash**, **Card**, **UPI** and **Other** (`CASH`, `CARD`, `UPI`, `OTHER`). The data model also defines `WALLET`, `BANK_TRANSFER`, `GIFT_CARD` and `STORE_CREDIT`; refunds can use `STORE_CREDIT`.

**Key routes:** `/pos`, `/orders`, `/tables`, `/reservations`, `/menu`, `/kds`, `/customers`, `/inventory`, `/delivery`, `/team`, `/accounting`, `/reports`, `/marketing`, `/operations`, `/attendance`, `/settings`

**Docs:** Swagger `http://localhost:4001/api/docs` · integration guide [`docs/integrations-api.md`](./docs/integrations-api.md) · API page `/docs/api`

---

<sub>OrderRestro is developed by [NodeDR Infotech Private Limited](https://www.nodedr.com/) · [nodedr.com](https://www.nodedr.com/) · AGPL-3.0</sub>

