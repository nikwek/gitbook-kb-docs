---
layout: default
category: how-it-works
title: Access control
order: 55
---

# Access control

Papertrail offers the ability to grant members selective access to groups of log
senders within the same Papertrail organization, as well as specific
permissions for managing users, changing plans, and purging logs.

# Where can I access this?

The [Members area](https://papertrailapp.com/account/members) of the account Settings
lists all of the members within the Papertrail organization. From there, any member can be
modified or removed.

![](/assets/images/permissions-members-area.png){: width="935"}

# Which permissions are available?

Here's what these different member settings mean:

* **Manage users and permissions** — full access to everything; the highest permission level a member can have.
* **Change plans and payment** — upgrade or downgrade the organization's plan, modify credit card information, and see previous payments.
* **Full Access** — view logs, modify group details, save searches, and create alerts.
* **Read-only** — view logs only, **cannot** modify group details, save searches, or create alerts.
* **Purge logs** — purge searchable logs.
* **Specific group access** — provides access to only specific log groups instead of all groups. Any group that is not specified will not be accessible at all, even as read-only.

## How do permissions interact?

Many permissions are independent. For example, a user can have billing access only with the **Change plans and payment** option, an alternative to adding individuals to [billing emails](https://papertrailapp.com/account/purchases). There are a few interdependent settings:

* **Manage users and permissions** implies full access to all functions, as mentioned above.
* Only members with billing access or full access to all log groups can subscribe to usage and billing emails, because other members don't have enough access to act on usage or billing information.
* Only members with full access to all groups can purge logs.

# How do I modify permissions?

**Required permission**: "Manage users and permissions"

Click **Edit** in the upper right next to the member's email, then make the desired changes:

![](/assets/images/permissions-how-to.gif)

## Can I edit a lot of members at once?

Sure! In the [Members area](https://papertrailapp.com/account/members), click
**Edit All** at the top of the page.

![](/assets/images/permissions-edit-all.png){: width="924"}

## Can I edit my own permissions?

No. Papertrail doesn't allow changing your own access permissions because it helps prevent edge
cases like an account being left without a member who has full access.

That said, it _is_ possible to edit email notification preferences using the
[Members area](https://papertrailapp.com/account/members) (this can also be done from your
[Profile](https://papertrailapp.com/account/profile)).

![](/assets/images/permissions-self-edit.gif)

# Finding a member

For organizations with a lot of members: when looking for a specific person, the list of members
can be filtered by typing the member's email:

![](/assets/images/permissions-filtering.gif)

If members are filtered, Papertrail will return the focus to the filter input after saving,
in an effort to speed up the flow of modifying additional members.

# Does this affect multiple organizations?

_Learn more about using Papertrail with [multiple organizations](/kb/how-it-works/managing-logs-from-multiple-companies/)._

Not at all. The permissions Papertrail offers are specific to a single organization, and each organization
is free to choose the workflow that makes the most sense, whether that's permissions,
[multiple organizations](/kb/how-it-works/managing-logs-from-multiple-companies/), or a combination of both.
