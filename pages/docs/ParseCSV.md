## Authentication

FireMon APIs use either Basic Authentication or an Authentication Token. To use an authentication token, follow the guide [here](https://app.nuclino.com/FireMon/FireMon-Operations-Guide/FireMon-API-Authentication-Token-9a4f345d-f6be-4afc-adfa-d84c928c12cd "Nuclino").

## Step 1: Parse CSV Requirements

### Endpoint

| **Request Method** | **Endpoint**                                                                                                         |
| ------------------ | -------------------------------------------------------------------------------------------------------------------- |
| POST               | {{base\_url}}/policyplanner/api/policyplan/domain/{{domainId}}/workflow/{{workflowId}}/childKey/add\_access/parseCSV |

### Request Headers

| **Key**         | **Value**           |
| --------------- | ------------------- |
| Content-Type    | multipart/form-data |
| X-FM-Auth-Token | `token`             |
| Accept          | application/json    |
| Accept-Encoding | gzip, deflate, br   |

### Request Body

CSV File

### Parameters

| **Name**               | **Value**    |
| ---------------------- | ------------ |
| domainId               | `1`          |
| workflowId             | `workflowId` |
| requirementsDateFormat | MM/dd/yyyy   |

### Example Response

```json
{
    "policyPlanRequirementErrorDTOs": [
        {
            "policyPlanRequirementDTO": {
                "requirementType": "RULE",
                "changePlanReviewDecision": "UNDECIDED",
                "childKey": "add_access",
                "planned": false,
                "changes": [],
                "variables": {},
                "apps": [],
                "destinations": [
                    "192.168.10.0/24"
                ],
                "services": [
                    "tcp/443"
                ],
                "sources": [
                    "10.0.0.1"
                ],
                "users": [],
                "action": "ACCEPT"
            },
            "parsingErrors": []
        },
        {
            "policyPlanRequirementDTO": {
                "requirementType": "RULE",
                "changePlanReviewDecision": "UNDECIDED",
                "childKey": "add_access",
                "planned": false,
                "changes": [],
                "variables": {},
                "apps": [],
                "destinations": [
                    "192.168.10.0/24"
                ],
                "services": [
                    "tcp/443"
                ],
                "sources": [
                    "10.0.0.2"
                ],
                "users": [],
                "action": "ACCEPT"
            },
            "parsingErrors": []
        }
    ]
}
```

## Step 2: Parse Response from Step 1 to format Request Body for Step 3

From Step 1, you'll receive a JSON response that has a `"policyPlanRequirementErrorDTOs"` object which is an array of `"policyPlanRequirementDTO"` and `"parsingErrors"` objects. This JSON response will need to be reformatted into a new JSON object. This object will have a parent object `"requirements"` which will be an array containing the `"policyPlanRequirementDTO"` values from step one.

### Example Reformatted JSON Body

```json
{
    "requirements":
    [
        {
            "requirementType": "RULE",
            "changePlanReviewDecision": "UNDECIDED",
            "childKey": "add_access",
            "planned": false,
            "changes":
            [],
            "variables":
            {},
            "apps":
            [],
            "destinations":
            [
                "192.168.10.0/24"
            ],
            "services":
            [
                "tcp/443"
            ],
            "sources":
            [
                "10.0.0.1"
            ],
            "users":
            [],
            "action": "ACCEPT"
        },
        {
            "requirementType": "RULE",
            "changePlanReviewDecision": "UNDECIDED",
            "childKey": "add_access",
            "planned": false,
            "changes":
            [],
            "variables":
            {},
            "apps":
            [],
            "destinations":
            [
                "192.168.10.0/24"
            ],
            "services":
            [
                "tcp/443"
            ],
            "sources":
            [
                "10.0.0.2"
            ],
            "users":
            [],
            "action": "ACCEPT"
        }
    ]
}

```

### Example Python Function to format Requirements JSON

```python
def format_requirements(response_json: dict) -> dict:
    """
    Formats Requirement JSON after CSV Parsing
    :param response_json: JSON returned from CSV parsing
    :return: Formatted Requirements JSON
    """
    formatted_requirements = {"requirements":[]}
    for p in response_json['policyPlanRequirementErrorDTOs']:
        formatted_requirements['requirements'].append(p['policyPlanRequirementDTO'])
    return formatted_requirements
```

## Step 3: GET Ticket JSON

### Endpoint

| **Request Method** | **Endpoint**                                                                                            |
| ------------------ | ------------------------------------------------------------------------------------------------------- |
| GET                | {{base\_url}}/policyplanner/api/domain/{{domainId}}/workflow/{{workflowId}}/packet/{{workflowPacketId}} |

### Request Headers

| **Key**         | **Value**                       |
| --------------- | ------------------------------- |
| Content-Type    | application/json; charset=utf-8 |
| X-FM-Auth-Token | `token`                         |

### Parameters

| **Name**         | **Value**          |
| ---------------- | ------------------ |
| domainId         | `1`                |
| workflowId       | `workflowId`       |
| workflowPacketId | `workflowPacketId` |

## Step 4: Parse Ticket JSON for workflowTaskId

From Step 3, you'll receive a JSON response that has a `"status"` object and a `"workflowPacketTasks"` object. Here is an example, that has been reduced to show these objects.

```json
{
  "id": 75,
  "status": "Edit Request",
  "workflowPacketTasks": [],
}
```

The `"workflowPacketTasks"` is an array of objects. Each object in the array will have a `"name"` value and a `"workflowTask"` object.

The object in the array whose `"name"` value matches the `"status"` of the ticket and does not have a `"completed"` value is the is the object you'll want to target.

This object will have a a `"workflowTask"` object, which will have a corresponding `"id"` to use as `workflowTaskId`.

Stripped down Example:

```json
{
   "id":75, <- Use this value for workflowPacketId
   "status":"Edit Request",
   "workflowPacketTasks":[
      {
         "id":426,
         "workflowTask":{
            "id":18, <- Use this value for workflowTaskId
            "name":"Edit Request"
         }
      }
   ]
}
```

### Example Python Function to Parse Policy Planner ticket JSON for workflowTaskId

```python
def get_workflow_task_id(ticket_json: dict) -> str:
    """
    Retrieves workflowTaskId value from current stage of provided ticket
    :param ticket_json: JSON of ticket, retrieved using pull_ticket function
    :return: workflowTaskId of current stage for given ticket
    """
    curr_stage = ticket_json['status']
    workflow_packet_tasks = ticket_json['workflowPacketTasks']
    for t in workflow_packet_tasks:
        if t['workflowTask']['name'] == curr_stage and 'completed' not in t:
            return str(t['workflowTask']['id'])
```

## Step 5: POST Requirements to Policy Planner Ticket

### Endpoint

| **Request Method** | **Endpoint**                                                                                                                                            |
| ------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------- |
| POST               | {{base\_url}}/policyplanner/api/policyplan/domain/{{domainId}}/workflow/{{workflowId}}/task/{{workflowTaskId}}/packet/{{workflowPacketId}}/requirements |

*You can append* *`/replace`* *to the end of the above endpoint to replace all current requirements on the targeted ticket*

### Request Headers

| **Key**         | **Value**                       |
| --------------- | ------------------------------- |
| Content-Type    | application/json; charset=utf-8 |
| X-FM-Auth-Token | `token`                         |

### Parameters

| **Name**         | **Value**                    |
| ---------------- | ---------------------------- |
| domainId         | `1`                          |
| workflowId       | `workflowId`                 |
| workflowTaskId   | `workflowTaskId` from Step 4 |
| workflowPacketId | `workflowPacketId`           |

### Request Body Example

Request Body from Step 2

## Step 6: Stage Attachment to Policy Planner

### Endpoint

| **Request Method** | **Endpoint**                                                                                  |
| ------------------ | --------------------------------------------------------------------------------------------- |
| POST               | {{base\_url}}/policyplanner/api/domain/{{domainId}}/workflow/{{workflowId}}/packet/attachment |

### Request Headers

| **Key**         | **Value**           |
| --------------- | ------------------- |
| Content-Type    | multipart/form-data |
| X-FM-Auth-Token | `token`             |
| Accept          | application/json    |
| Accept-Encoding | gzip, deflate, br   |

### Parameters

| **Name**   | **Value**    |
| ---------- | ------------ |
| domainId   | `1`          |
| workflowId | `workflowId` |

### Request Body

CSV File

### Example Response

```json
{
    "attachments": [
        {
            "uuid": "e438cf8c-424d-4d47-8b02-ec7b75a71e5d",
            "contentType": "text/csv",
            "name": "test_req.csv",
            "size": 325
        }
    ]
}
```

## Step 7: Upload Attachment to Policy Planner Ticket

### Endpoint

| **Request Method** | **Endpoint**                                                                                                       |
| ------------------ | ------------------------------------------------------------------------------------------------------------------ |
| PUT                | {{base\_url}}/policyplanner/api/domain/{{domainId}}/workflow/{{workflowId}}/packet/{{workflowPacketId}}/attachment |

### Request Headers

| **Key**         | **Value**                           |
| --------------- | ----------------------------------- |
| Content-Type    | application/json;charset=UTF-8      |
| X-FM-Auth-Token | `token`                             |
| Accept          | application/json, text/plain, \*/\* |
| Accept-Encoding | gzip, deflate, br                   |

### Parameters

| **Name**         | **Value**          |
| ---------------- | ------------------ |
| domainId         | `1`                |
| workflowId       | `workflowId`       |
| workflowPacketId | `workflowPacketId` |

### Request Body

Response from Step 6, with a `description` added.

### Example Body

```json
{
    "attachments": [
        {
            "uuid": "e438cf8c-424d-4d47-8b02-ec7b75a71e5d",
            "contentType": "text/csv",
            "name": "test_req.csv",
            "size": 325,
            "description": "Attached original CSV file"
        }
    ]
}
```

