End of Support
Home Assistant has introduced local calendars in 2022, in the 2023.1 release they added an option for different recurent events. With this, most of the functionality of this custom helper is supported natively. So I will end developing and supporting this helper in 2023.

Table of Contents
Description

Installation

Manual Installation
Installation via Home Assistant Community Store (HACS)
Configuration

Blueprints for Manual Update

Public Holidays
Include and Exclude
Offset
Import TXT
Monthly on a fixed date
State and Attributes

Lovelace configuration examples

Garbage Collection
The garbage_collection component is a Home Assistant helper that creates a custom sensor for monitoring a regular garbage collection schedule. The sensor can be configured for a number of different patterns:

weekly schedule (including multiple collection days, e.g. on Tuesday and Thursday)
every-n-weeks repeats every period of weeks, starting from the week number first_week. It uses the week number - therefore, it restarts each year, as the weeks start from number 1 each year.
bi-weekly in even-weeks or odd-weeks (technically, it is the same as every 2 weeks with 1st or 2nd first_week)
every-n-days (repeats regularly from the given first date). If n is a multiply of 7, it works similar to every-n-weeks, with the difference that it does not use the week numbers (that restart each year) but continues infinitely from the initial date.
monthly schedule (nth weekday each month), or a specific weekday of each nth week. Using the period it could also be every 2nd, 3rd etc month.
annually (e.g. birthdays). This is once per year. Using include dates, you can add additional dates manually.
blank does not automatically schedule any collections - to be used in cases where you want to make completely own schedule with manual_update.
You can also configure seasonal calendars (e.g. for bio-waste collection), by configuring the first and last month. And you can group entities, which will merge multiple schedules into one sensor.

These are some examples using this sensor. The Lovelace config examples are included below.

MANUAL INSTALLATION
Download the latest release.
Unpack the release and copy the custom_components/garbage_collection directory into the custom_components directory of your Home Assistant installation.
Restart Home Assistant.
Configure the garbage_collection helper.
INSTALLATION VIA Home Assistant Community Store (HACS)
Ensure that HACS is installed.
Search for and install the "Garbage Collection" integration.
Restart Home Assistant.
Configure the garbage_collection helper.
Configuration
Go to Settings/Devices & Services/Helpers, click on the + CREATE HELPER button, select Garbage Collection and configure the helper.
If you would like to add more than one collection schedule, click on the + CREATE HELPER button again and add another Garbage Collection helper instance.

The configuration hapend in 2 steps. In the first step, you select the frequency and common parameters. In the second step you configure additional parameters depending on the selected frequency.

The configuration via configuration.yaml has been deprecated. If you have previously configured the integration there, it will be imported to ConfigFlow, and you should remove it.

STEP 1 - Common Parameters
Parameter	Required	Description
Friendly name	Yes	Sensor friendly name
Frequency	Yes	"weekly", "even-weeks", "odd-weeks", "every-n-weeks", "every-n-days", "monthly", "annual", "group" or "blank"
Icon	No	Default icon Default: mdi:trash-can
Icon today	No	Icon if the collection is today Default: mdi:delete-restore
Icon tomorrow	No	Icon if the collection is tomorrow Default: mdi:delete-circle
Expire After	No	Time in format format HH:MM. If the collection is due today, start looking for the next occurence after this time (i.e. if the weekly collection is in the morning, change the state from 'today' to next week in the afternoon)
Verbose state	No	The sensor state will show collection date and remaining days, instead of number.
Obsolete - do the formatting on the dashboard instead.
Default: False
Hidde in calendar	No	Hide in calendar (useful for sensors that are used in groups)
Default: False
Manual update	No	(Advanced). Do not automatically update the status. Status is updated manualy by calling the service garbage_collection.update_state from an automation triggered by event garbage_collection_loaded, that could manually add or remove collection dates, and manually trigger the state update at the end. See the example.
Default: False
Verbose format	No	(relevant when verbose state is True). Verbose status formatting string. Can use placeholders {date} and {days} to show the date of next collection and remaining days.
Obsolete - do the formatting on the dashboard instead.
Default: 'on {date}, in {days} days'
When the collection is today or tomorrow, it will show Today or Tomorrow
(currently in English, French, Czech and Italian).
Date format	No	In the verbose format, you can configure the format of date (using strftime format)
Obsolete - do the formatting on the dashboard instead.
Default: '%d-%b-%Y'
STEP 2 - parameters depending on the selected frequency
...FOR ALL FREQUENCIES EXCEPT ANNUAL, GROUP and BLANK
Parameter	Required	Description
First month	No	Month three letter abbreviation, e.g. "jan", "feb"...
Default: "jan"
Last month	No	Month three letter abbreviation.
Default: "dec"
...FOR ALL FREQUENCIES EXCEPT ANNUAL, EVERY-N-DAYS, GROUP and BLANK
Parameter	Required	Description
Collection days	Yes	Day three letter abbreviation, list of "mon", "tue", "wed", "thu", "fri", "sat", "sun".
...FOR COLLECTION EVERY-N-WEEKS
Parameter	Required	Description
Period	No	Collection every "period" weeks (integer 1-53)
Default: 1
First week	No	First collection on the "first week" week (integer 1-53)
Default: 1
(The week number is using ISO-8601 numeric representation of the week)

Note: This parameter cannot be used to set the beginning of the collection period (use the first month parameter for that). The purpose of first week is to simply 'offset' the week number, so the collection every ;'n' weeks does not always trigger on week numbers that are multiplication of 'n'. Technically, the value of this parameter shall be less than period, otherwise it will give weird results. Also note, that the week numbers restart each year. Use every-n-days frequency if you need a consistent period across the year ends.
...FOR COLLECTION EVERY-N-DAYS
Parameter	Required	Description
First date	Yes	Repeats every n days from this first date
(date in the international ISO format 'yyyy-mm-dd').
Period	No	Collection every "period" days (warning - in this configuration, it is days, not weeks!)
Default: 1 (daily, which makes no sense I suppose)
...FOR MONTHLY COLLECTION
The monthly schedule has two flavors: it can trigger either on the nth occurrence of the weekday in a month, or on the weekday in the nth week of each month.

Parameter	Required	Description
Order of weekday	Yes	List of week numbers of collection day each month. E.g., if collection_day is "sat", 1 will mean 1st Saturday each month (integer 1-5)
Order of week, instead of weekday order	No	CONFIGURE THIS ONE ONLY IF YOU ARE SURE YOU NEED IT. This will alter the behaviour of order of weekday, so that instead of nth weekday of each month, take the weekday of the nth week of each month.
So if the month starts on Friday, the Wednesday of the 1st week would actually be last Wednesday of the previous month and the Wednesday of 2nd week will be the 1st Wednesday of the month. So if you have just randomy clicked on the option, it might appear as if it calculates a wrong date! Yes, this is confusing, but there are apparently some use case for this.
Period	No	If period is not defined (or 1), the schedule will repeat monthly. If period is 2, it will be every 2nd month. If period is 3, it will be once per quarter, and so on.
The first month parameter will then define the starting month. So if the first month is jan (or not defined), and period is 2, the collection will be in odd months (jan, mar, may, jul, sep and nov). If first month is feb, it will be in even months. (integer 1-12)
Default: 1
...FOR ANNUAL COLLECTION
Parameter	Required	Description
Date	Yes	The date of collection, in format 'mm/dd' (e.g. '11/24' for November 24 each year)
...FOR GROUP
Parameter	Required	Description
List of entities	Yes	A list of entity_ids to merge
Blueprints for Manual Update
Prerequisites
To use the blueprints, you need to set the garbage_collection entity for Manual update, that will fire the garbage_collection_loaded event on each sensor update and trigger the automation blueprint.
Install/Import blueprint
From the blueprint, create and configure the automation
Public Holidays
There are a couple of blueprints, automatically moving the collection falling on a public holiday. Or if there was a public holiday in the week before the scheduled collection.

The Public Holidays blueprints use a separate custom helper Holidays, available through HACS, that you can configure for different countries.

Open your Home Assistant instance and show the blueprint import dialog with a specific blueprint pre-filled.	Move the collection to the next day, if the collection falls on public holiday
Open your Home Assistant instance and show the blueprint import dialog with a specific blueprint pre-filled.	Remove events falling on provided "exclude" list of dates. Then check the calendar of public holidays and move events that fall on a public holiday to the next day. Finally, add additional events on dates from "include" list.
Open your Home Assistant instance and show the blueprint import dialog with a specific blueprint pre-filled.	Move forward one day if a public holiday was in the collection week, before or on the collection day (and keep moving if the new collection day also falls on a holiday)
Open your Home Assistant instance and show the blueprint import dialog with a specific blueprint pre-filled.	Move forward by a day for each public holiday was in the collection week, before or on the collection day (and keep moving if the new collection day also falls on a holiday). So if there were two public holidays, move by two days.
Open your Home Assistant instance and show the blueprint import dialog with a specific blueprint pre-filled.	Move forward by one day if there was a public holiday in the collection week, before or on the collection day (and keep moving if the new collection day also falls on a holiday). Only move by one day, but if there was more than one public holiday in the week, carry it over to the following week. So if there were 2 public holidays this week, move it by one day this week and one day next week.
Open your Home Assistant instance and show the blueprint import dialog with a specific blueprint pre-filled.	Skip the holiday
Include and Exclude
A list of fixed dates to include and exclude from the calculated schedule.

Open your Home Assistant instance and show the blueprint import dialog with a specific blueprint pre-filled.	Include and Exclude
Open your Home Assistant instance and show the blueprint import dialog with a specific blueprint pre-filled.	Include
Open your Home Assistant instance and show the blueprint import dialog with a specific blueprint pre-filled.	Exclude
Offset
The offset blueprint will move the calculated collections by a number of days. This can be used, for example, to schedule collection for last Saturday each month - just set the collection to the first Saturday each month and offset it by -7 days.

Open your Home Assistant instance and show the blueprint import dialog with a specific blueprint pre-filled.

Import txt
This blueprint requires a command_line sensor reading content of a txt file, containig a set of dates, one per line.

Open your Home Assistant instance and show the blueprint import dialog with a specific blueprint pre-filled.

command_line sensor example
Monthly on a fixed date
This will create a schedule on a fixed date each month. For example on the 3rd each month. The helper does not allow it, as it is generally designed around paterns evolving around weekly schedules (since garbage collection typically happens on a set day in a week, rather than set day in a month). But few of you wanted that, so here you go.

Open your Home Assistant instance and show the blueprint import dialog with a specific blueprint pre-filled. One fixed date

Open your Home Assistant instance and show the blueprint import dialog with a specific blueprint pre-filled. Two dates

STATE AND ATTRIBUTES
State
The state can be one of

Value	Meaning
0	Collection is today
1	Collection is tomorrow
2	Collection is later
If the verbose_state parameter is set, it will show the date, and the remaining days. For example: "Today" or "Tomorrow" or "on 10-Sep-2019, in 2 days" (configurable)

Attributes
Attribute	Description
next_date	The date of next collection
days	Days till the next collection
last_collection	The date and time of the last collection
Services
garbage_collection.collect_garbage
If the collection is scheduled for today, mark it completed and look for the next collection. It will set the last_collection attribute to the current date and time.

Attribute	Description
entity_id	The garbage collection entity id (e.g. sensor.general_waste)
last_collection	(optional) Set the last collection date to this value. This can be used to re-set the next collection calculation, if the last collection date was set in error. If omitted, it will set the last collection to the current date & time.
Manual update
There are standard blueprints provided to handle manual updates - to move collection on public holidays or offset the collection.

If these blueprints do not work for you, you can create your own custom rules to handle any scenario. If you do so, please share the blueprints with the others by posting them to the blueprints directory - someone else might find them useful. Thanks! To help you with creating custom automations, see the following examples:
![image](https://github.com/ck272003/PROJECT-ON-TENSORFLOW/assets/64009365/316d771b-51d5-47ce-82fe-48e0e784c346)

