## Utility Workflows & Atomic Actions for SecureX Orchestrator

[![published](https://static.production.devnetcloud.com/codeexchange/assets/images/devnet-published.svg)](https://developer.cisco.com/codeexchange/github/repo/ciscomanagedservices/sxo-utilities) [![sxoanalyzed](https://svgshare.com/i/_4q.svg)](https://github.com/ciscomanagedservices/sxo-analyzer)

Please refer to descriptions provided within the atomic actions/workflows for additional guidance on usage.

### In this repository, you'll find the following workflows & atomic actions:

1. **SecureX Orchestrator**
    
    <details>
    <summary><strong>Generate AO API Token [<a href="/SXO-GenerateAPIToken__definition_workflow_01NILYC8ELB2P35zuxiTzIZA63s12AUnVjL/">link</a>]</strong></summary>
    
    <br>WARNING: EXPERIMENTAL USE ONLY
    
    SecureX Orchestrator (SXO) APIs are not generally available/exposed in production yet. However, if the ‘AO’ API scope were to be added to a SecureX API Client, this workflow demonstrates how to use such an API Client to generate and retrieve a JWT token that can be used to make API calls to SXO, assuming the API methods are known to the user. The methodology used in this workflow may be subject to change once this functionality is available in production.

    Steps involved:
    1.	Generate a new Access Token for Cisco Threat Response (CTR) using the SecureX API Client (with AO scope added) credentials defined in the Account Key for the ‘CTR Admin’ target. This returns the SecureX Auth Token.
    2. Use the SecureX Auth Token to make a call to CTR’s AO endpoint to generate the JWT
    3.	Clean up response
    4.	Output a secure string with the SXO JWT

    <br>Observations & Usage Advise:

    1.	A given SXO JWT is valid only for ~10 minutes and needs to be regenerated thereafter using the same methodology.
    2.	For northbound access to the API, it is recommended to run this workflow on a schedule (every <10 min), preserve the SXO JWT in a central secret management system like HashiCorp Vault & have external services retrieve the token from the Vault prior to making API calls to SXO. There are atomic actions available for Vault on this GitHub repo.
    3.	If you do not have access to the AO API docs, you may benefit from the Swagger JSONs available provided in this repo [here](/SXO-GenerateAPIToken__definition_workflow_01NILYC8ELB2P35zuxiTzIZA63s12AUnVjL/) that you may use with the Swagger HTTP Activity

    </details>
    
    <details>
    <summary><strong>Oversight (Error Handling Framework) [<a href="/SXO-Oversight__definition_workflow_01PKT8VMZQBWQ1KvAii6vD7lk3YfFI2HEJj/">link</a>]</strong></summary>
    
    <br>Oversight is an unsupervised error handling framework for SecureX Orchestrator (SXO) workflows. 

    Oversight monitors a set of workflows (that you define) and based on what type of error (that you define) may be seen, triggers an appropriate error handler workflow (that you define).
    
    NOTE: This workflow relies on a SecureX API Client with the 'AO' API scope.

    Steps to use:
    1. Import this workflow to your SXO Org. Ensure you have your `CTR Admin` target setup that [this](/SXO-GenerateAPIToken__definition_workflow_01NILYC8ELB2P35zuxiTzIZA63s12AUnVjL/) workflow uses to pass a valid SXO JWT to Oversight.
    2. You may want to put this workflow on a schedule (default, daily 9 AM UTC) or trigger it via an external system based on your use-case or time criticality of the workflows you're monitoring.
    3. The atomic action creates a Global Variable table named `Oversight - Monitored Workflows`. Access this table via `Variables` > `Global Variables` to add what Workflow IDs (last path segment available in the URL when you open a workflow) you'd like oversight to monitor, what error regular expression signatures/patterns you'd like to look for and what Workflow ID you'd like oversight to call when the error signature/pattern is seen.
    4. Your handler workflow must have at least the following three input variables for oversight to pass error context into:
       1. `Message` (string): the exact error message
       2. `Instance Id` (string): link to the instance run where the error is seen
       3. `Activity Name` (string): name of the activity in the instance where the error is seen


    <!--br>💡 **Pro Tip:** Oversight can also monitor triggers attached to your workflow and invoke your error handler if the trigger status is not in `Started-polling`. -->

    </details>
    
    <details>
        
    <summary><strong>Breadcrumbs (Logging & Tracing Framework) [<a href="/SXO-BreadcrumbsLoggingBlock__definition_workflow_01KT1XNI3HHD76eXrxg4GfbATOyZdxTjyLL/">logging workflow</a>] [<a href="/SXO-BreadcrumbsTriggerTrace__definition_workflow_01QI4N9T2R2GH5g8GWPWpr6V06NZlpV298Q/">tracing workflow</a>]</strong></summary>

    <br>Breadcrumbs lets you add checkpoints to your SecureX Orchestrator (SXO) workflows that you can then use to track progress in near real-time or retrospectively via external systems. This can be specially useful as a stack trace for error/exception handling. 

    NOTE: Breadcrumbs relies on a SecureX API Client with the 'AO' API scope.

    In this repository, there are **two** workflows related to Breadcrumbs:
    1. **Breadcrumbs - Logging Block** [[link](/SXO-BreadcrumbsLoggingBlock__definition_workflow_01KT1XNI3HHD76eXrxg4GfbATOyZdxTjyLL/)]
    <br>Place this workflow strategically in multiple 'checkpoints' of your parent workflow and Breadcrumbs will store all progress between two consecutive logging blocks in a table data type named `All My Breadcrumbs`.

        **Steps to use:**
        1. Import the workflow to your SXO Org. Ensure you have the `CTR Admin` target in your Org setup that [this](/SXO-GenerateAPIToken__definition_workflow_01NILYC8ELB2P35zuxiTzIZA63s12AUnVjL/) workflow uses to pass a valid SXO JWT token to Breadcrumbs.
        2. This atomic action creates a Global Variable table named `All My Breadcrumbs` to log to. The table has the following columns:
            1. `Alternate Instance ID`: A friendly name that you provide to make it easier for you to identify & retrieve data for your workflow instance. This name can also have run time variables.
            2. `Instance ID`: The instance or 'run' identifier of the workflow.
            3. `Activity ID`: An identifier for each activity or block in your workflow between two consecutive Breadcrumbs.
            4. `Log`: The name of the activity with any custom log messsage. The custom log message can also have run time variables.
            5. `State`: Activity Status (success/error)
            6. `Timestamp`: Activity timestamp in YYYY-MM-DDThh:mm:ss.sssZ format
        3. Drop this workflow (it should be visible under the 'workflows' tab in the left pane as `Breadcrumbs - Logging Block`) into your parent workflow wherever you deem appropriate. You're free to drop as many logging blocks as you'd like, however, each block you drop to your workflow adds ~5 seconds (may vary under heavy load) to your workflow execution time.

            > 💡 **Pro Tip:** A best practice is to place a logging block immediately after an activity that involves communicating with an external entity (like a third-party API or database). You can then use the logging block to capture progress or any errors that the third-party returns.
        4. You'll notice three input variables: an `Instance ID` (required), an `Alternate Instance ID` (optional) & a `Custom Log Message` (optional).
            1. Populate the `Instance ID` using the Variable Browser > `Workflow` > `Output` > `Instance Id`. 
            2. A good naming practice for your `Alternate Instance ID` is `<some text that identifies your workflow> - <workflow start time>`. You can dynamically populate the workflow start time using the Variable Browser > `Workflow` > `Output` > `Start time`.
            3. Whatever you define in the `Custom Log Message` is suffixed to the Activity Name & stored.
        5. Run the workflow you've added Breadcrumbs to. You should be able to see new entries added to the table now.

    2. **Breadcrumbs - Trigger Trace** [[link](/SXO-BreadcrumbsTriggerTrace__definition_workflow_01QI4N9T2R2GH5g8GWPWpr6V06NZlpV298Q/)]
    <br>A sample workflow that can be externally triggered via SXO's REST API to trace the breadcrumbs of another workflow from the `All My Breadcrumbs` table & output progress in near real-time or retrospectively.

        **Steps to use:**
        1. Import the workflow to your SXO Org. 
        2. Supply an `Instance ID` or an `Alternate Instance ID` as input to this workflow to search for corresponding Breadcrumbs. If you use the `Alternate Instance ID`, you can also provide a _fuzzy_ string to search with, i.e. you do not need to provide an exact match. 
        3. The output of this workflow is a JSON string with Breadcrumbs corresponding to your search. If there are multiple matches, multiple objects are returned, like:

            ```json
            {
                "Sample WF1 - 2021-09-06T08:17:22.928216678Z": [{
                        "activity_id": "01R5OZ5IQY9CN3x9onnjR7z80wQoYb7ZkPh",
                        "alt_instance_id": "Sample WF1 - 2021-09-06T08:17:22.928216678Z",
                        "instance_id": "01R5OZ5IPN8HS5RvvYIcSoQsFhwhxiXjpae",
                        "log": "Split String",
                        "state": "success",
                        "timestamp": "2021-09-06T08:17:23.921234578Z"
                    }],
                "Sample WF1 - 2021-09-09T09:10:10.233331678Z": [{
                    "activity_id": "01R5OZ5IQY9CN3x9onnjR7z80wQoYb7ZkPh",
                    "alt_instance_id": "Sample WF1 - 2021-09-09T09:10:10.233331678Z",
                    "instance_id": "01S5O344N8HS5RvvYIcSoQsFh244iXjreq",
                    "log": "Split String",
                    "state": "success",
                    "timestamp": "2021-09-09T09:10:11.232115338Z"
                }]
            }
            ``` 

    **Usage Guidance:**

    1. Since logging blocks are added sequentially to your workflow, you may want to ensure that "Continue Execution on Failure" is checked on your critical activities for a logging block to be able to capture errors that may occur in activities preceeding it.
    2. A logging block must exist at the same level as the activities it needs to log. For example: placing a logging block just after a conditional block **does not** automatically log all activities inside the conditional - it only logs the conditional's overall state; if you'd like to log activities inside a conditional branch, you need to place one or more logging blocks within that conditional branch.
    3. For usage in parallel blocks, you must have an active logging block in **one parallel branch only** and 'silent' logging blocks in all other parallel branches within the parallel block. A 'silent' logging block is one that is placed with the "Skip Activity Execution" parameter checked. This prevents the silent block from logging the activities above it, but can still be _seen_ by the subsequent (non-silent or active) logging block as the preceeding logging block.
    4. The architecture that Breadcrumbs uses is for demonstration purposes. In production, you may want to use a database for more performant logging as opposed to using SXO Table Data Types. We tend to implement one of three methodologies for production projects:

        | Methodology                                                     | Pros                                                                                                                                           | Cons                                                                                                                                                               |
        |-----------------------------------------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------|
        | Log to SXO tables alone                                         | - Quickest write executions for a small number of workflows<br>- Minimal operational overhead                                                  | - Not scalable for concurrent reads/writes, i.e. multiple parent workflows logging breadcrumbs concurrently to a single SXO table.                                                       |
        | Log to SXO tables as cache, move to a DB for persistent logging on a schedule | - Ideal for large scale workflows that require quick write execution<br>- Benefit from quick writes to SXO tables and performant reads from DB | - Requires an external database<br>- Added overhead to maintain a separate workflow that syncs cache to database                                                   |
        | Log directly to a DB                                            | - Handle concurrent reads/writes with ease                                                                                                     | - Slowest execution, prolongs workflow run time (recommended for workflows that prioritize reliability over speed of execution)<br>- Requires an external database |

    </details>

  ---
  
2. **String**
    
    <details>
    <summary><strong>Basic Auth Encode [<a href="/String-Base64AuthEncode__definition_workflow_01P6CIVMFKOUU3CwI3kPJhsn8NmUkUvlodw/">link</a>]</strong></summary>
    
    <br>This atomic action takes a username & password and returns a secure string with the Base64 equivalent that can be readily passed to an HTTP Basic Authorization Header.

    </details>

  ---
  
3. **HashiCorp Vault**

    <details>
    <summary><strong>Create/Update Secret [<a href="/Vault-CreateUpdateSecret__definition_workflow_01OR0GMVJSGWV0BfwgnaW7ppXJWKlkg40Bb/">link</a>]</strong></summary>
    
    <br>WARNING: This will overwrite all the secrets stored in the path specified.
    
    Pre-Requisite: Create an HTTP Target for your HashiCorp Vault instance and point this atomic to it.
    
    Docs: https://www.vaultproject.io/api/secret/kv/kv-v2#create-update-secret

    </details>

    <details>
    <summary><strong>Renew Vault Token [<a href="/Vault-RenewToken__definition_workflow_01OR1PAO3TAW07ao4DevuwZOBAoDcSkNKjm/">link</a>]</strong></summary>
            
    <br>Pre-Requisite: Create an HTTP Target for your HashiCorp Vault instance and point this atomic to it.
    
    This atomic returns the same token you pass in, but the lease period for the given token will be extended. The ideal way to use this would be to use this atomic in a workflow, which would be run on a schedule. Ensure that the "Clean Up After Successful Execution" option is checked on the workflow to avoid exposure of sensitive/privileged information.
    
    Docs: https://www.vaultproject.io/api-docs/auth/token#renew-a-token

    </details>

    <details>
    <summary><strong>Retrieve Secret [<a href="/Vault-RetrieveSecret__definition_workflow_01OQ7P0UDJKGS6WRwjfMEktE3hb3NFqg7he/">link</a>]</strong></summary>

    <br>No Targets need to be pre-defined for this atomic action. Instead, information is fed in as inputs.
    
    The atomic action expects the secret path to be of the form `engine/data/secret`, where `engine` refers to the highest level secret engine and `secret` refers to the secret stored in the engine. Other inputs are self-explanatory. Output is a Secure String variable called `Token` that is the token retrieved from Vault for a given key/identifier.
    
    Docs: https://www.vaultproject.io/docs/secrets/kv/kv-v1

    </details>

---

Contributors:

1. Aman Sardana (amasarda@cisco.com)
2. Anant Nambiar (ananambi@cisco.com)

Cisco CX Managed Services - Operate, July 2021
