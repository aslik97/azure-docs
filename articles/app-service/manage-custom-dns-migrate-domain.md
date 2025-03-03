---
title: Migrate an active DNS name
description: Learn how to migrate a custom DNS domain name that is already assigned to a live site to Azure App Service without any downtime.
tags: top-support-issue

ms.assetid: 10da5b8a-1823-41a3-a2ff-a0717c2b5c2d
ms.topic: article
ms.date: 08/25/2020
ms.custom: seodec18

---
# Migrate an active DNS name to Azure App Service

This article shows you how to migrate an active DNS name to [Azure App Service](../app-service/overview.md) without any downtime.

When you migrate a live site and its DNS domain name to App Service, that DNS name is already serving live traffic. You can avoid downtime in DNS resolution during the migration by binding the active DNS name to your App Service app preemptively.

If you're not worried about downtime in DNS resolution, see [Map an existing custom DNS name to Azure App Service](app-service-web-tutorial-custom-domain.md).

## Prerequisites

To complete this how-to, [make sure that your App Service app is not in FREE tier](manage-scale-up.md#scale-up-your-pricing-tier).

## Bind the domain name preemptively

When you bind a custom domain preemptively, you accomplish both of the following before making any changes to
your existing DNS records:

- Verify domain ownership
- Enable the domain name for your app

When you finally migrate your custom DNS name from the old site to the App Service app, there will be no downtime in DNS resolution.

[!INCLUDE [Access DNS records with domain provider](../../includes/app-service-web-access-dns-records.md)]

### Get domain verification ID

Get the domain verification ID for you app by following the steps at [Get domain verification ID](app-service-web-tutorial-custom-domain.md#2-get-a-domain-verification-id).

### Create domain verification record

To verify domain ownership, add a TXT record for domain verification. The hostname for the TXT record depends on the type of DNS record type you want to map. See the following table (`@` typically represents the root domain):

| DNS record example | TXT Host | TXT Value |
| - | - | - |
| \@ (root) | _asuid_ | [Domain verification ID for your app](app-service-web-tutorial-custom-domain.md#2-get-a-domain-verification-id) |
| www (sub) | _asuid.www_ | [Domain verification ID for your app](app-service-web-tutorial-custom-domain.md#2-get-a-domain-verification-id) |
| \* (wildcard) | _asuid_ | [Domain verification ID for your app](app-service-web-tutorial-custom-domain.md#2-get-a-domain-verification-id) |

In your DNS records page, note the record type of the DNS name you want to migrate. App Service supports mappings from CNAME and A records.

> [!NOTE]
> Wildcard `*` records won't validate subdomains with an existing CNAME's record. You may need to explicitly create a TXT record for each subdomain.

### Enable the domain for your app

1. In the [Azure portal](https://portal.azure.com), in the left navigation of the app page, select **Custom domains**. 

    ![Custom domain menu](./media/app-service-web-tutorial-custom-domain/custom-domain-menu.png)

1. In the **Custom domains** page, select **Add custom domain**.

    ![Add host name](./media/app-service-web-tutorial-custom-domain/add-host-name-cname.png)

1. Type the fully qualified domain name you want to migrate, that corresponds to the TXT record you create, such as `contoso.com`, `www.contoso.com`, or `*.contoso.com`. Select **Validate**.

    The **Add custom domain** button is activated. 

1. Make sure that **Hostname record type** is set to the DNS record type you want to migrate. Select **Add hostname**.

    ![Add DNS name to the app](./media/app-service-web-tutorial-custom-domain/validate-domain-name-cname.png)

    It might take some time for the new hostname to be reflected in the app's **Custom domains** page. Try refreshing the browser to update the data.

    ![CNAME record added](./media/app-service-web-tutorial-custom-domain/cname-record-added.png)

    Your custom DNS name is now enabled in your Azure app. 

## Remap the active DNS name

The only thing left to do is remapping your active DNS record to point to App Service. Right now, it still points to your old site.

<a name="info"></a>

### Copy the app's IP address (A record only)

If you are remapping a CNAME record, skip this section. 

To remap an A record, you need the App Service app's external IP address, which is shown in the **Custom domains** page.

In the **Custom domains** page, copy the app's IP address.

![Portal navigation to Azure app](./media/app-service-web-tutorial-custom-domain/mapping-information.png)

### Update the DNS record

Back in the DNS records page of your domain provider, select the DNS record to remap.

For the `contoso.com` root domain example, remap the A or CNAME record like the examples in the following table: 

| FQDN example | Record type | Host | Value |
| - | - | - | - |
| contoso.com (root) | A | `@` | IP address from [Copy the app's IP address](#info) |
| www\.contoso.com (sub) | CNAME | `www` | _&lt;appname>.azurewebsites.net_ |
| \*.contoso.com (wildcard) | CNAME | _\*_ | _&lt;appname>.azurewebsites.net_ |

Save your settings.

DNS queries should start resolving to your App Service app immediately after DNS propagation happens.

## Migrate domain from another app

You can migrate an active custom domain in Azure, between subscriptions or within the same subscription. However, such a migration without downtime requires the source app and the target app are assigned the same custom domain at a certain time. Therefore, you need to make sure that the two apps are not deployed to the same deployment unit (internally known as a webspace). A domain name can be assigned to only one app in each deployment unit.

You can find the deployment unit for your app by looking at the domain name of the FTP/S URL `<deployment-unit>.ftp.azurewebsites.windows.net`. Check and make sure the deployment unit is different between the source app and the target app. The deployment unit of an app is determined by the [App Service plan](overview-hosting-plans.md) it's in. It's selected randomly by Azure when you create the plan and can't be changed. Azure only makes sure two plans are in the same deployment unit when you [create them in the same resource group *and* the same region](app-service-plan-manage.md#create-an-app-service-plan), but it doesn't have any logic to make sure plans are in different deployment units. The only way for you to create a plan in a different deployment unit is to keep creating a plan in a new resource group or region until you get a different deployment unit.

## Next steps

Learn how to bind a custom TLS/SSL certificate to App Service.

> [!div class="nextstepaction"]
> [Secure a custom DNS name with a TLS binding in Azure App Service](configure-ssl-bindings.md)
