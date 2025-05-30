# Oracle Workflow Adhoc Roles Management Guide

This guide provides detailed instructions for managing adhoc roles in Oracle Workflow, including creation, modification, and deletion of roles, as well as adding or removing users.

---

## **1. Creating an Adhoc Role**
To create a new adhoc role, use the `WF_DIRECTORY.CREATEADHOCROLE` procedure. The role will automatically expire after a specified date.

### **Example:**
```sql
L_ROLE_NAME := 'INTG_ESCALTE_EMAIL_ADH';
L_ROLE_DISPLAY_NAME := 'INTG Escalation Emails Adhoc';
L_USER_LISTS := 'USER_NAME1,USER_NAME2';

WF_DIRECTORY.CREATEADHOCROLE(
    ROLE_NAME          => L_ROLE_NAME,
    ROLE_DISPLAY_NAME  => L_ROLE_DISPLAY_NAME,
    ROLE_DESCRIPTION   => 'Default Role for LPS Project Approval Notifications',
    ROLE_USERS         => L_USER_LISTS,
    EXPIRATION_DATE    => SYSDATE + 15
);
```

### **Parameters:**
- **`ROLE_NAME`**: Unique identifier for the role (e.g., `INTG_ESCALTE_EMAIL_ADH`).
- **`ROLE_DISPLAY_NAME`**: Human-readable name for the role.
- **`ROLE_DESCRIPTION`**: Brief description of the role.
- **`ROLE_USERS`**: Comma-separated list of usernames to be added to the role.
- **`EXPIRATION_DATE`**: Date when the role expires (e.g., `SYSDATE + 15` for 15 days from now).

---

## **2. Adding Users to an Adhoc Role**
Use `WF_DIRECTORY.AddUsersToAdHocRole` to add users to an existing role.

### **Example:**
```sql
BEGIN
    WF_DIRECTORY.AddUsersToAdHocRole(
        role_name  => 'PHANI_DEMO_ROLE',
        role_users => 'USERNAME1'
    );
END;
```

### **Notes:**
- Usernames **must be in uppercase**.
- Multiple usernames should be space-separated.

---

## **3. Removing Users from an Adhoc Role**
Use `WF_DIRECTORY.RemoveUsersFromAdHocRole` to remove users from a role.

### **Example:**
```sql
BEGIN
    WF_DIRECTORY.RemoveUsersFromAdHocRole(
        role_name  => 'PHANI_DEMO_ROLE',
        role_users => 'USER3'
    );
END;
```

### **Notes:**
- Usernames **must be in uppercase**.
- Multiple usernames should be space-separated.

---

## **4. Checking Role and User Associations**
To verify role details and associated users, run the following queries:

### **Check Role Details:**
```sql
SELECT * FROM wf_roles WHERE name LIKE '%INTG_ESCALTE_EMAIL_ADH%';
```

### **Check Users in Role:**
```sql
SELECT * FROM wf_user_roles WHERE role_name LIKE '%INTG_ESCALTE_EMAIL_ADH%';
```

---

## **5. Expiring and Deleting an Adhoc Role**
There are two methods to expire and delete an adhoc role:

### **Method 1 (Manual Expiration & Cleanup):**
```sql
BEGIN
    apps.wf_directory.RemoveUsersFromAdHocRole(
        role_name  => 'ROLE_NAME',
        role_users => 'USER1 USER2'
    );
    
    UPDATE wf_local_roles 
    SET expiration_date = SYSDATE - 1,
        status = 'INACTIVE'
    WHERE name = 'ROLE_NAME';
    
    COMMIT;
    wf_purge.AdHocDirectory();
    COMMIT;
END;
```

### **Method 2 (Recommended - Automated Cleanup):**
```sql
BEGIN
    WF_DIRECTORY.SetAdHocRoleExpiration('ROLE_NAME', SYSDATE - 1);
    COMMIT;
    
    WF_DIRECTORY.SETADHOCROLESTATUS('ROLE_NAME', 'INACTIVE');
    COMMIT;
    
    WF_DIRECTORY.deleterole('ROLE_NAME', 'WF_LOCAL_ROLES', 0);
    COMMIT;
END;
```

### **Notes:**
- Method 2 is more efficient as it uses built-in procedures for expiration and deletion.
- Ensure the role name is correctly specified.

---

## **Best Practices**
1. **Use Uppercase for Usernames**: Oracle Workflow is case-sensitive.
2. **Set Expiration Dates**: Avoid lingering roles by setting appropriate expiration dates.
3. **Verify Before Deletion**: Always check `wf_roles` and `wf_user_roles` before removing a role.
4. **Regular Cleanup**: Use `WF_PURGE.AdHocDirectory()` to remove expired roles.

---

## **Troubleshooting**
- **Issue**: Role not found in `wf_roles`.  
  **Solution**: Verify the role name and check for typos.
  
- **Issue**: Users not added/removed.  
  **Solution**: Ensure usernames are in uppercase and correctly formatted.