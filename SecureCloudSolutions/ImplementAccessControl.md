# Implement Claims-Based Access Control (CBAC) and Role-Based Access Control (RBAC) authorization

## Reading material:
https://docs.microsoft.com/en-us/azure/role-based-access-control/overview

https://docs.microsoft.com/en-us/azure/role-based-access-control/role-assignments-portal

https://docs.microsoft.com/en-us/azure/role-based-access-control/role-definitions

https://docs.microsoft.com/en-us/azure/role-based-access-control/custom-roles

https://stackoverflow.com/questions/22814023/role-based-access-control-rbac-vs-claims-based-access-control-cbac-in-asp-n

https://blogs.msdn.microsoft.com/usisvde/2012/03/13/windows-azure-security-best-practices-part-5-claims-based-identity-single-sign-on/

## Videos


## What is Role-Based Access Control?
RBAC: Role-Based Access Control, an Authorization Manager-based access control paradigm that controls the access to the resources or business process based on role permissions.
Allowing any user / application access to things based on their assigned roles / inherited roles permissions.

## What is Claims-Based Access Control?
CBAC: Claims-Based Access Control, an access control paradigm that uses the claims to make access-control decisions to resources.
Allowing any user / application access to things based on their individual permissions. 

## What is the difference?
A Claim is an attribute that makes an assertion ( in this case a permission ) about a Principle with which it is associated. A Role is a set of permissions assigned to any Principles assigned to said role.

Here is a fundamental example of both permission strategies for an MVC Action showing the difference:

```
// ALL Principles assigned to Sale or Marketing roles can create customers
[Authorize(Roles = "CustomerCreator")]
public ActionResult CreateCustomer()

// ONLY Principles with CanCreateCustomer permissions can create customers
[ClaimAuthorize(Permission="CanCreateCustomer")]
public ActionResult CreateCustomer()
```

Claims as a concept are more generic compared to Roles. All claims are explicitly attached to an identity and include an explicit scope. A Role is a union of Users and Permissions. Claims do not necessarily replace Roles or Permissions, they are additional pieces of information that one can use to make an Authorization decision.

To use the nightclub metaphor, a Driver's License that contains the Date of Birth Claim "authorizes" the visitor to enter the club. This is a decision made on the value of the attribute (i.e DoB = {MM-DD-YYYY}) and the fact that the claim is issued by a trusted authority (i.e. Issued by the Government).

However, the building where the nightclub is might contain offices, rooms, a kitchen, other floors, elevators, a basement, etc. where only employees of the club can enter. Furthermore, certain employees might have access to certain places that other employees may not. For example, a Manager may have access to an office floor above that other employees cannot access. In this case there are two Roles. Manager and Employee.

While visitors' access to the public nightclub area is authorized by a single claim, employees need access by Role. For them, a Driver's License is not enough. What they need is an Employee Badge that they scan to enter doors. Somewhere there is an RBAC system that grants badges in the Manager Role access to the top floor, and badges in the Employee Role access to other rooms.

## Definitions
1. CBAC: Claims-Based Access Control, an access control paradigm that uses the claims to make access-control decisions to resources.
2. RBAC: Role-Based Access Control, an Authorization Manager-based access control paradigm that controls the access to the resources or business process based on role permissions.
3. IAM: Identity and Access Management
4. Role definition: A collection of permissions. It's sometimes just called a role. A role definition lists the operations that can be performed, such as read, write, and delete.
5. Security Principal: An object that represents a user, group, service principal, or managed identity
6. Service Principal: A security identity used by applications or services to access specific Azure resources. You can think of it as a user identity (username and password or certificate) for an application.
7. Managed identity: An identity in Azure Active Directory that is automatically managed by Azure.
8. Scope: The set of resources that access applies to. When you grant access at a parent scope ( Subscription, Resource Group, etc ), those permissions are inherited to the child scopes.
9. System-Defined Principal: This principal represents all users, groups, service principals, and managed identities in an Azure AD directory. It will have a Principal ID of GUID 00000000-0000-0000-0000-000000000000.

## Misc notes
    
* When planning your access control strategy, it's a best practice to grant users the least privilege to get their work done.

### Claims Based Access Control


### Roles Based Access Control
* In RBAC, to grant access, you create a role assignment. A role assignment consists of three elements: security principal, role definition, and scope.

* RBAC is an additive model, so your effective permissions are the addition of your role assignments.

* Scopes that can have roles assigned to them are management group, subscription, resource group, or resource

* To make a user an administrator of an Azure subscription, assign them the Owner role at the subscription scope. The Owner role gives the user full access to all resources in the subscription, including the right to delegate access to others.

* Inherited role assignments cannot be removed. If you need to remove an inherited role assignment, you must do it at the scope where the role assignment was created.

* Azure supports up to 2000 role assignments per subscription.

* When you transfer a subscription to a different tenant, all role assignments are permanently deleted from the source tenant and are not migrated to the target tenant. You must re-create your role assignments in the target tenant.

* When creating or deleting role assignments, it can take up to 30 minutes for changes to take effect. You can force a refresh of your role assignment changes by signing out and signing in. If you are making role assignment changes with REST API calls, you can force a refresh by refreshing your access token.

* To create and remove role assignments, you must have Microsoft.Authorization/roleAssignments/* permission. This permission is granted through the Owner or User Access Administrator roles.

* Deny assignments take precedence over role assignments. Currently, deny assignments are read-only and can only be set by Azure. 

* Management access is not inherited to your data. This separation prevents roles with wildcards (*) from having unrestricted access to your data. For example, if a user has a Reader role on a subscription, then they can view the storage account, but by default they can't view the underlying data.

* Custom roles are stored in an Azure Active Directory (Azure AD) directory and can be shared across subscriptions.

## powershell notes
Grant Access to a resource
```
// Get the ID of your subscription using the Get-AzSubscription
Get-AzSubscription

// Example return
// Name     : Pay-As-You-Go
// Id       : 00000000-0000-0000-0000-000000000000
// TenantId : 22222222-2222-2222-2222-222222222222
// State    : Enabled

// Save the subscription scope in a variable
$subScope = "/subscriptions/00000000-0000-0000-0000-000000000000"

// Assign the Reader role to the user at the subscription scope.
New-AzRoleAssignment -SignInName rbacuser@example.com `
  -RoleDefinitionName "Reader" `
  -Scope $subScope
```

List access
```
Get-AzRoleAssignment -SignInName rbacuser@example.com -Scope $subScope

// Example return
// RoleAssignmentId   : /subscriptions/00000000-0000-0000-0000-000000000000/providers/Microsoft.Authorization/roleAssignments/22222222-2222-2222-2222-222222222222
// Scope              : /subscriptions/00000000-0000-0000-0000-000000000000
// DisplayName        : RBAC Tutorial User
// SignInName         : rbacuser@example.com
// RoleDefinitionName : Reader
// RoleDefinitionId   : acdd72a7-3385-48ef-bd42-f606fba81ae7
// ObjectId           : 11111111-1111-1111-1111-111111111111
// ObjectType         : User
// CanDelegate        : False

// To verify the access for the resource group, use the Get-AzRoleAssignment command to list the role assignments.
Get-AzRoleAssignment -SignInName rbacuser@example.com -ResourceGroupName "rbac-tutorial-resource-group"

// Example return
// RoleAssignmentId   : /subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/rbac-tutorial-resource-group/providers/Microsoft.Authorization/roleAssignments/33333333-3333-3333-3333-333333333// 333
// Scope              : /subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/rbac-tutorial-resource-group
// DisplayName        : RBAC Tutorial User
// SignInName         : rbacuser@example.com
// RoleDefinitionName : Contributor
// RoleDefinitionId   : b24988ac-6180-42a0-ab88-20f7382dd24c
// ObjectId           : 11111111-1111-1111-1111-111111111111
// ObjectType         : User
// CanDelegate        : False
```
