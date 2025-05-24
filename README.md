# Jira JQL to get user-related issues by date
A jira JQL that can help you fetch all issues in context with your (or any other) user from a specific date or date range. Perfect for scripting purposes on the Jira API.

# Who's it for?
Anyone who has to fill out timesheets for their work and can't even remember what they've had for lunch two hours ago.
> [!IMPORTANT]
> You likely have custom fields and attributes in your jira instance, like custom role levels. It's up to you to add those customizations to the query.

# Query breakdown
Here's the query split into its base components as well as propositions for variable names in scripting.

## Global variables
I'm gonna assume you want to script this and will reference below varialbes with a `$` prefix in the queries.
|Usage|Key|Value|
|---|---|---|
|Define start date (time will default to 00:00:00)|`startDate`|`1970-01-01`|
|Define end date (time will default to 00:00:00)|`endDate`|`1970-01-02`|
|OPTIONAL: Define username to search for someone else than oneself|`jiraUser`|`currentUser()`|

## JQL subqueries
These are separate, working subqueries which can later be assembled into a larger, complex JQL.  
You will notice that the variable keys are split into `jqlWho` and `jqlWhen` prefixes to divide them further into manageable fragments for easier management in the code.

Note that the queries are written as you would paste them in the web query editor of jira. For any programming or scripting language it is likely necessary to additionally escape the double quotes (`"`) and backslashes (`\`) for proper interpretation.

### Date filters (`jqlWhen` prefix)
|Usage|Key|Value|
|---|---|---|
|Range for issue creation date|`jqlWhenCreated`|`(created >= $startDate AND created < $dateMax)`|
|Range for issue update date|`jqlWhenUpdated`|`(updated >= $startDate AND updated < $dateMax)`|
|Status changes in the issue workflow|`jqlWhenStatusChanged`|`(status CHANGED ON $startDate)`|
|Reporter changes|`jqlWhenReporterChanged`|`(reporter CHANGED ON $startDate)`|
|Assignee changes|`jqlWhenAssigneeChanged`|`(assignee CHANGED ON $startDate)`|
|Fix version changes|`jqlWhenFixversionChanged`|`(fixVersion CHANGED ON $startDate)`|
|Priority changes|`jqlWhenPriorityChanged`|`(priority CHANGED ON $startDate)`|
|Resolution changes|`jqlWhenResolutionChanged`|`(resolution CHANGED ON $startDate)`|
|Unrestricted comments|`jqlWhenCommentedBasic`|`issueFunction IN commented("on $startDate")`|
|Restricted comments to Administrators|`jqlWhenCommentedAdmin`|`issueFunction IN commented("on $startDate rolelevel \"Administrators\"")`|
|Restricted comments to Developers|`jqlWhenCommentedDev`|`issueFunction IN commented("on $startDate rolelevel \"Developers\"")`|
|Restricted comments to Users|`jqlWhenCommentedUsers`|`issueFunction IN commented("on $startDate rolelevel \"Users\"")`|

### User filters (`jqlWho` prefix)
|Usage|Key|Value|
|---|---|---|
|Creator is user|`jqlWhoCreator`|`creator = $jiraUser`|
|Reporter is user|`jqlWhoReporter`|`reporter = $jiraUser`|
|Reporter edited by user|`jqlWhoReporterEdit`|`reporter CHANGED BY $jiraUser`|
|Assignee is user|`jqlWhoAssignee`|`assignee = $jiraUser`|
|Assignee edited by user|`jqlWhoAssigneeEdit`|`assignee CHANGED BY $jiraUser`|
|Watcher is user|`jqlWhoWatcher`|`watcher = $jiraUser`|
|Watcher edited by user|`jqlWhoStatusEdit`|`status CHANGED BY $jiraUser`|
|Fix version changed by user|`jqlWhoFixversionEdit`|`fixVersion CHANGED BY $jiraUser`|
|Priority edited by user|`jqlWhoPriorityEdit`|`priority CHANGED BY $jiraUser`|
|Resolution changed by user|`jqlWhoResolutionEdit`|`resolution CHANGED BY $jiraUser`|
|Worklog time booked by user|`jqlWhoWorklogAuthor`|`worklogAuthor = $jiraUser`|

### Subquery assembly
Assemble the subqueries into one large query. Shortened in the example below, for brevity.
|Usage|Key|Value|
|---|---|---|
|Assemble all time queries|`jqlWhen`|`($jqlWhenCreated OR $jqlWhenUpdated OR ...)`|
|Assemble all user queries|`jqlWho`|`($jqlWhoCreator OR $jqlWhoReporter OR ...)`|
|Combine time and user queries|`jql`|`$jqlWhen AND $jqlWho`|

## Query result
You query should end up looking something like this, after assembling and printing the `$jql` variable:
```SQL
(
    (created >= 1970-01-01 AND created < 1970-01-02)
    OR (updated >= 1970-01-01 AND updated < 1970-01-02)
    OR (status CHANGED ON 1970-01-01)
    OR (reporter CHANGED ON 1970-01-01)
    OR (assignee CHANGED ON 1970-01-01)
    OR (fixVersion CHANGED ON 1970-01-01)
    OR (priority CHANGED ON 1970-01-01)
    OR (resolution CHANGED ON 1970-01-01)
    OR (
        issueFunction IN commented("on 1970-01-01")
        OR issueFunction IN commented("on 1970-01-01 rolelevel \"Administrators\"")
        OR issueFunction IN commented("on 1970-01-01 rolelevel \"Developers\"")
        OR issueFunction IN commented("on 1970-01-01 rolelevel \"Users\"")
    )
)
AND (
    creator = currentUser()
    OR reporter = currentUser()
    OR reporter CHANGED BY currentUser()
    OR assignee = currentUser()
    OR assignee CHANGED BY currentUser()
    OR watcher = currentUser()
    OR status CHANGED BY currentUser()
    OR fixVersion CHANGED BY currentUser()
    OR priority CHANGED BY currentUser()
    OR resolution CHANGED BY currentUser()
    OR worklogAuthor = currentUser()
)
```