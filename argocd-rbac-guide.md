# ArgoCD User Management and RBAC Guide

This document outlines the steps to create and manage local users and assign Role-Based Access Control (RBAC) policies in ArgoCD. 

In this guide, we will create two users:
* `devopsadmin`: Assigned the built-in **admin** role.
* `developer`: Assigned the built-in **read-only** role.

## Overview

ArgoCD user management and access control are primarily managed through two Kubernetes ConfigMaps in the `argocd` namespace:
1.  **`argocd-cm`**: Used to define new local users and their capabilities (e.g., UI login, API key access).
2.  **`argocd-rbac-cm`**: Used to define RBAC policies, mapping users to specific roles and permissions.

---

## Prerequisites

* `kubectl` access to the cluster where ArgoCD is installed.
* The `argocd` CLI tool installed locally.
* The initial ArgoCD `admin` password.

---

## Step 1: Define New Users (`argocd-cm`)

First, we need to create the user accounts and grant them the ability to log in to the UI and generate API keys. We do this by patching the `argocd-cm` ConfigMap.

Run the following `kubectl` commands:

### Define the 'devopsadmin' user
```
kubectl patch configmap argocd-cm -n argocd --type merge -p \
'{"data":{"accounts.devopsadmin": "apiKey, login"}}'
```

### Define the 'developer' user
```
kubectl patch configmap argocd-cm -n argocd --type merge -p \
'{"data":{"accounts.developer": "apiKey, login"}}'
```
Note: ArgoCD automatically detects changes to this ConfigMap; no pod restarts are required.

## Step 2: Assign Roles (`argocd-rbac-cm`)

By default, new users have no permissions. We must explicitly map our newly created users to ArgoCD's built-in roles (role:admin and role:readonly) using the policy.csv key in the argocd-rbac-cm ConfigMap.

Run the following command to patch the RBAC ConfigMap:
```
kubectl patch configmap argocd-rbac-cm -n argocd --type merge -p \
'{"data":{"policy.csv": "g, devopsadmin, role:admin\ng, developer, role:readonly"}}'
```
Understanding the Policy Format
The policy.csv entries follow the format: g, <user_or_group>, <role>

g, devopsadmin, role:admin: Grants all administrative privileges to devopsadmin.

g, developer, role:readonly: Grants view-only access to developer.

## Step 3: Set Initial Passwords
Newly created users do not have a password by default and cannot log in until one is set. You must use the argocd CLI to set these passwords.

### Log in to ArgoCD as the built-in admin:
```
argocd login <ARGOCD_SERVER> --username admin --password <YOUR_ADMIN_PASSWORD>

Example: argocd login 172.31.252.59:32632 --insecure
```
### Update the password for devopsadmin:
```
argocd account update-password \
  --account devopsadmin \
  --new-password <NEW_DEVOPS_PASSWORD> \
  --current-password <YOUR_ADMIN_PASSWORD>
  ```
### Update the password for developer:
```
argocd account update-password \
  --account developer \
  --new-password <NEW_DEVELOPER_PASSWORD> \
  --current-password <YOUR_ADMIN_PASSWORD>
```
## Step 4: Verification
To verify that the accounts have been created and the capabilities are correct, run the following CLI command:
```
argocd account list

NAME         ENABLED  CAPABILITIES
admin        true     login
devopsadmin  true     apiKey, login
developer    true     apiKey, login
```
The new users can now log in to the ArgoCD web UI or CLI using their respective credentials, and they will be restricted to the permissions defined in their assigned roles.

## Step 5: Verifying RBAC Rules with the `can-i` Command

Before handing over credentials, it is a best practice to verify that your RBAC policies are working exactly as intended. ArgoCD provides a built-in `can-i` utility under the `admin settings rbac` command to test permissions without needing to log in as each user.

The syntax for testing is:
`argocd admin settings rbac can-i <action> <resource> <project>/<object> --sub <username>`

Here are practical examples to test the permissions of our two new users against a sample application called `kreatekloud-app` in the `default` project.

### Testing `devopsadmin` (role:admin)

The `devopsadmin` user has full administrative rights. We expect all of these commands to return `yes`.

**1. Can the admin sync an application?**
```
argocd admin settings rbac can-i sync applications 'default/kreatekloud-app' --sub devopsadmin
```
Expected Output: yes

**2. Can the admin create a new cluster connection?**
```
argocd admin settings rbac can-i create clusters '*' --sub devopsadmin
```
Expected Output: yes

### Testing developer (role:readonly)
The developer user is restricted to read-only actions. They should be able to view resources but not mutate or sync them.

**1. Can the developer view (get) an application's status?**
```
argocd admin settings rbac can-i get applications 'default/kreatekloud-app' --sub developer
```
Expected Output: yes

**2. Can the developer view logs for an application?**
```
argocd admin settings rbac can-i get logs 'default/kreatekloud-app' --sub developer
```
Expected Output: yes

**3. Can the developer sync (deploy) an application?**

```
argocd admin settings rbac can-i sync applications 'default/kreatekloud-app' --sub developer
```
Expected Output: no

**4. Can the developer add a new Git repository?**
```
argocd admin settings rbac can-i create repositories '*' --sub developer
```
Expected Output: no

Pro-Tip: If you ever get an unexpected no, double-check the argocd-rbac-cm ConfigMap for typos in the policy.csv mapping or ensure the target project explicitly permits the resource you are testing.




