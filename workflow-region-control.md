# Oracle Workflow Region Control and Notification Templates

This guide documents various attributes and parameters that can be used to control Oracle Workflow regions, notifications, and templates.

## Workflow Region Items Handling

Control the display and behavior of workflow regions using these parameters:

| Parameter | Description |
|-----------|-------------|
| `#HIDE_MOREINFO` | Hides the "Request more information" option |
| `#HIDE_REASSIGN` | Hides the resign button |
| `#HISTORY` | Hides the Action History |
| `#FROM_ROLE` | Specifies role for source of information |
| `#WF_REASSIGN_LOV` | Specifies the users to whom a message can be reassigned |
| `#SUBMIT_COMMENTS` | Allows process owner to add a note for the action of submitting the process and sending the initial notification |
| `#WF_SIG_POLICY` | Requires a user's response to a notification be signed electronically |
| `#WF_SECURITY_POLICY` | Controls whether notifications with sensitive content can be sent in e-mail |
| `#RELATED_APPL` | Controls the Related Applications region which by default contains icons with links to attached URLs, documents, or forms |
| `#HDR_REGION` | Controls the notification header region which by default contains general message information such as from role, recipient, sent date, due date, and notification ID |

## Notification Mailer Attributes

Configure email-related attributes for notifications:

| Attribute | Description |
|-----------|-------------|
| `#WFM_FROM` | Specifies the value that appears in the From field of the message header when the e-mail notification message is delivered to a user |
| `#WFM_REPLYTO` | Specifies the address of the e-mail account that receives incoming messages, to which e-mail notification responses should be sent |
| `#WFM_HTMLAGENT` | Specifies the base URL that identifies the HTML web agent that handles HTML notification responses |

## Notification Mailer Message Template Attributes

Control notification templates for different scenarios:

| Attribute | Description |
|-----------|-------------|
| `#WFM_OPEN_MAIL` | Template for e-mail notifications that require a response (templated response method) |
| `#WFM_OPEN_MAIL_DIRECT` | Template for e-mail notifications that require a response (direct response method) |
| `#WFM_OPEN_MAIL_FYI` | Template for e-mail notifications that do not require a response |
| `#ATTACHED_URLS` | Template for creating the Notification References attachment for HTML-formatted notification messages that include URL attributes with Attach Content checked |
| `#WFM_CANCELED` | Template to inform the recipient that a previously sent notification is canceled |
| `#WFM_OPEN_INVALID` | Template to send when the user responds incorrectly to a notification |
| `#WFM_CLOSED` | Template to inform the recipient that a previously sent notification is now closed |
| `#WFM_OPEN_MORE_INFO` | Template to use to send a request for more information from one user to another user |

## Implementation Examples

### Hiding Specific Workflow Regions

```sql
-- In your PL/SQL workflow process
wf_engine.SetItemAttr(itemtype  => l_itemtype,
                     itemkey   => l_itemkey, 
                     aname     => '#HIDE_REASSIGN',
                     avalue    => 'Y');
                     
wf_engine.SetItemAttr(itemtype  => l_itemtype,
                     itemkey   => l_itemkey, 
                     aname     => '#HIDE_MOREINFO',
                     avalue    => 'Y');
```

### Setting Up Email Templates

```sql
-- Set FROM address for notifications
wf_engine.SetItemAttr(itemtype  => l_itemtype,
                     itemkey   => l_itemkey, 
                     aname     => '#WFM_FROM',
                     avalue    => 'workflow@example.com');

-- Set template for notifications requiring response
wf_engine.SetItemAttr(itemtype  => l_itemtype,
                     itemkey   => l_itemkey, 
                     aname     => '#WFM_OPEN_MAIL',
                     avalue    => 'CUSTOM_RESPONSE_TEMPLATE');
```

### Customizing Notification Security

```sql
-- Require electronic signature for responses
wf_engine.SetItemAttr(itemtype  => l_itemtype,
                     itemkey   => l_itemkey, 
                     aname     => '#WF_SIG_POLICY',
                     avalue    => 'REQUIRED');

-- Control sensitive content in emails
wf_engine.SetItemAttr(itemtype  => l_itemtype,
                     itemkey   => l_itemkey, 
                     aname     => '#WF_SECURITY_POLICY',
                     avalue    => 'NO_EMAIL');
```

## Oracle APEX Integration

When integrating Oracle Workflows with Oracle APEX, you can use JavaScript and PL/SQL to enhance the workflow experience:

### Workflow Region Control in APEX

```javascript
// Example JavaScript to hide workflow regions dynamically
function customizeWorkflowRegions() {
    // Hide reassign button based on user role
    if (!hasRequiredRole('WF_ADMIN')) {
        $(".wf-reassign-button").hide();
    }
    
    // Hide more info request for completed items
    if ($("#P1_WF_STATUS").val() === "COMPLETED") {
        $(".wf-moreinfo-region").hide();
    }
}

// Apply customizations on page load
$(document).ready(function() {
    customizeWorkflowRegions();
});
```

### Setting Workflow Attributes from APEX

```sql
-- PL/SQL process to set workflow attributes from APEX page items
DECLARE
    l_itemtype VARCHAR2(100) := :P1_ITEMTYPE;
    l_itemkey  VARCHAR2(100) := :P1_ITEMKEY;
BEGIN
    -- Set attributes based on APEX page values
    wf_engine.SetItemAttr(
        itemtype => l_itemtype,
        itemkey  => l_itemkey, 
        aname    => '#FROM_ROLE',
        avalue   => :P1_CURRENT_ROLE
    );
    
    -- Conditionally hide reassign button
    IF :P1_ALLOW_REASSIGN = 'N' THEN
        wf_engine.SetItemAttr(
            itemtype => l_itemtype,
            itemkey  => l_itemkey, 
            aname    => '#HIDE_REASSIGN',
            avalue   => 'Y'
        );
    END IF;
    
    -- Apply custom email template
    wf_engine.SetItemAttr(
        itemtype => l_itemtype,
        itemkey  => l_itemkey, 
        aname    => '#WFM_OPEN_MAIL',
        avalue   => :P1_EMAIL_TEMPLATE
    );
END;
```

## Best Practices

1. **Documentation**: Always document which attributes are used in each workflow
2. **Testing**: Test notification templates in all supported email clients
3. **Security**: Be cautious with `#WF_SECURITY_POLICY` settings for sensitive data
4. **Maintenance**: Review and update notification templates regularly
5. **Performance**: Minimize the number of attributes set for each workflow instance

## Additional Resources

For more detailed information, refer to Oracle's official documentation:
[Oracle Workflow Documentation](https://docs.oracle.com/cd/B14099_19/integrate.1012/b12161/defcom42.htm)