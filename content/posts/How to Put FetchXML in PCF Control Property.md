+++
title= 'How to Put FetchXML in PCF Control Property with Static and Dynamic Values'
date=2025-05-18T17:45:21+05:30
draft=false
tags=['dynamics 365', 'pcf', 'fetchxml']
categories=['technology', 'how-to']
description='Learn how to embed FetchXML in PCF control properties, covering both static and dynamic values for enhanced data retrieval.'
author = 'Manishkumar Vishwakarma'
+++

# How to Put FetchXML in PCF Control Property with Static and Dynamic Values

## Introduction

PowerApps Component Framework (PCF) allows developers to create custom controls for model-driven and canvas apps. One common requirement when working with PCF controls is to use FetchXML queries to retrieve data from Dataverse. This blog will guide you through the process of embedding FetchXML in PCF control properties, covering both static and dynamic values.

---

## What is FetchXML?

FetchXML is a proprietary query language used in Microsoft Dataverse to retrieve data. It is XML-based and supports complex queries, including joins, filters, and aggregations. FetchXML is widely used in model-driven apps, workflows, and custom controls.

---

## Setting Up a PCF Control

Before diving into FetchXML, ensure you have a basic PCF control set up. If you're new to PCF, follow these steps:

1. Install the [Power Platform CLI](https://learn.microsoft.com/en-us/power-apps/developer/component-framework/get-powerapps-cli).
2. Create a new PCF project:
    ```bash
    pac pcf init --namespace SampleNamespace --name FetchXMLControl --template field
    ```
3. Build and test the control:
    ```bash
    npm install
    npm start
    ```

---

## Adding FetchXML to PCF Control Properties

### 1. **Using Static Values**

Static FetchXML queries are hardcoded into the PCF control. This is useful for scenarios where the query does not change dynamically.

#### Example FetchXML Query
```xml
<fetch top="10">
  <entity name="account">
     <attribute name="name" />
     <attribute name="accountid" />
     <order attribute="name" descending="false" />
  </entity>
</fetch>
```

#### Steps to Embed Static FetchXML
1. Add a property to the `ControlManifest.Input.xml` file:
    ```xml
    <property name="fetchXML" type="SingleLine.Text" required="true" />
    ```
2. In your `index.ts` file, retrieve the property value:
    ```typescript
    const fetchXML = this.context.parameters.fetchXML.raw || "";
    ```
3. Use the FetchXML in your control logic:
    ```typescript
    const query = fetchXML;
    this.context.webAPI.retrieveMultipleRecords("account", `?fetchXml=${encodeURIComponent(query)}`)
      .then((response) => {
         console.log(response.entities);
      });
    ```

---

### 2. **Using Dynamic Values**

Dynamic FetchXML queries are generated at runtime based on user input or other parameters.

#### Example: Dynamic Query Generation
```typescript
const dynamicFetchXML = `
<fetch top="10">
  <entity name="contact">
     <attribute name="fullname" />
     <attribute name="contactid" />
     <filter>
        <condition attribute="lastname" operator="eq" value="${userInput}" />
     </filter>
  </entity>
</fetch>`;
```

#### Steps to Use Dynamic FetchXML
1. Accept user input or parameters in your control.
2. Construct the FetchXML string dynamically:
    ```typescript
    const userInput = this.context.parameters.searchTerm.raw || "";
    const dynamicFetchXML = `
    <fetch top="10">
      <entity name="contact">
         <attribute name="fullname" />
         <filter>
            <condition attribute="lastname" operator="eq" value="${userInput}" />
         </filter>
      </entity>
    </fetch>`;
    ```
3. Execute the query:
    ```typescript
    this.context.webAPI.retrieveMultipleRecords("contact", `?fetchXml=${encodeURIComponent(dynamicFetchXML)}`)
      .then((response) => {
         console.log(response.entities);
      });
    ```

---

## Best Practices

1. **Validate FetchXML**: Always validate your FetchXML queries to avoid runtime errors.
2. **Error Handling**: Implement error handling for API calls to manage scenarios like network issues or invalid queries.
3. **Security**: Avoid exposing sensitive data in FetchXML queries.
4. **Performance**: Use filters and limits (`top` attribute) to optimize query performance.

---

## Conclusion

Embedding FetchXML in PCF control properties allows you to create powerful, data-driven custom controls. Whether you're using static or dynamic values, the flexibility of FetchXML combined with PCF opens up endless possibilities for enhancing your applications. Start experimenting with FetchXML in your PCF projects today!

---

Happy coding!