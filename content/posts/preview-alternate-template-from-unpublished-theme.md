+++
title = "Preview alternate template from unpublished theme"
date = "2020-09-16"
author = "Glauber Magalhães"
cover = ""
tags = ["Shopify Liquid", "Tip"]
slug = "preview-alternate-template-from-unpublished-theme"
description = "How to preview template on a page from an unpublished theme on Shopify"
showFullContent = false
showComments = true
+++


When we create a page in Shopify Admin, there is an option to choose a template that will be used to load that page.
The problem is that you cannot set this option to use a template for a theme that is not published.

So how do you load a page using an unpublished theme template? Follow:


1. Create a new page on the Shopify Admin
1. On your unpublished theme, create a template on templates folder, for example: `templates/page.YOUR_TEMPLATE_SLUG.liquid`
1. Finally, open a preview of your unpublished theme adding the querystring `view=YOUR_TEMPLATE_SLUG`. For example: https://YOUR_STORE.myshopify.com/pages/YOUR_PAGE?view=YOUR_TEMPLATE_SLUG&preview_theme_id=YOUR_THEME_ID


Hope this information is useful for you since Shopify has not documented this.
