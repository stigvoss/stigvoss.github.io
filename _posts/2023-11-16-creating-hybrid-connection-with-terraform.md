---
layout: post
title:  "Workarounds for creating Azure Hybrid Connections with Terraform"
date:   2023-11-16 15:00:00 +0100
categories: terraform azurerm hybrid-connection
---

Due to some bugs in the current [Terraform azurerm provider](https://registry.terraform.io/providers/hashicorp/azurerm/latest) (v3.80.0), certain workarounds are needed to create a working hybrid connection.

The issue is present for both Function Apps and Web Apps.

Following the [official documentation's examples](https://registry.terraform.io/providers/hashicorp/azurerm/latest/docs/resources/function_app_hybrid_connection#example-usage), the following code will not work and will result in a seemingly successful deployment, but the hybrid connection will not work. No connection will be able to be established, but everything looks fine in the Azure Portal.

I have two workaround solutions for this issue.

## Workaround 1 - Run a provisioner after deployment of `azurerm_function_app_hybrid_connection`
```hcl
resource "azurerm_function_app_hybrid_connection" "example" {
  function_app_id = azurerm_linux_function_app.example.id
  relay_id        = azurerm_relay_hybrid_connection.example.id
  hostname        = "server.example.local"
  port            = 443

  send_key_name = azurerm_relay_hybrid_connection_authorization_rule.example.name

  provisioner "local-exec" {
    command = "az functionapp hybrid-connection add --hybrid-connection ${azurerm_relay_hybrid_connection.example.name} --namespace ${azurerm_relay_namespace.example.name} -n ${azurerm_linux_function_app.example.name} -g ${azurerm_resource_group.example.name}"
  }
}
```

The thing to note in this solution is that the provisioner will change the `relay_id` which differs in letter casing from what Terraform expects. This will cause Terraform to see the resource as changed and will recreate the hybrid connection on every run.

## Workaround 2 - Use a `null_resource` and provisioners to create the hybrid connection with Azure CLI
```hcl
resource "null_resource" "function_app_hybrid_connection" {
  triggers = {
    function_app_name = azurerm_linux_function_app.example.name
    resource_group_name  = azurerm_resource_group.example.name
    namespace_name = azurerm_relay_namespace.example.name
    hybrid_connection_name = azurerm_relay_hybrid_connection.example.name
  }

  provisioner "local-exec" {
    command = "az functionapp hybrid-connection add --hybrid-connection ${self.triggers.hybrid_connection_name} --namespace ${self.triggers.namespace_name} -n ${self.triggers.function_app_name} -g ${self.triggers.resource_group_name}"
  }

  provisioner "local-exec" {
    when = destroy
    command = "az functionapp hybrid-connection remove --hybrid-connection ${self.triggers.hybrid_connection_name} --namespace ${self.triggers.namespace_name} -n ${self.triggers.function_app_name} -g ${self.triggers.resource_group_name}"
  }
}
```

This solution will not use the `azurerm_function_app_hybrid_connection` resource at all, but instead use a `null_resource` with provisioners to create the hybrid connection with Azure CLI. The method prevents Terraform from seeing changes to the resource and will not recreate the hybrid connection on every run, but nor will it be able to update the hybrid connection if it is changed in the Azure Portal.

## Remember the Metadata
When creating the Hybrid Connection in the Azure Relay, remember to add the metadata `endpoint` with the value of the hostname and port of the server you want to connect to, otherwise the provisioner will fail and the Azure Portal will not display the information correctly.

```hcl
resource "azurerm_relay_hybrid_connection" "example" {
  name                          = "example"
  resource_group_name           = azurerm_resource_group.example.name
  relay_namespace_name          = azurerm_relay_namespace.example.name
  requires_client_authorization = true

  user_metadata = jsonencode(
  [
    {
      key   = "endpoint",
      value = "server.example.local:443"
    }
  ])
}
```