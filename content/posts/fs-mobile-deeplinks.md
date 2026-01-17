+++
title= "Deep‑linking the Dynamics 365 Field Service Mobile App (2026) — Create, Prefill, and Target Specific Forms"
date= 2026-01-16T10:00:00+05:30
lastmod= 2026-01-16T10:00:00+05:30
description= "End-to-end guide to ms-apps-fs deep links for Field Service Mobile: open create forms, prefill fields via extraqs, target a specific formId, and make links clickable in Outlook."
tags= ["Dynamics 365", "Field Service", "Power Apps", "Model-driven apps", "Deep links", "Mobile"]
categories= ["Dynamics 365", "Field Service"]
series= ["FS Mobile How-Tos"]
author= "Manishkumar C. Vishwakarma"
draft= false
cover=
  image= "images/new-corrective-actions.png"
  alt= "Prefilled “New Corrective Actions” form opened via deep link in Field Service Mobile"
  caption= "New Corrective Actions form opened from a deep link with defaults"
+++

> **TL;DR**  
> - Use the **Field Service Mobile handler**: `ms-apps-fs://<org-url>_<app-id>?…` to open model-driven forms directly in the mobile app.  
> - To **create** a record, **leave `id=` blank**.  
> - To **prefill fields**, pack defaults into a single `extraqs` value; inside `extraqs` **separate pairs with `%26`** (encoded `&`) and **URL‑encode** values.  
> - To **force a specific form**, add `formid=<form-guid>` inside `extraqs`.  
> - Outlook Mobile won’t auto-link custom schemes in plain text—send as **HTML** with an `<a>` tag or add an **HTTPS fallback**.  
>
> **Docs:**  
> - Field Service deep links — handler and params:  
>   https://learn.microsoft.com/en-us/dynamics365/guidance/resources/field-service-mobile-use-deep-links  
> - Defaulting fields with `extraqs` — encoding, booleans, choices, lookups, dates:  
>   https://learn.microsoft.com/en-us/power-apps/developer/model-driven-apps/set-field-values-using-parameters-passed-form

---

## Why deep links?

Deep links remove extra taps by launching **Field Service Mobile** directly to the form a technician needs—**including “Create”**—with **key fields prefilled**. This speeds up updates, improves data quality, and makes the experience feel native.

---

## What Microsoft supports (recap)

- **Handler & base** for Field Service Mobile (Power Platform version):  
  `ms-apps-fs://<org-url>_<app-id>?tenantId=<tenant-id>&isShortcut=true&appType=AppModule&openApp=true&restartApp=true&forceOfflineDataSync=true`  
  *(See: Microsoft Learn — “Use deep links to the Field Service mobile app”)*  
  https://learn.microsoft.com/en-us/dynamics365/guidance/resources/field-service-mobile-use-deep-links
- **Opening forms**: add `pagetype=entityrecord&etn=<entity>` and either `id=<GUID>` (open existing) **or** leave `id=` blank (open **Create**).  
- **Default values**: pass them in **one** URL‑encoded `extraqs` parameter.  
  *(See: “Set column values using parameters passed to a form” for encoding and datatype rules.)*  
  https://learn.microsoft.com/en-us/power-apps/developer/model-driven-apps/set-field-values-using-parameters-passed-form

---

## The generic deep‑link template

```text
ms-apps-fs://<org-url>_<app-id>?tenantId=<tenant-id>&isShortcut=true&appType=AppModule&openApp=true&restartApp=true&forceOfflineDataSync=true&pagetype=entityrecord&etn=<entity-logical-name>&id=<record-id>&extraqs=<url-encoded-field1=value1%26field2=value2%26field3=value3>
```

**Parameters (short):**
- `<org-url>`: Dataverse host **without** `https://` (e.g., `org2605dc9c.crm8.dynamics.com`)
- `<app-id>`: App Module ID (GUID)
- `<tenant-id>`: Azure AD tenant
- `pagetype=entityrecord`: open a model-driven **form**
- `etn=<entity-logical-name>`: target table (e.g., `crd81_correctiveactions`)
- `id=<record-id>`: record GUID; **leave blank** for **Create**
- `extraqs=<…>`: one URL‑encoded string; **inside**, join pairs with **`%26`**

**Docs:**  
- FS deep links: https://learn.microsoft.com/en-us/dynamics365/guidance/resources/field-service-mobile-use-deep-links  
- `extraqs` details: https://learn.microsoft.com/en-us/power-apps/developer/model-driven-apps/set-field-values-using-parameters-passed-form

---

## Create vs. Open existing

- **Open existing** → `id=<GUID>`  
- **Create new** → `id=` (left blank)

*(FS deep-link doc explains the `id` behavior.)*

---

## Prefilling fields with `extraqs`

Supported types (must be on the form):
- **String** — `field=value`
- **Two Options** — `true/false` or `1/0`
- **Choice** — **numeric** option value
- **Multi-select Choice** — comma-separated numeric values (e.g., `0,2,3`)
- **Lookup** — for simple lookup:  
  `field=<GUID>%26fieldname=<Display%20Name>`  
  For **Customer/Owner** lookup, also add `fieldtype=account|contact|systemuser|team`
- **Date/Time** — `YYYY-MM-DD` or ISO `YYYY-MM-DDThh:mm:ssZ` (URL‑encode `:`)

> **Encoding rules:**  
> - `extraqs` is **one** parameter → **URL‑encode** it.  
> - Inside `extraqs`, separate **field=value** pairs with **`%26`**.  
> - If a **value** contains `&` or `=`, **double‑encode that value** before placing it in `extraqs`.  
> *(See: “Set column values using parameters passed to a form”)*

---

## Target a **specific form** (`formid`)

To force a particular form instance, add **`formid=<form-guid>`** **inside** `extraqs`, alongside other defaults.  
*(Background: opening forms with a specific `formId` is a standard model‑driven capability.)*  
- General open‑form guidance: https://learn.microsoft.com/en-us/power-apps/developer/model-driven-apps/open-forms-views-dialogs-reports-url  
- Client API showing `formId`: https://learn.microsoft.com/en-us/power-apps/developer/model-driven-apps/clientapi/reference/xrm-navigation/openform

**Your form id:** `5156d9d8-59ce-4d70-adbe-5ec33ec2f496`

---

## End‑to‑end example (Create + defaults + specific form)

The following opens the **Create** form for `crd81_correctiveactions`, pre-populates several fields, and **targets the specific form** above.

```text
ms-apps-fs://org2605dc9c.crm8.dynamics.com_49b36e09-1aba-f011-bbd3-6045bda5eb64?tenantId=2a83e6ad-4daa-4257-9c9a-9897be5e5aa9&isShortcut=true&appType=AppModule&openApp=true&restartApp=true&forceOfflineDataSync=true&pagetype=entityrecord&etn=crd81_correctiveactions&id=&extraqs=formid=5156d9d8-59ce-4d70-adbe-5ec33ec2f496%26crd81_correctiveactionname=Leak%20Fix%20%E2%80%93%20Pump%20A%26crd81_correctiveactiondescription=Reported%20by%20Ops%3B%20priority%20high%26crd81_correctiveactionid=0002%26crd81_implanted=true%26crd81_priority=0%26crd81_contact=12873526-4fbd-f011-bbd3-7c1e523c67ed%26crd81_contactname=Dino%20Shah%26crd81_tag=0,2%26crd81_placedon=2026-01-16
```

> **What users see**  
> A prefilled **New Corrective Actions** form (your targeted form) ready to save.

<img width="1869" height="324" alt="image" src="https://github.com/user-attachments/assets/20f389a5-5b91-45b1-9ecf-80297863fa88" />
<img width="770" height="461" alt="image" src="https://github.com/user-attachments/assets/37d8a6db-6f5f-4771-9103-f1738446f838" />

**Docs:**  
- FS deep links: https://learn.microsoft.com/en-us/dynamics365/guidance/resources/field-service-mobile-use-deep-links  
- `extraqs` rules & examples: https://learn.microsoft.com/en-us/power-apps/developer/model-driven-apps/set-field-values-using-parameters-passed-form  
- Form targeting background:  
  https://learn.microsoft.com/en-us/power-apps/developer/model-driven-apps/open-forms-views-dialogs-reports-url •  
  https://learn.microsoft.com/en-us/power-apps/developer/model-driven-apps/clientapi/reference/xrm-navigation/openform

---

## Outlook Mobile: making the deep link tappable

- Outlook Mobile **does not auto‑link** custom schemes in **plain text**.  
- Send as **HTML** and wrap the link with an anchor:

```html
<a href="ms-apps-fs://<org>_<appId>?tenantId=<tenantId>&amp;isShortcut=true&amp;appType=AppModule&amp;openApp=true&amp;restartApp=true&amp;forceOfflineDataSync=true&amp;pagetype=entityrecord&amp;etn=<entity>&amp;id=&amp;extraqs=formid=<FORM_GUID>%26field1=value1">
  Open in Field Service Mobile
</a>
```

- If your org policies (Intune/MAM) block custom URI schemes, include an **HTTPS fallback** that’s always clickable:

```text
https://<org>.crm.dynamics.com/main.aspx?appid=<appId>&pagetype=entityrecord&etn=<entity>
```

**Docs:**  
- Deep-link overview for mobile: https://learn.microsoft.com/en-us/power-apps/mobile/mobile-deep-links  
- Web URL pattern: https://learn.microsoft.com/en-us/power-apps/developer/model-driven-apps/open-forms-views-dialogs-reports-url

---

## Troubleshooting

1. **App doesn’t open**  
   - Confirm handler `ms-apps-fs://<org>_<appId>?tenantId=…&…` and required flags.  
   - Ensure the app module is **shared** and user roles are assigned.  
   - Doc: https://learn.microsoft.com/en-us/dynamics365/field-service/mobile/set-up-field-service-mobile

2. **Link not clickable in email**  
   - Must be **HTML** message; Classic Outlook (“Insert as Text”) or **Power Automate** (Is HTML = Yes).  
   - Add **HTTPS fallback**.

3. **A field (or form) doesn’t default**  
   - Check the **logical name**, ensure the field is on the **form** and published.  
   - **Lookups**: set `field=<GUID>` and `fieldname=<Display%20Name>`; for Customer/Owner add `fieldtype=`.  
   - **Choices**: pass **numeric** values; **Date/Time**: try `YYYY‑MM‑DD`.  
   - Verify `formid` belongs to the table and is available in the app.

4. **Values contain `&` or `=`**  
   - **Double‑encode** that *value* before placing inside `extraqs`.

---

## Copy‑ready snippets

**Minimal Create (no defaults)**
```text
ms-apps-fs://<org-url>_<app-id>?tenantId=<tenant-id>&isShortcut=true&appType=AppModule&openApp=true&restartApp=true&forceOfflineDataSync=true&pagetype=entityrecord&etn=<entity-logical-name>&id=
```

**Create + specific form**
```text
...&extraqs=formid=<FORM_GUID>
```

**Create + specific form + defaults (pattern)**
```text
...&extraqs=formid=<FORM_GUID>%26field1=value1%26field2=value2
```

**HTML anchor (Outlook‑friendly)**
```html
<a href="ms-apps-fs://<org>_<appId>?tenantId=<tenantId>&amp;isShortcut=true&amp;appType=AppModule&amp;openApp=true&amp;restartApp=true&amp;forceOfflineDataSync=true&amp;pagetype=entityrecord&amp;etn=<entity>&amp;id=&amp;extraqs=formid=<FORM_GUID>%26field1=value1">
  Open in Field Service Mobile
</a>
```

---

## References

- **Use deep links to the Field Service mobile app** (official handler & parameters)  
  https://learn.microsoft.com/en-us/dynamics365/guidance/resources/field-service-mobile-use-deep-links
- **Set column values using parameters passed to a form** (`extraqs`, encoding, lookups, dates)  
  https://learn.microsoft.com/en-us/power-apps/developer/model-driven-apps/set-field-values-using-parameters-passed-form
- **Open apps, forms, views, dialogs, and reports with a URL** (web pattern & security note)  
  https://learn.microsoft.com/en-us/power-apps/developer/model-driven-apps/open-forms-views-dialogs-reports-url
- **openForm (Client API)** (`formId` option when opening forms programmatically)  
  https://learn.microsoft.com/en-us/power-apps/developer/model-driven-apps/clientapi/reference/xrm-navigation/openform
- **Set up the mobile app** (sharing & permissions for Field Service Mobile)  
  https://learn.microsoft.com/en-us/dynamics365/field-service/mobile/set-up-field-service-mobile
