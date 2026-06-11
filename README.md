# ATLAS — IT Equipment Provisioning MCP Server

ATLAS is a Workato-based MCP (Managed Content Processing) server that automates IT equipment provisioning for employees. It handles the full lifecycle from inventory check to approval — using Google Sheets, Salesforce, Jira, and Email.

---

## Recipes

| Code | Recipe | Description |
|------|--------|-------------|
| `CI` | check_inventory | Looks up available equipment by category |
| `GEP` | get_employee_profile | Fetches employee details and entitlements |
| `RI` | reserve_item | Reserves equipment items for an employee |
| `CPT` | create_provisioning_ticket | Creates a Jira ticket and logs the reservation |
| `SAR` | send_approval_request | Sends an approval email to the manager |

---

## Architecture

The recipes work together in a sequence to fulfil an equipment request:

```mermaid
flowchart LR
    CI[CI\nCheck Inventory] --> GEP[GEP\nGet Employee Profile]
    GEP --> RI[RI\nReserve Item]
    RI --> CPT[CPT\nCreate Provisioning Ticket]
    CPT --> SAR[SAR\nSend Approval Request]
```

---

## Recipe Flows

### CI — Check Inventory

**Inputs:** `category`  
**Outputs:** `items`

```mermaid
flowchart TD
    T([🔵 Function call · Real-time\nTriggered with equipment category])
    T --> A1
    subgraph ACTIONS
        A1[1 · Search rows in Google Sheets\nFinds available items matching the given category]
        A2([2 · RETURN result\nReturns list of matching inventory items])
        A1 --> A2
    end
```

---

### GEP — Get Employee Profile

**Inputs:** `employee_email`  
**Outputs:** `employee_id`, `full_name`, `role`, `department`, `manager_email`, `entitlement`

```mermaid
flowchart TD
    T([🔵 Function call · Real-time\nTriggered with employee email])
    T --> A1
    subgraph ACTIONS
        A1[1 · Search Salesforce\nLooks up employee record by email]
        A2[2 · Search rows in Google Sheets\nFetches entitlement policy for the employee's role]
        A3([3 · RETURN result\nReturns profile and entitlement data])
        A1 --> A2 --> A3
    end
```

---

### RI — Reserve Item

**Inputs:** `employee_email`, `employee_id`, `employee_full_name`, `items`  
**Outputs:** `reservation_id`, `status`

```mermaid
flowchart TD
    T([🔵 Function call · Real-time\nTriggered with employee and item details])
    T --> A1
    subgraph ACTIONS
        A1[1 · Declare variable\nInitialises reservation tracking variable]
        A2{2 · FOR EACH item · REPEAT}
        A3[3 · Search rows in Google Sheets\nChecks stock availability for the item]
        A4[4 · Update row in Google Sheets\nMarks item as reserved]
        A5[5 · Add row in Google Sheets\nLogs the reservation entry]
        A6([6 · RETURN result\nReturns reservation ID and status])
        A1 --> A2
        A2 --> A3 --> A4 --> A5 --> A2
        A2 -->|done| A6
    end
```

---

### CPT — Create Provisioning Ticket

**Inputs:** `employee_email`, `reservation_id`, `priority`, `items`  
**Outputs:** `ticket_key`, `ticket_url`, `status`, `reservation_id`

```mermaid
flowchart TD
    T([🔵 Function call · Real-time\nTriggered with reservation details])
    T --> A1
    subgraph ACTIONS
        A1[1 · Create issue in Jira\nCreates an IT Helpdesk task with email, items, and priority]
        A2[2 · Search rows in Google Sheets · Batch\nFinds reservations matching the reservation ID]
        A3{3 · FOR EACH row · REPEAT}
        A4[4 · Update row in Google Sheets\nWrites Jira ticket key back to the reservation row]
        A5([5 · RETURN result\nReturns ticket key, URL, status, and reservation ID])
        A1 --> A2 --> A3
        A3 --> A4 --> A3
        A3 -->|done| A5
    end
```

---

### SAR — Send Approval Request

**Inputs:** `manager_email`, `employee_name`, `items`, `ticket_url`, `reservation_id`  
**Outputs:** `status`, `sent_to`

```mermaid
flowchart TD
    T([🔵 Function call · Real-time\nTriggered with manager and ticket details])
    T --> A1
    subgraph ACTIONS
        A1[1 · Send email\nSends approval request to the manager with ticket link]
        A2([2 · RETURN result\nReturns send status and recipient email])
        A1 --> A2
    end
```

---

## Integrations

| Service | Used in |
|---------|---------|
| Google Sheets | CI, RI, CPT, GEP |
| Salesforce | GEP |
| Jira | CPT |
| Email | SAR |

---

## How to Import

1. Download the `.recipe.json` file for the recipe you want
2. In Workato, go to **Projects → Import**
3. Upload the JSON file
4. Configure the connections (Google Sheets, Salesforce, Jira, Email) to match your workspace

---

## Project Structure

```
ATLAS/
├── README.md
├── ci_rec_check_inventory.recipe.json
├── gep_rec_get_employee_profile.recipe.json
├── ri_rec_reserve_item.recipe.json
├── cpt_rec_create_provisioning_ticket.recipe.json
└── sar_rec_send_approval_request.recipe.json
```
