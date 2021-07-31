## Utility Workflows & Atomic Actions for SecureX Orchestrator

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


    <br>Pro Tip: Oversight can also monitor triggers attached to your workflow and invoke your error handler if the trigger status is not in `Started-polling`!

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
