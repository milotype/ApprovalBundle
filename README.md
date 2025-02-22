# ApprovalBundle

A plugin for [Kimai](https://www.kimai.org/) - a timetracking open source tool - to approve timesheets of users on a weekly basis including APIs.

Checkout the [Documentation](./documentation.md) for content information.

Here is a short live demo:

![Example process for Teamleads](./_documentation/ApprovalTeamlead.gif)

## Requirements

- Requires Kamai 2, V1.16.10 or higher

Optional but recommended:
- MetaFields plugin
- LockdownPerUser plugin ([GitHub](https://github.com/kevinpapst/LockdownPerUserBundle)) 

## Features

- Users can send a week for approval (in sequential order)
- User lockdown -> a submitted/approved week can no longer be edited (apart from admins) - there is one lockdown date per user (LockdownPerUserBundle)
- Teamlead/Admin can approve or deny the week
- Overview of approvals, missing approvals and the status
- Mailing options to recall approval tasks if outstanding

## Status

The approval bundle is already working pretty well. Some updates will be done soon, as some functionality and checks are not final yet. Unless these things are implemented the version is below 1.

## Issues

It is highly recommended to use the **same timezone** setting for all users. Furthermore all users should use the **same "Start day of the week"** setting - ideally everybody should use "Monday". Otherwise issues could appear as, e.g. Monday times can be located on a Sunday when the teamlead and the user using different timezones. Furthermore the "Start day" is used to store the approval week. When the "Start day" is Sunday for a user and Monday for the teamlead, the approval will not work appropriately.

## Installation

First unzip the plugin into to your Kimai `plugins` directory:

```bash
unzip ApprovalBundle-x.x.zip -d <kimai path>/var/plugins/
```

And then reload Kimai and install all migrations:

```bash
bin/console kimai:reload
bin/console kimai:bundle:approval:install
```

The plugin should appear now.

## Settings

### Meta-Field Setup (optional)

The ApprovalBundle needs some meta fields and settings to be done. The daily and workly hours are displayed. For this the daily working time per day needs to be specified per user. Typically it might be 8h per week day. But there are very different situations, so someone might only work 4 days a week or less hours a day.

The following meta-fields can to be created ([Custom-Field-Plugin](https://www.kimai.org/store/custom-fields-bundle.html) is required for this):

- Custom-Fields -> Users
- The following fields must be from type = "duration", required field, visible, Role = "ROLE_SUPER_ADMIN", default for most should be default = "08:00", for Saturday/Sunday it should be "00:00" - the names could be anything, but the meaning is according those descriptions
  - Daily working time Monday (daily_working_time_monday)
  - Daily working time Tuesday (daily_working_time_tuesday)
  - Daily working time Wednesday (daily_working_time_wednesday)
  - Daily working time Thursday (daily_working_time_thursday)
  - Daily working time Friday (daily_working_time_friday)
  - Daily working time Saturday (daily_working_time_saturday)
  - Daily working time Sunday (daily_working_time_sunday)

It might be that the defaults are initially not automatically taken. Then you have to go through the System -> Users -> Select each user -> Settings -> "Save" (to store the defaults).

**Remark LockdownBundle**

The lockdown bundle also comes along with some custom user fields. In case an empty value is not accepted for start of approval timeframe, please enter "0000-01-01 00:00:01". The same you can enter for the other two time-settings.

### Team Setup

Next the teams needs to be setup. The teams define which person approves the time for what user. It is typically a picture of the organization. The teamlead is reponsible to approve times from it's team members. A teamlead can also be a member of a different team and for this has also an approver. The super user can perform approvals for all. It is expected that the teamlead has also the role of the teamlead - otherwise he/she cannot see the approvals.

### Approval Settings

The final approval settings can be done via approval -> settings. Please enter the names of the daily_working_time_(day of the week) in the appropriate fields. A customer for off-days can also be set - then break times are not considered for those. The E-Mail link will be used as prefix to have the mails containing the correct links for approval views. You might want to enter something like `https://kimai.example.de/`. Finally the approval week start date defines a date where the approval workflow should start. All prior unapproved weeks are ignored.

### Role Settings

There are two new roles available for the team approval. The `view_team_approval` ideally should be YES for all but the user. This allows up from the teamlead hierarchy to see the approvals of their team. The `view_all_approval` should either be YES for System-Admin only or for System-Admin and Admin, depending on your schema. 

## APIs

The following APIs are available. You might want to check out the API swagger documentation within Kimai where the API commands can directly be executed as well.

    Kimai -> click your icon -> settings -> API -> click the book icon top right

## Add to approve API

It's possible to "add to approve" the selected week by API.

request method: **POST**

url: `{your url address}/api/add_to_approve?user={user ID}&date={monday of selected week: Y-m-d}`

headers:
```
X-AUTH-USER: login
X-AUTH-TOKEN: token/password
```

response:
- response code 200 - URL, to the selected week "added to approval"
- response 400 - "Approval already exists" / "User not from your team"  / "Please add previous weeks to approve"
- response 403 - by bad authentication header
- response 404 - wrong user

Admin can "add to approve" all users.
Teamlead can "add to approve" only users from his team.
Normal users can "add to approve" only their own.

## Week status API

It's possible to check status of selected week

request method: **GET**

url: `{your url address}/api/week-status?user={user ID}&date={monday of selected week: Y-m-d}`

headers:
```
X-AUTH-USER: login
X-AUTH-TOKEN: token/password
```

response:
- response code 200 - information about status
- response 400 - "Access denied" / "User not from your team"
- response 403 - by bad authentication header
- response 404 - wrong user

Admin can check the status of all users
Teamlead can check the status of his team users
Normal users can check their status

## Next week API

It's possible to check which week can be currently submitted

request method: **GET**

url: `{your url address}/api/next-week?user={user ID}`

headers:
```
X-AUTH-USER: login
X-AUTH-TOKEN: token/password
```

response:

- response code 200 - information about the week
- response 403 - by bad authentication header
- response 404 - wrong user / no data

## Cronjobs

Cronjobs can be setup to activate mailings with respect to outstanding approval processes. The following commands are available:

All commands are run with the command:
`bin/console kimai:bundle:approval:{{ command from table }}`
Command send lists of users (without system-admin and disabled users)

E.g.
`bin/console kimai:bundle:approval:admin-not-submitted-users`


|   Command                             |   Email to:                                                   |   Contents                                                                                                                    |
|---------------------------------------|---------------------------------------------------------------|-------------------------------------------------------------------------------------------------------------------------------|
|   admin-not-submitted-users           |   System-Admin                                                |   List of all users with his 'not submitted' weeks. Command send lists of users (without system-admin and disabled users).    |
|   teamlead-not-submitted-last-week    |   Active team-leaders                                         |   List of team users with his 'not submitted' weeks. Command send lists of users (without system-admin and disabled users).   |
|   user-not-submitted-weeks            |   All active users (without admins and System-Admin)          |   List of weeks that are 'not submitted'.                                                                                     |

## Contribution

Many thanks go to [HMR-IT](https://www.hmr-it.de) which had been highly involved in this project.

Additional thanks go to Milo Ivir for additional translations and to Kevin Papst for code enhancements and the update to use this bundle with less pre-requisites.