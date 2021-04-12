# Known Issue: WI Form IndexedDB caches rules causing some server side rule evaluations to not be reevaluted
## Issue
This issue may cause rules to not evaluate properly on the web client if a users's group memebership is updated.  For example, a user is added to a group that lifts a restricted state change, but their client doesn't immediately accept that new ability.

You can see a real life example of this in [this work item](https://dev.azure.com/mseng/AzureDevOps/_workitems/edit/1812997/) where a user was added to bug creators but still is unable to set the state to "Proposed" because the rules were evaluated when the user was not part of that group.

If you hit this and take no action, the cache will get a full update of membership information within 3 days, and the issue will self-resolve.

### Cause
The Work Item Form uses IndexedDB to cache the Work Item Type information as well as a couple of other pieces of Work Item related info.  Included in Work Item Type is the rules of that work item.  User-based XML restrictions ("for" and "not") are evaluated on the server-side and Prohibited/Allowed values are added to the rules based on the evaluation. 

Unfortunately, this means if team/user changes happen then those rules will not be reevaluated as we will get the rules from the cache. Invalidating the cache on updates to teams/users is a potential solution but due to performance constraints, we cannot send all group membership changes to update the cache on the client.

## Fixes
### To Avoid
To avoid triggering this issue in the first place, adding yourself to all the relevant teams before attempting to create a work item would ensure that the cache stores the correct information.

### Clearing indexDB
If the cache is already storing the incorrect rules, you can either wait for the client cache to expire (3 days max), or can clear the cache by running the following command in the browser command window and refreshing:
`window.indexedDB.deleteDatabase("wit")`
