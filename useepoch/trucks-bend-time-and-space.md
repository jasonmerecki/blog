# Trucks Bend Time And Space!
*Or, why time is hard in computer science.*

It is surprisingly easy for computer programmers to make mistakes with time calculations and time storage in computer programming. Time math requires transformations that people can do in their head without realizing. I will show some of the pitfalls that developers can make and hopefully help some avoid them. 

These examples will show formatted time in "YYYY-MM-DD HH:SS" and use 24-hour time.


## Gort goes to work

A computer engineer named Gort has been hired to write software for a package delivery company. He is used to thinking about time from a wall-clock time, and chooses to store the date and time values in a wall-clock way when he reads it. 

The client will have package pickup and dropoff times around North America.  The incoming schedule data Gort receives looks like this:

| Pickup time |  |
|--|--|
| 2020-10-31 08:00|  |
| 2020-10-31 08:30|  |
| 2020-10-31 09:00|  |

Gort follows his client's request and sorted the pickup times sorted from earliest to latest, so they can coordinate their drivers. Then Gord adds the location to the output, and gets this result:

| Pickup time | Location |
|--|--|
| 2020-10-31 08:00| Chicago, IL |
| 2020-10-31 08:30| San Francisco, CA |
| 2020-10-31 09:00| Dallas, TX |

The client is upset. The pickup in San Francisco is in Pacific time zone, and 08:30 in that location is after the 09:00 pickup in Dallas. But sorting the Wall-Clock time only knows the Wall-Clock time for comparison. 

## First, some definitions

**Wall-Clock time:** The way people are used to measuring time through the day. When people view "11:30 AM" on the wall clock they conclude it's mid-day and time for lunch, while viewing "11:00 PM" indicates it is night time and possibly time to sleep. 

**Time Zone:** A geographic area where people have agreed to set their calendars and clocks in the same way.  Most use the Gregorian calendar, and set their clocks so that "11:30 AM" is roughly the middle of daylight hours. At certain times of the year, people in the Time Zone may or may not change their clocks by 1 hour, to follow Daylight Saving Time (or British Summer Time if you prefer). 

The best lookup source to associate a Time Zone with a location is the IANA Time Zone database.  A good example of an actual Time Zone map by this definition is here: [http://efele.net/maps/tz/us/](http://efele.net/maps/tz/us/).

**UTC Time:** Computers measure time by the elapsed milliseconds starting from a fixed point in time, called the Epoch, which is based on January 1, 1970, 12:00 AM in Greenwich Mean Time. The resulting value is the Coordinated Universal Time (UTC). For example, on Friday October 16 at 6:00 AM GMT, there were 1602828000000 milliseconds elapsed since the Epoch, thus the UTC Time.

**UTC Offset:** The value that allows computers to take UTC Time and a Time Zone, and format it to a Wall-Clock display time for people in a specific Time Zone to read.  The offset is expressed in hours and minutes, such that +05:30 means add 5 hours and 30 minutes to the UTC Time which will format into a local Wall-Clock time.

**UTC Offset Name:** Since people think of time offsets relative to the Time Zone, there are names for the offsets which match to the Time Zone name. At the UTC time above, Friday October 16 6:00 AM GMT, the offset in India is +05:30 and displays as India Standard Time (IST) while in San Francisco the offset is -07:00, and displays as Pacific Daylight Time (PDT).

Since these offset names are also called "Time Zones" in conversation, it can create confusion. Don't be fooled! Offsets are NOT Time Zones. [Read this blog](https://spin.atomicobject.com/2016/07/06/time-zones-offsets/) which describes the difference well. 

Be careful of sites which present a reference list of Time Zones when what they actually list are the Offset names (i.e. "Central Daylight Time").  Two such sites are listed in the "Helpful Links" section.

Arizona offers a good illustration. Most of the state does not observe Daylight Saving time, and the state is usually associated with Mountain Time Zone.  This means Denver can have an offset of MDT while the Phoenix offset is MST, while they represent the same UTC Time point:

![Denver and Phoenix](https://github.com/jasonmerecki/blog/blob/main/useepoch/DenPhoExample.png)

The actual geographical area Time Zone is "America/Phoenix" as it defines a geographical area where people agree to set calendars and clocks together. This specific geographic area has agreed to have a -07:00 offset all year long, and does not observe Daylight Saving Time. The name for -07:00 to describe this offset is "Mountain Standard Time" (and[some systems will call this "Arizona Time"](https://stackoverflow.com/questions/42424829/getting-the-arizona-standard-time-in-net), further underscoring the need to differentiate a Time Zone region from a UTC Offset name).


## Putting it together

By example, a UTC time value of 1602828000000 and a Time Zone of India Time (i.e. Asia/Kolkata) goes through these steps:

 - Start with UTC Time 1602828000000
 - Lookup the offset for Asia/Kolkata for that time of year, which is +05:30 
 - Applies the offset hours/minutes, resulting in a 'local' millisecond time
 - Format the result using calendar/clock fields, resulting in "October 16, 2020 11:30 AM"

The same display time steps for San Francisco 

 -  Start with UTC Time 1602828000000
 - Lookup the offset for America/Los_Angeles for that time of year, which is +07:00 
 - Applies the offset hours/minutes, resulting in a 'local' millisecond time
 - Format the result using calendar/clock fields, resulting in "October 15, 2020 11:00 PM"

I purposely described this as a display formatting concern. I view this the same way as a computer storing a one byte binary value as 00000110 and formatting it to display as the number 6 for people to read. The underlying value did not change, only the display. 

The key point to UTC Time is that it is the *same time value no matter the Wall-Clock format.*  People in Mumbai are enjoying lunch while people in San Francisco are enjoying their late night, both at exactly UTC time 1602828000000.



## Scheduled pickup time

Gort has a solution, allow the server to convert the location time to a local server time.   If all times are expressed as a local server time, then the comparison works.

| Pickup time | Location | Server Time|
|--|--|--|
| 2020-10-31 08:00| Chicago, IL | 2020-10-31 09:00|
| 2020-10-31 09:00| Dallas, TX | 2020-10-31 10:00|
| 2020-10-31 08:30| San Francisco, CA | 2020-10-31 11:30|

This seems to work... at first.

## Location pings/updates

Next, Gort must collect location updates from the trucks making the deliveries.  The following GPS pings are received by Gort's application:

| Ping time | Lat/Lon location |
|--|--|
| 2020-10-31 05:41 | 41.709001, -86.806262 |
| 2020-10-31 05:52 | 41.676503, -86.835408 |
| 2020-10-31 06:17 | 41.827821, -86.674901 |
| 2020-10-31 06:23 | 41.765610, -86.743576 |
| 2020-10-31 06:25 | 41.601892, -87.148836 |

The client asks Gort to plot these on a map to show the driver's route and progress. Naturally, Gort sorts the pings from earliest-to-latest to show progress over time, with this result:

![Mapped GPS pings](https://github.com/jasonmerecki/blog/blob/main/useepoch/GortMap01a.png)

It looks like this delivery company has trucks that bend time and space! 

Actually, the truck has crossed the time zone line. South of the Time Zone line, pings 1, 2, and 5 happened where clocks were set back 1 hour from those north of the line.

![Time zone line in blue](https://github.com/jasonmerecki/blog/blob/main/useepoch/GortMap01b.png)

This happened because, Gort's app only uses wall-clock time with no offset, even though the GPS sends offset information. For example, the GPS can send "2020-10-31 05:52 -05:00" but Gort's app truncated the offset.

## More time math challenges

For the same pickup time as before, Gort's app also gets delivery time. Drivers get paid for their time on the road, and Gort's app must calculate the elapsed time. 

| Pickup time | Pickup Location | Delivery time | Delivery Location|
|--|--|--|--|
| 2020-10-31 08:00| Chicago, IL | 2020-11-01 08:00| Houston, TX |

Gort's app calculates that the trip took 24 hours. But the client is upset again, the driver says the trip took 25 hours. Gort finds that the origin and destination are in the the same Time Zone, and is confused, if the driver never crossed a Time Zone, why is this answer wrong?

It's wrong, because Daylight Saving Time ended on November 1, 2020, and everyone in America/Chicago Time Zone rolled back their clocks 1 hour, adding that hour overnight.

Thus the driver is correct, the time between "2020-10-31 08:00" and "2020-11-01 08:00" is 25 hours, in the America/Chicago Time Zone.

Gort adds a routine in his app to add one hour to all the calculated elapsed durations on November 1, 2020.  Client is happy again. Until they find a different delivery:


| Pickup time | Pickup Location | Delivery time | Delivery Location|
|--|--|--|--|
| 2020-10-31 08:00| Denver, CO | 2020-11-01 08:00| San Francisco, CA |
| 2020-10-31 08:00| Phoenix, AZ | 2020-11-01 08:00| San Francisco, CA |

The client reports that the first result should be 26 hours while the second is only 25 hours. Gort is in a frenzy, both Denver and Phoenix look like they are in Mountain Time Zone on a map. How could they be different?

Phoenix (and most of Arizona) does not observe Daylight Saving Time and did not roll back their clocks. Arizona is a special geographic Time Zone called America/Phoenix. Remember, a Time Zone is a geographic region where people agree how to apply an offset, but the offset may be different in the same Time Zone depending on the time of the year. 

What Gort needs in the app is the UTC Time for storage and math, and the offset for a display format.  

## UTC Time solves this

Briefly, here are the outcomes of the challenges above, using UTC Time.

Note that the formatted display with the offset also helps visually explain the outcome; it is easier to see why 08:30 comes after 09:00 when the offset context is part of the output.

For sorting pickup times, the underlying UTC Time value allows proper comparisons, and the indicated offset will format the UTC Time into a string the users will understand (bold highlighting included to emphasize the comparison outcome)

| UTC Time | Pickup time formatted | Location |
|--|--|--|
| 16041**492**00000 | 2020-10-31 08:00 -05:00| Chicago, IL |
| 16041**528**00000 | 2020-10-31 09:00 -05:00| Dallas, TX |
| 16041**582**00000 | 2020-10-31 08:30 -07:00| San Francisco, CA |

The GPS ping location updates are sent with a time offset, which all systems can store into a UTC Time type. Changing the app to use the offset results in this sorting, and the truck will appear to drive smoothly from East to West on the road:

| UTC Time | Ping time formatted | Lat/Lon location |
|--|--|--|
| 1604139420000 | 2020-10-31 06:17 -04:00 | 41.827821, -86.674901 |
| 1604139780000 | 2020-10-31 06:23 -04:00 | 41.765610, -86.743576 |
| 1604140860000 | 2020-10-31 05:41 -05:00 | 41.709001, -86.806262 |
| 1604141520000 | 2020-10-31 05:52 -05:00 | 41.676503, -86.835408 |
| 1604143500000 | 2020-10-31 06:25 -05:00 | 41.601892, -87.148836 |

For the elapsed delivery time, the UTC Time difference results in the proper elapsed time. And, the time format with the offset value helps the user understand why the difference is 25 hours, they can see the offset changed between the pickup and delivery even though the locations are in the same Time Zone:

| UTC Time pickup| Pickup time format | Pickup Location | UTC Time delivery | Delivery time format | Delivery Location|
|--|--|--|--|--|--|
| 1604149200000 | 2020-10-31 08:00 -05:00| Chicago, IL | 1604239200000 | 2020-11-01 08:00 -06:00| Houston, TX |

The difference between the delivery and the pickup is now 90000000 milliseconds, which is equivalent to 25 hours.


## Why this is hard  

The bottom line is that Wall Clock time creates bugs when the values apply to different Time Zone areas, and require math operations, such as greater-than (sorting) and subtraction (time elapsed).

These are avoidable by using UTC Time instead, and treating the display of a clock time as a format for display, not as an actual value. 

However, people think using Wall Clock time. No one can look at time 1602828000000 and understand it.  At some point is must be converted to a readable format like "October 15, 2020 23:00 -07:00" and the computer engineer in San Francisco can read and understand the time.

The challenge is that the engineer reading the data may conclude that it "looks right", or they may overlook the offset of "-07:00", trying only to work with the clock time being displayed. 

Working with UTC Time requires the tech teams (engineering, QA) to shift how they think about time values. That's hard, since people read clocks since elementary school and rarely need to deal with time math. 

It is further hard because in conversation, people will refer to an offset at the location's Time Zone and that leads to further errors. Sometimes developers will look up an offset name such as "Central Daylight Time (CDT)" and store that as a location's Time Zone. The problem is that CDT means -05:00 offset, and when that location stops observing Daylight Saving Time, the offset CDT no longer applies.




## Eliminate datetime type! 

Microsoft .NET and SQL Server both support a datetime type which cannot store offset information across time zone offsets.  The alternative is the datetimeoffset type, which defines an exact point in time (a UTC Time) and has offset information. 

In my opinion, the very existence of the datetime type has made it easier for developers to fall into the pitfalls described above. 

Even [Microsoft's own documentation](https://docs.microsoft.com/en-us/dotnet/standard/datetime/choosing-between-datetime) suggests that the datetimeoffset should be the default choice. 


## Helpful Links

World clock converter: https://www.timeanddate.com/worldclock/converter.html

Another way to describe these ideas
https://stackoverflow.com/questions/4331189/datetime-vs-datetimeoffset

These two sites list Time Zone abbreviations but they actually mean UTC Offset names

https://www.timeanddate.com/time/zones/

https://www.timetemperature.com/abbreviations/united_states_time_zone_abbreviations.shtml




> Written with [StackEdit](https://stackedit.io/).
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTIwNjMyODA1MDAsLTM2NjEyNzk2MiwtNj
U1MjEwMjE1LC0yMzkwNjU1NCwtMTk3NDUwMzg1NiwtNDQ3OTMw
ODE5LC02NzY3Njc1MDQsMTY2MjA5NjkxMiwtNDUzMzEzMzM1LC
04NTY4NjUzMDEsLTE1NjE0NzcxNDAsLTI2NDkxNDIzNiwtMjY0
OTE0MjM2LDEyODY5NTUxMjUsMzc0NTUwODA2LDgyNjg2MTYwMC
wxNTg5MDM3NzUxLDI0NTA0OTA1NSw1OTM2ODIwLDExNzcwOTk2
OTRdfQ==
-->