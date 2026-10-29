# Shipment Management Solution


## 1. Overview

The End-to-End Shipment Management Solution is a multi-application workflow that manages the complete shipment lifecycle across three connected applications: Shipment, Warehouse, and Carrier.

Together, these applications support order creation, inventory validation, shortage handling, warehouse preparation, carrier assignment, delivery execution, final delivery confirmation, AI-assisted decision support, email notifications, and GenAI-generated document creation.

### Demo Link

- Interactive Demo - [Demo](https://demo-now.techzone.ibm.com/psl/01s087v)
- ppt - [Shipment Management Intro](https://github.ibm.com/dba/IRT-Solution-Scenarios/blob/main/ShipmentManagement/Documents/Client_Intro_Shipment_Management.pptx)

## 2. Solution Architecture Overview

### 2.1. Shipment Application
The Shipment Application is the starting point of the process. It is responsible for creating the shipment order, validating inventory, triggering procurement if stock is insufficient, and preparing shipment details for warehouse processing.

### 2.2. Warehouse Application

The Warehouse Application is the middle operational stage. It is responsible for loading shipment details, confirming warehouse readiness, applying warehouse management actions, using AI planning tools, and preparing the order for carrier assignment.

### 2.3. Carrier Application

The Carrier Application is the final execution stage. It is responsible for loading finalized shipment details, updating delivery execution status, and completing the shipment lifecycle.


### 2.4. End-to-End Integrated Process Flow

### Stage 1 – Shipment Application

The user creates the shipment request, validates inventory, handles shortage/procurement if needed, and prepares shipment details.

### Stage 2 – Warehouse Application

The warehouse team loads the shipment request, confirms readiness, performs warehouse planning, uses AI-assisted tools, and prepares carrier handoff data.

### Stage 3 – Carrier Application

The carrier team loads the finalized order, updates delivery execution details, and completes the final delivery lifecycle.

---

## 3. CP4BA Components used
### 3.1. BAA
- UI built for:
    - Shipment Creation
    - Warehouse Inventory Confirmation
    - Carrier Delivery Update
- Dynamic forms with:
    - Required field validation
    - Document upload UI integrated with FileNet
- Embedded navigation page 
- Other Available Options
    - Coach Views (BAW UI)
- Rationale for Choosing BAA
    - Fast low code UI development for business users
    - Seamless integration with BAW and FileNet
    - Supports BAW services and processes and event actions

### 3.2. BAW
- Workflow orchestration for:
    - ShipmentProcess
    - WarehouseProcess
    - CarrierProcess 
    - CarrierDeliveryUpdateProcess
- Integration services for:
    - FileNet document creation
    - Updating Shipment folder data
- Business logic + conditional routing
- Escalations:
    - Shipment → inventory shortfall
    - Warehouse → manager escalation
    - Tracking → missing tracking ID
    - Carrier → delivery failed
- Rationale for Choosing BAW
    - Best for making desired workflows
    - Provides process visibility, KPIs for BAI Dashboard
    - Easy integration with FileNet and ADS
    - Native escalation and exception handling
    - Stable orchestrator for shipment → warehouse → carrier flow

### 3.3. ADS
- ADS model for:
    - Transport mode decision
    - Delivery date estimation
- Rationale for Choosing ADS
    - Easy integration with BAW and BAA
    - Supports large data volumes and queries
    - Centralized, reusable business logic
    - Easy for business teams to maintain rules
    - Versioning + testing of decision tables

### 3.3. FNCM
- Document storage
        - PO Receipt
        - Shipping Label
        - Proof of Delivery (POD)
- Rationale for Choosing FNCM
    - Reliable document management
    - Centralized storage for shipment records
    - Easy integration with BAW and BAA

### 3.4. BAI
- Dashboard for:
    - Shipment status overview
    - Warehouse inventory levels
    - Carrier delivery performance
- KPIs:
    - Shipment on-time delivery rate
    - Inventory turnover
    - Carrier delivery success rate
- Rationale for Choosing BAI
    - Centralized visibility for business users
    - Real-time KPI monitoring
    - Easy to integrate with BAW workflows
    - Supports decision-making and reporting

---

## 4. AI Architecture Across the Solution

### 4.1. Agentic AI
### 4.1.1 Inventory AI Agent
- The Inventory AI Agent is connected to a **live inventory data source (ex. Excel dataset)**  
- It performs:
  - Natural language understanding (product and quantity input)
  - Data retrieval from the inventory dataset
  - Shortage calculation and reasoning

#### Chatbot Functionality
- The same AI agent is exposed as a **chatbot for inventory queries**
- Supports questions such as:
  - "What is the current inventory for PS5?"
  - "Do we have enough stock for this order?"
- Enables **self-service access** to inventory insights without navigating workflow screens  

#### Dynamic Data Behavior
- The AI agent always uses the **latest available inventory data**
- Any updates to the data source (e.g., adding +100 units of a product) are immediately reflected in AI responses  

#### Value Compared to Script-Based Logic
- Not limited to predefined inputs and hardcoded rules  
- Supports **conversational interaction and flexible querying**   
- Decouples data, logic, and process layers  

> This enables real-time, adaptive decision support where the system continuously reflects current business data and allows users to interact both through workflow and conversation.

### 4.1.2 Sales Forecast AI Agent
- The Sales Forecast AI Agent is connected to a **sales history data source covering the past years by product**
- It performs:
  - Natural language understanding for product, month, and year inputs
  - Retrieval of relevant historical sales data
  - Forecast calculation using predefined forecasting logic
  - Business-friendly response generation for forecast results

#### Chatbot Functionality
- The same AI agent is exposed as a **chatbot for sales queries**
- Supports questions such as:
  - "What is the sales forecast for PS5 in June 2026?"
  - "How many units of PS5 were sold in May 2025?"
  - "Explain the calculation behind this forecast"
  - "Use another forecasting method"

#### Dynamic Data Behavior
- The AI agent uses the **latest available sales dataset**
- Any updates to the sales data source are reflected in future responses without changing application logic

#### Value Compared to Script-Based Logic
- Not limited to fixed forecast inputs and hardcoded outputs  
- Supports **conversational forecasting, historical lookups, and explanation requests**  
- Provides **transparent, business-readable responses** instead of only numeric output  
- Decouples data, forecasting logic, and process flow  

> This enables flexible, explainable, and data-driven sales forecasting where users can interact through both workflow and conversation.

### 4.1.3 Carrier Confidence AI (Predictive Model + Weather AI Agent)
This AI capability introduces a carrier selection support layer within the Warehouse Application by combining **predictive scoring** with **dynamic weather-based reasoning**.

- The predictive component is implemented in **ADS** using a **PMML model**
- It evaluates carrier confidence based on predefined scoring logic and historical carrier performance data
- The output helps warehouse staff choose the **best carrier option** before final handoff

#### Predictive Model Functionality
- Calculates a **carrier confidence / delivery risk score**
- Uses structured shipment and carrier parameters such as:
  - Carrier selection
  - Shipment value
  - Special handling requirement
  - Tracking availability
  - Delivery distance
- Returns:
  - Risk score
  - Risk level
  - Recommended action
  - Top scoring reasons

#### Weather AI Functionality
- Weather is included as a **dynamic parameter** in the final confidence assessment
- The Weather AI Agent evaluates expected weather conditions across the planned delivery dates for the delivery location
- It contributes an additional confidence signal such as **High** or **Low** based on forecasted delivery conditions

#### Explainability
- The solution shows the **top reasons** behind the score so users can understand why a carrier is rated as low or high risk
- Weather analysis is also displayed separately to explain the weather contribution to the final recommendation

> This enables explainable, data-driven carrier selection where historical performance scoring is enhanced by real-time weather context.

### 4.2. GenAI

GenAI is used for business document generation and AI-supported content creation across the solution.

- PO Receipt
- Shipping Label
- Proof of Delivery (POD)

All generated business documents in this solution are created with the help of GenAI.

---

## 5. Application Architecture

### 5.1. Shipment Application


The Shipment Application initiates the shipment workflow. It captures order details, validates inventory, triggers procurement when needed, and prepares shipment details for the warehouse stage.

### 5.1.1. Main Functional Flow

#### Page 1 – Landing Page

- Entry point into the shipment workflow
- User selects the process/demo path
- User clicks Start Demo

#### Page 2 – Create Shipment Order

- Captures Order Number, Product Name, and Order Count
- User clicks Check Inventory to start inventory validation

    #### Inventory Validation Step

    - Compares requested quantity with available stock
    - Determines whether shipment can proceed
     - Triggers shortage/procurement flow if stock is insufficient

    #### Procurement Modal / Warehouse Procurement Request

    - Triggered when shortage exists
    - Shows Product Name, Shortage Quantity to Procure, Price per Item, Supervisor Name, Date, and optional Inventory AI explanation

#### Page 3 – Build Shipment

- Captures final shipment information including sender, recipient, route, pricing, and notes
- Prepares shipment for warehouse processing

### 5.1.2 AI Features

#### Inventory Control AI Response

**AI Type:** Agentic AI

**Purpose:** Explain whether inventory is sufficient, explain shortage conditions, and support proceed-versus-procure decision making.

**Implementation note:** The application combines Agentic AI explanation with structured workflow variables such as `isInventoryShort` and shortage quantities to keep the UI deterministic and actionable.

### 5.1.3. Email Triggers

#### Low Inventory Alert

- Triggered when inventory shortage is detected. Sent to the Warehouse Team. Includes the Agentic AI inventory shortage explanation.

#### Inventory Shortage Procurement Email

* Triggered when shortage procurement is submitted. Sent to the Warehouse Team. Includes shortage order details and a GenAI-generated PO Receipt.

#### Order Cancelled Email

* Triggered when Order is ready for shipment = No. Sent to the customer/order owner and includes the cancellation message.

#### Delivery Ready / Delivery Request Email

- Triggered when shipment creation completes successfully. Sent to the Warehouse Team. Includes shipment summary details and a GenAI-generated PO Receipt.

---

## 5.2. Warehouse Application



The Warehouse Application operationalizes the order after shipment creation. It loads the shipment, confirms warehouse readiness, applies warehouse-side actions, uses AI planning tools, and prepares the order for carrier execution.

### 5.2.1 Main Functional Flow

#### Page 1 – Fetch Order Information

- Loads shipment details from the Shipment Application using the Order Number
- User clicks Load Order Details

#### Page 2 – Warehouse Delivery Processing

- Displays shipment approval note automatically
- Shows Delivery Information
- Collects Warehouse Management fields such as Assigned Staff, Inventory Confirmed, Pickup Window, and Special Handling Required
- Includes AI Planning Tools section

#### Page 3 – Assign Carrier and Finalize Delivery

- Displays Delivery Information and Carrier Information
- Allows carrier-specific data entry
- Includes AI Carrier Confidence section before final warehouse submission

### 5.2.3 AI Features

#### Sales Forecast AI Popup

**AI Type:** Agentic AI

**Purpose:** Forecast expected product demand for a selected month/year and support warehouse planning and replenishment decisions.

#### Carrier Delivery Confidence Popup

**AI Type:** Predictive Model + Weather AI Agent

**Purpose:** Evaluate whether the selected carrier is a reliable delivery option before final handoff to the Carrier Application.

**How it works:** A predictive model calculates carrier delivery confidence using structured shipment/carrier factors, while the Weather AI Agent contributes weather reasoning to the final confidence assessment.

### 5.2.4 Email Triggers

#### Carrier Pickup Request Email

Triggered after warehouse processing is completed successfully. Sent to the carrier. Includes shipment handoff details and a GenAI-generated Shipping Label.

#### Order Ready to Ship Email

Triggered after warehouse processing is completed successfully. Sent to the customer. Includes order readiness details and tracking information.

Possible warehouse exception/escalation points include inventory confirmed = No, tracking-related issues, and other readiness-related exceptions.

---

## 5.3. Carrier Application

The Carrier Application is the final execution stage. It loads finalized shipment details, updates delivery execution status, and completes the shipment lifecycle.

### 5.3.1 Main Functional Flow

#### Page 1 – Fetch Order Details

- Loads finalized order details using the Order Number
- User clicks Load Order Details

#### Page 2 – Update Carrier Delivery Status

- Displays Shipment Information such as Order Number, Tracking Number, and Carrier Name
- Captures Delivery Timestamp, Delivery Status, and Delivery Notes
- User clicks Update Delivery

#### Page 3 – Delivery Completed

- Shows final shipment status dynamically
- User clicks Finish to close the flow

### 5.3.2 Email Triggers

#### Delivery Status Email

Triggered when delivery status is updated. Sent to the customer and includes the latest delivery status.

#### Delivered Email

Triggered when delivery status = Delivered. Sent to the customer and includes final delivery confirmation plus a GenAI-generated POD Document.

---

## 5.4. How the Three Applications Work Together

The three applications form one connected operational flow:

1. Shipment Application creates the order, validates inventory using Agentic AI, handles shortage/procurement when required, and prepares shipment data.
2. Warehouse Application retrieves the shipment, completes warehouse actions, uses Sales Forecast Agentic AI and Carrier Delivery Confidence (Predictive Model + Weather AI Agent), and finalizes carrier handoff.
3. Carrier Application loads the finalized order, updates delivery progress, records final status, and completes delivery closure.

---

## 5.5. Email and Document Flow Summary

### Shipment Application Emails

- Low Inventory Alert → Warehouse Team → includes Agentic AI inventory response
- Inventory Shortage Procurement Email → Warehouse Team → includes GenAI-generated PO Receipt
- Order Cancelled Email → Customer / order owner
- Delivery Ready / Delivery Request Email → Warehouse Team → includes GenAI-generated PO Receipt

### Warehouse Application Emails

- Carrier Pickup Request Email → Carrier → includes GenAI-generated Shipping Label
- Order Ready to Ship Email → Customer

### Carrier Application Emails

- Delivery Status Email → Customer
- Delivered Email → Customer → includes GenAI-generated POD Document

---

## 5.6. GenAI-Generated Documents

All generated business documents in the solution are created with the help of GenAI.

- PO Receipt – used in procurement and shipment-completion communication
- Shipping Label – used in carrier pickup communication
- Proof of Delivery (POD) – used in final delivered customer communication

---

## 5.7. Stakeholder Communication Summary

### Warehouse Team receives

- Low inventory alert
- Procurement request email
- Shipment/delivery request email

### Carrier receives

- Shipment pickup request
- GenAI-generated shipping label

### Customer receives

- Cancellation email, if applicable
- Order ready to ship email
- Delivery status updates
- Final delivered confirmation with POD

---

## 5.8. One-Line Purpose of Each Application

- Shipment Application – Creates and validates the shipment order using workflow logic and Agentic AI inventory reasoning.
- Warehouse Application – Prepares the order for warehouse execution, planning, and carrier handoff using Agentic AI and Predictive AI support.
- Carrier Application – Updates delivery execution and completes the shipment lifecycle.

---

## 5.9. Final Overall Summary

This solution is a connected Shipment → Warehouse → Carrier workflow with both operational and communication layers.

Operationally, it supports order creation, inventory validation, shortage handling, procurement, warehouse preparation, carrier assignment, delivery execution, and final delivery confirmation.

Intelligently, it uses Agentic AI for inventory reasoning and sales forecasting, a Predictive Model plus Weather AI Agent for carrier delivery confidence, and GenAI for all generated business documents.

Communicatively, it keeps warehouse teams, carriers, and customers informed throughout the shipment lifecycle through automated emails and GenAI-generated documents.

---

## 5.10. Branding and Configuration

The solution supports dynamic branding to allow customization of company identity across all applications. This ensures the system can adapt to different organizations without requiring application changes.

### 5.10.1 Company Name Configuration

Users can configure a custom company name that is automatically displayed across the Shipment, Warehouse, and Carrier applications.

If no company name is configured, the system automatically displays a default company name: NextGen.

### 5.10.2 Logo Configuration

The application allows users to manage and update logo files used for branding. The selected logo is displayed consistently across all application headers and landing pages.

If no custom logo is configured, the system uses a default logo to ensure consistent visual presentation.

### 5.10.3 Runtime Behavior

The company name and logo are dynamically loaded at runtime. Users always see a valid company name and logo across all applications, ensuring there are no empty or unconfigured branding states.

### 5.10.4 User Experience Impact

From the user perspective, the application reflects organization-specific branding across all modules. The branding is applied automatically without requiring any additional setup during usage.

---

# 6. Deployment Instructions

### Import Applications to BA Studio
1. Navigate to BAA
1. Upload BAA .twx file
1. Navigate to BAW
1. Upload BAW .twx file
---

### BAW Configuration
1. Open the **Shipment Management System** application
2. Navigate to the **Servers** tab
3. Update the **Host Name** to match the current environment
4. Click **Test Connection**
- Ensure the connection is successful
5. Navigate to the exposed automation service:
- **CreateDataBase**
6. Run the service:
- **DatabaseCreation**

---

### Decision Setup

1. Create a Decision **"Approval Plan"**
   1. Import a decision service **"Approval Plan.zip"**
   2. Share Changes -> Create Version -> Deploy and publish it

2. Create a Decision **"Carrier Risk Automation"**
   1. Import a decision service **"Carrier Delivery Risk Decision"**
   2. Click on **"Carrier Delivery Failure Score"** and click on the **"Edit Configuration"** button
   3. Select **"Remote machine learning model"** and click **Next**
   4. Click **New Provider** and in the popup:
      - Select type as **"Embedded machine learning"**
      - Give the name **"Embedded Carrier PMML Provider"**
   5. Go back to **"Carrier Delivery Failure Score"** and click on the **"Edit Configuration"** button
   6. Select **"Remote machine learning model"** and click **Next**
   7. Select the machine learning provider **"Embedded Carrier PMML Provider"**
   8. Upload the file **"carrier_delivery_failure_scorecard_v2.pmml"** provided in the **Documents** folder
   9. Click **Next → Next → Apply**
   10. Go to **"Carrier Delivery Risk Decision"** -> Share Changes -> Create Version -> Deploy and publish it

## Navigator Setup

### Import Applications in IBM Navigator
1. Open the IBM Navigator
1. Click on Administration option from hamburger Menu
1. Click on Connections
2. Select **APPENGO**
3. Click **Connect**
4. Go to **Application**
5. Import the following **BAA Applications** (.zip files):
    - Shipment App
    - Warehouse App
    - Carrier App
1. On the app tile for the solution click on the menu and click **Preview** 
8. Note the URL from the tab that opens .

---

### Add folders
1. Create a folder named:
- **Shipment ops**
2. Inside **Shipment ops**, create a folder:
- **Templates**
3. Inside the **Templates** folder, upload the following PDF templates:
    - `PO_Receipt.pdf`
    - `proof_of_delivery.pdf`
    - `Shipping_Label.pdf`
4. Inside the **AppData** folder, upload the following PDF templates:
    - `Test Logo.png`

---

### BAI Setup

1. Choose the ** BAI .zip file** and click **Import** (bottom right corner) one by one

2. On the app tile for the solution , click and wait for the dashboard to open
  
3. Click on **Edit Details** button and add the newest version of the monitoring sources , after running the application atleast twice .   

### AI Agent setup

1. Follow the instructions: https://ibm.ent.box.com/notes/2277399246904 

### Customize the application

1. create folders : Shipment Ops > AppData > logo

2. Add default.jpeg. download the file from templates  
