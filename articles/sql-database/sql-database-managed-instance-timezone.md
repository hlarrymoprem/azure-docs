---
title: Azure SQL Database Managed Instance Time Zone | Microsoft Docs"
description: Learn about time zone specifics of Azure SQL Database Managed Instance
services: sql-database
ms.service: sql-database
ms.custom: 
ms.devlang: 
ms.topic: conceptual
author: MladjoA
ms.author: mlandzic
ms.reviewer: 
manager: craigg
ms.date: 04/09/2019
---
# Time Zone in Azure SQL Database Managed Instance

While using Coordinated Universal Time (UTC) is a recommended practice for data tier of cloud solutions, Azure SQL Database Managed Instance offers a choice of time zone to meet the needs of the existing applications that store date and time values and call date and time functions with an implicit context of a specific time zone.

T-SQL functions like [GETDATE()](https://docs.microsoft.com/sql/t-sql/functions/getdate-transact-sql) or CLR code observe the time zone set on the instance level. SQL Agent jobs also follow schedule according to the time zone of the instance.

  >[!NOTE]
  > Managed Instance is the only deployment option of Azure SQL Database that supports time zone setting. Other deployment options always follow UTC.
Use [AT TIME ZONE](https://docs.microsoft.com/sql/t-sql/queries/at-time-zone-transact-sql) in single and pooled SQL databases if you need to interpret date and time information in non-UTC time zone.

## Supported time zones

A set of supported time zones is inherited from the underlying operating system of the managed instance and it is being regularly updated to get new time zone definitions and reflect changes to the existing ones.

A list with names of the supported time zones is exposed through the [sys.time_zone_info](https://docs.microsoft.com/sql/relational-databases/system-catalog-views/sys-time-zone-info-transact-sql) system view.

## Setting time zone

A time zone of managed instance can be set during instance creation only. The default time zone is Coordinated Universal Time (UTC).

  >[!NOTE]
  > The time zone of an existing managed instance cannot be changed.

### Setting the time zone through Azure portal

While entering parameters for the new instance, select a time zone from the list of supported time zones:
  
![Setting Time Zone during instance creation](media/sql-database-managed-instance-timezone/01-setting_timezone-during-instance-creation.png)

### Azure Resource Manager template

Specify timezoneId property in your [Resource Manager template](https://aka.ms/sql-mi-create-arm-posh) to set the time zone during instance creation.

```json
"properties": {
                "administratorLogin": "[parameters('user')]",
                "administratorLoginPassword": "[parameters('pwd')]",
                "subnetId": "[parameters('subnetId')]",
                "storageSizeInGB": 256,
                "vCores": 8,
                "licenseType": "LicenseIncluded",
                "hardwareFamily": "Gen5",
                "collation": "Serbian_Cyrillic_100_CS_AS",
                "timezoneId": "Central European Standard Time"
            },

```

List of supported values for timezoneId property can be found at the end of this article.

If not specified, time zone will be set to UTC.

## Checking the time zone of instance

[CURRENT_TIMEZONE](https://docs.microsoft.com/sql/t-sql/functions/current-timestamp-transact-sql) function returns a display name of the time zone of the instance.

## Cross-feature considerations

### Restore and import

You can restore backup file or import data to managed instance from an instance or a server with different time zone settings. However, make sure to do so with caution and to analyze the application behavior and the results of the queries and reports, just like when transferring data between two SQL Server instances with different time zone settings.

### Point-in-time restore

When performing point-in-time restore, the time to restore to is interpreted as UTC time to avoid any ambiguity due to daylight saving time and its potential changes.

### Auto-failover groups

Using the same time zone across primary and secondary instance in failover group is not enforced, but it is strongly recommended.
  >[!IMPORTANT]
  > While there are valid scenarios for having different time zone on geo-secondary instance used for read scale only, please note that in the case of manual or automatic failover to secondary instance it will retain its original time zone.

## Limitations

- Time zone of the existing managed instance cannot be changed.
- External processes launched from the SQL Agent jobs do not observe time zone of the instance.
- Managed Instance’s native [New-AzSqlInstance](https://docs.microsoft.com/powershell/module/az.sql/new-azsqlinstance) PowerShell cmdlet does not support passing time zone parameter yet. Use PowerShell wrapper with [Resource Manager template](https://aka.ms/sql-mi-create-arm-posh) instead.
- CLI command [az sql mi create](https://docs.microsoft.com/cli/azure/sql/mi?view=azure-cli-latest#az-sql-mi-create) does not support time zone parameter yet.

## List of supported time zones

| **Time Zone ID** | **Time Zone Display Name** |
| --- | --- |
| Dateline Standard Time | (UTC-12:00) International Date Line West |
| UTC-11 | (UTC-11:00) Coordinated Universal Time-11 |
| Aleutian Standard Time | (UTC-10:00) Aleutian Islands |
| Hawaiian Standard Time | (UTC-10:00) Hawaii |
| Marquesas Standard Time | (UTC-09:30) Marquesas Islands |
| Alaskan Standard Time | (UTC-09:00) Alaska |
| UTC-09 | (UTC-09:00) Coordinated Universal Time-09 |
| Pacific Standard Time (Mexico) | (UTC-08:00) Baja California |
| UTC-08 | (UTC-08:00) Coordinated Universal Time-08 |
| Pacific Standard Time | (UTC-08:00) Pacific Time (US & Canada) |
| US Mountain Standard Time | (UTC-07:00) Arizona |
| Mountain Standard Time (Mexico) | (UTC-07:00) Chihuahua, La Paz, Mazatlan |
| Mountain Standard Time | (UTC-07:00) Mountain Time (US & Canada) |
| Central America Standard Time | (UTC-06:00) Central America |
| Central Standard Time | (UTC-06:00) Central Time (US & Canada) |
| Easter Island Standard Time | (UTC-06:00) Easter Island |
| Central Standard Time (Mexico) | (UTC-06:00) Guadalajara, Mexico City, Monterrey |
| Canada Central Standard Time | (UTC-06:00) Saskatchewan |
| SA Pacific Standard Time | (UTC-05:00) Bogota, Lima, Quito, Rio Branco |
| Eastern Standard Time (Mexico) | (UTC-05:00) Chetumal |
| Eastern Standard Time | (UTC-05:00) Eastern Time (US & Canada) |
| Haiti Standard Time | (UTC-05:00) Haiti |
| Cuba Standard Time | (UTC-05:00) Havana |
| US Eastern Standard Time | (UTC-05:00) Indiana (East) |
| Turks And Caicos Standard Time | (UTC-05:00) Turks and Caicos |
| Paraguay Standard Time | (UTC-04:00) Asuncion |
| Atlantic Standard Time | (UTC-04:00) Atlantic Time (Canada) |
| Venezuela Standard Time | (UTC-04:00) Caracas |
| Central Brazilian Standard Time | (UTC-04:00) Cuiaba |
| SA Western Standard Time | (UTC-04:00) Georgetown, La Paz, Manaus, San Juan |
| Pacific SA Standard Time | (UTC-04:00) Santiago |
| Newfoundland Standard Time | (UTC-03:30) Newfoundland |
| Tocantins Standard Time | (UTC-03:00) Araguaina |
| E. South America Standard Time | (UTC-03:00) Brasilia |
| SA Eastern Standard Time | (UTC-03:00) Cayenne, Fortaleza |
| Argentina Standard Time | (UTC-03:00) City of Buenos Aires |
| Greenland Standard Time | (UTC-03:00) Greenland |
| Montevideo Standard Time | (UTC-03:00) Montevideo |
| Magallanes Standard Time | (UTC-03:00) Punta Arenas |
| Saint Pierre Standard Time | (UTC-03:00) Saint Pierre and Miquelon |
| Bahia Standard Time | (UTC-03:00) Salvador |
| UTC-02 | (UTC-02:00) Coordinated Universal Time-02 |
| Mid-Atlantic Standard Time | (UTC-02:00) Mid-Atlantic - Old |
| Azores Standard Time | (UTC-01:00) Azores |
| Cape Verde Standard Time | (UTC-01:00) Cabo Verde Is. |
| UTC | (UTC) Coordinated Universal Time |
| GMT Standard Time | (UTC+00:00) Dublin, Edinburgh, Lisbon, London |
| Greenwich Standard Time | (UTC+00:00) Monrovia, Reykjavik |
| W. Europe Standard Time | (UTC+01:00) Amsterdam, Berlin, Bern, Rome, Stockholm, Vienna |
| Central Europe Standard Time | (UTC+01:00) Belgrade, Bratislava, Budapest, Ljubljana, Prague |
| Romance Standard Time | (UTC+01:00) Brussels, Copenhagen, Madrid, Paris |
| Morocco Standard Time | (UTC+01:00) Casablanca |
| Sao Tome Standard Time | (UTC+01:00) Sao Tome |
| Central European Standard Time | (UTC+01:00) Sarajevo, Skopje, Warsaw, Zagreb |
| W. Central Africa Standard Time | (UTC+01:00) West Central Africa |
| Jordan Standard Time | (UTC+02:00) Amman |
| GTB Standard Time | (UTC+02:00) Athens, Bucharest |
| Middle East Standard Time | (UTC+02:00) Beirut |
| Egypt Standard Time | (UTC+02:00) Cairo |
| E. Europe Standard Time | (UTC+02:00) Chisinau |
| Syria Standard Time | (UTC+02:00) Damascus |
| West Bank Standard Time | (UTC+02:00) Gaza, Hebron |
| South Africa Standard Time | (UTC+02:00) Harare, Pretoria |
| FLE Standard Time | (UTC+02:00) Helsinki, Kyiv, Riga, Sofia, Tallinn, Vilnius |
| Israel Standard Time | (UTC+02:00) Jerusalem |
| Kaliningrad Standard Time | (UTC+02:00) Kaliningrad |
| Sudan Standard Time | (UTC+02:00) Khartoum |
| Libya Standard Time | (UTC+02:00) Tripoli |
| Namibia Standard Time | (UTC+02:00) Windhoek |
| Arabic Standard Time | (UTC+03:00) Baghdad |
| Turkey Standard Time | (UTC+03:00) Istanbul |
| Arab Standard Time | (UTC+03:00) Kuwait, Riyadh |
| Belarus Standard Time | (UTC+03:00) Minsk |
| Russian Standard Time | (UTC+03:00) Moscow, St. Petersburg |
| E. Africa Standard Time | (UTC+03:00) Nairobi |
| Iran Standard Time | (UTC+03:30) Tehran |
| Arabian Standard Time | (UTC+04:00) Abu Dhabi, Muscat |
| Astrakhan Standard Time | (UTC+04:00) Astrakhan, Ulyanovsk |
| Azerbaijan Standard Time | (UTC+04:00) Baku |
| Russia Time Zone 3 | (UTC+04:00) Izhevsk, Samara |
| Mauritius Standard Time | (UTC+04:00) Port Louis |
| Saratov Standard Time | (UTC+04:00) Saratov |
| Georgian Standard Time | (UTC+04:00) Tbilisi |
| Volgograd Standard Time | (UTC+04:00) Volgograd |
| Caucasus Standard Time | (UTC+04:00) Yerevan |
| Afghanistan Standard Time | (UTC+04:30) Kabul |
| West Asia Standard Time | (UTC+05:00) Ashgabat, Tashkent |
| Ekaterinburg Standard Time | (UTC+05:00) Ekaterinburg |
| Pakistan Standard Time | (UTC+05:00) Islamabad, Karachi |
| India Standard Time | (UTC+05:30) Chennai, Kolkata, Mumbai, New Delhi |
| Sri Lanka Standard Time | (UTC+05:30) Sri Jayawardenepura |
| Nepal Standard Time | (UTC+05:45) Kathmandu |
| Central Asia Standard Time | (UTC+06:00) Astana |
| Bangladesh Standard Time | (UTC+06:00) Dhaka |
| Omsk Standard Time | (UTC+06:00) Omsk |
| Myanmar Standard Time | (UTC+06:30) Yangon (Rangoon) |
| SE Asia Standard Time | (UTC+07:00) Bangkok, Hanoi, Jakarta |
| Altai Standard Time | (UTC+07:00) Barnaul, Gorno-Altaysk |
| W. Mongolia Standard Time | (UTC+07:00) Hovd |
| North Asia Standard Time | (UTC+07:00) Krasnoyarsk |
| N. Central Asia Standard Time | (UTC+07:00) Novosibirsk |
| Tomsk Standard Time | (UTC+07:00) Tomsk |
| China Standard Time | (UTC+08:00) Beijing, Chongqing, Hong Kong, Urumqi |
| North Asia East Standard Time | (UTC+08:00) Irkutsk |
| Singapore Standard Time | (UTC+08:00) Kuala Lumpur, Singapore |
| W. Australia Standard Time | (UTC+08:00) Perth |
| Taipei Standard Time | (UTC+08:00) Taipei |
| Ulaanbaatar Standard Time | (UTC+08:00) Ulaanbaatar |
| Aus Central W. Standard Time | (UTC+08:45) Eucla |
| Transbaikal Standard Time | (UTC+09:00) Chita |
| Tokyo Standard Time | (UTC+09:00) Osaka, Sapporo, Tokyo |
| North Korea Standard Time | (UTC+09:00) Pyongyang |
| Korea Standard Time | (UTC+09:00) Seoul |
| Yakutsk Standard Time | (UTC+09:00) Yakutsk |
| Cen. Australia Standard Time | (UTC+09:30) Adelaide |
| AUS Central Standard Time | (UTC+09:30) Darwin |
| E. Australia Standard Time | (UTC+10:00) Brisbane |
| AUS Eastern Standard Time | (UTC+10:00) Canberra, Melbourne, Sydney |
| West Pacific Standard Time | (UTC+10:00) Guam, Port Moresby |
| Tasmania Standard Time | (UTC+10:00) Hobart |
| Vladivostok Standard Time | (UTC+10:00) Vladivostok |
| Lord Howe Standard Time | (UTC+10:30) Lord Howe Island |
| Bougainville Standard Time | (UTC+11:00) Bougainville Island |
| Russia Time Zone 10 | (UTC+11:00) Chokurdakh |
| Magadan Standard Time | (UTC+11:00) Magadan |
| Norfolk Standard Time | (UTC+11:00) Norfolk Island |
| Sakhalin Standard Time | (UTC+11:00) Sakhalin |
| Central Pacific Standard Time | (UTC+11:00) Solomon Is., New Caledonia |
| Russia Time Zone 11 | (UTC+12:00) Anadyr, Petropavlovsk-Kamchatsky |
| New Zealand Standard Time | (UTC+12:00) Auckland, Wellington |
| UTC+12 | (UTC+12:00) Coordinated Universal Time+12 |
| Fiji Standard Time | (UTC+12:00) Fiji |
| Kamchatka Standard Time | (UTC+12:00) Petropavlovsk-Kamchatsky - Old |
| Chatham Islands Standard Time | (UTC+12:45) Chatham Islands |
| UTC+13 | (UTC+13:00) Coordinated Universal Time+13 |
| Tonga Standard Time | (UTC+13:00) Nuku'alofa |
| Samoa Standard Time | (UTC+13:00) Samoa |
| Line Islands Standard Time | (UTC+14:00) Kiritimati Island |

## See also

- [CURRENT_TIMEZONE (Transact-SQL)](https://docs.microsoft.com/sql/t-sql/functions/current-timezone-transact-sql)
- [AT TIME ZONE (Transact-SQL)](https://docs.microsoft.com/sql/t-sql/queries/at-time-zone-transact-sql)
- [sys.time_zone_info (Transact-SQL)](https://docs.microsoft.com/sql/relational-databases/system-catalog-views/sys-time-zone-info-transact-sql)