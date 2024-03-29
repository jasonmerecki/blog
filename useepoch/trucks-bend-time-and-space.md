# Trucks Bend Time And Space!
*Or, why time is hard in computer science.*

It is surprisingly easy for computer programmers to make mistakes with time calculations and time storage in computer programming. Time math requires transformations that people can do in their head without realizing. I will show some of the pitfalls that developers can make and hopefully help some avoid them. 

These examples will show formatted time in "YYYY-MM-DD HH:SS" and use 24-hour time.


## Gort goes to work

A computer engineer named Gort has been hired to write software for a package delivery company. He is used to thinking about time by reading wall clocks, and chooses to store the date and time values in a wall clock way when he reads it. 

The client will have package pickup and dropoff times around North America.  The incoming schedule data Gort receives looks like this:

| Pickup time |  |
|--|--|
| 2020-10-31 08:00|  |
| 2020-10-31 08:30|  |
| 2020-10-31 09:00|  |

Gort follows his client's request and sorted the pickup times from earliest to latest, so they can coordinate their drivers. Then Gort adds the location to the output, and gets this result:

| Pickup time | Location |
|--|--|
| 2020-10-31 08:00| Chicago, IL |
| 2020-10-31 08:30| San Francisco, CA |
| 2020-10-31 09:00| Dallas, TX |

The client is upset. The pickup in San Francisco is in Pacific time zone, and 08:30 in that location is after the 09:00 pickup in Dallas. But sorting the wall clock time compares only the time part for comparison, not the time zone offset.

Let's look closer at what happened, and how to avoid this outcome. 

## First, some definitions

**Wall-Clock time:** The way people are used to measuring time through the day. When people view "11:30 AM" on the wall clock, they conclude it's mid-day and time for lunch, while viewing "11:00 PM" indicates it is night time and possibly time to sleep. 

**Time Zone:** A geographic area where people have agreed to set their calendars and clocks in the same way.  Most use the Gregorian calendar, and set their clocks so that "11:30 AM" is roughly the middle of daylight hours. At certain times of the year, people in the Time Zone may change their clocks by 1 hour, to follow daylight saving time. 

The best source of Time Zone areas by this definition is the standard IANA Time Zone database.  A good example of an actual Time Zone map is here: [http://efele.net/maps/tz/us/](http://efele.net/maps/tz/us/). 

People may be surprised to see odd sections of their legal Time Zone split into a different area, and that is because the area set their clocks in a different way in the past. Thus the zone "America/Indiana/Knox" is special within the state of Indiana. Although that zone now matches "America/New_York", it must remain in a different zone for correct formatting of past dates, when they did not match that zone. 

**UTC Time:** Computers measure time by the elapsed milliseconds starting from a fixed point in time, called the Epoch, which is based on January 1, 1970, 12:00 AM in Greenwich Mean Time. The resulting value is the Coordinated Universal Time (UTC). For example, on Friday October 16 at 6:00 AM GMT, there were 1602828000000 milliseconds elapsed since the Epoch, thus the UTC Time.

**UTC Offset:** The value that allows computers to take UTC Time and a Time Zone, and format it to a Wall-Clock display time for people in a specific Time Zone to read.  The offset is expressed in hours and minutes, such that +05:30 means add 5 hours and 30 minutes to the UTC Time, which will format into a local Wall-Clock time.

**UTC Offset Name:** Since people think of time offsets relative to the Time Zone, there are names for the offsets which match to the Time Zone name. At the UTC time above, Friday October 16 6:00 AM GMT, the offset in Mumbai is +05:30 and displays as India Standard Time (IST) while in San Francisco the offset is -07:00, and displays as Pacific Daylight Time (PDT).

Since these offset names are also called "Time Zones" in conversation, it can create confusion. Don't be fooled! Offsets are NOT Time Zones. [Read this blog](https://spin.atomicobject.com/2016/07/06/time-zones-offsets/) which describes the difference well. 

Arizona offers a good illustration. The state's legal Time Zone is Mountain Time which includes the states of Arizona and Colorado. But most of Arizona does not observe daylight saving time.  This means Denver, Colorado can have an offset of -06:00 MDT while the Phoenix, Arizona offset is -07:00 MST, when they represent the same point in time:

![Denver and Phoenix](https://github.com/jasonmerecki/blog/blob/main/useepoch/DenPhoExample.png)

The key difference is that Phoenix is in the Time Zone area named  "America/Phoenix".  It defines the geographical area  where people agree to set calendars and clocks together, and have a -07:00 offset all year long. It is separate from the "America/Denver" zone, where those people have agreed to observe daylight saving time. They are both Mountain Time on a map, but are distinct Time Zones since they set clocks differently. 

([Some systems will call this "Arizona Time"](https://stackoverflow.com/questions/42424829/getting-the-arizona-standard-time-in-net), which is actually a synonym for the -07:00 offset, applied all the time).


## Putting it together

By example, a UTC time value of 1602828000000 and a location of Mumbai goes through these steps:

 - Start with UTC Time 1602828000000
 - Lookup the offset for Asia/Kolkata for that time of year, which is +05:30 
 - Applies the offset hours/minutes, resulting in a 'local' millisecond time
 - Format the result using calendar/clock fields, resulting in "October 16, 2020 11:30 AM"

The same display time steps for San Francisco:

 -  Start with UTC Time 1602828000000
 - Lookup the offset for America/Los_Angeles for that time of year, which is -07:00 
 - Applies the offset hours/minutes, resulting in a 'local' millisecond time
 - Format the result using calendar/clock fields, resulting in "October 15, 2020 11:00 PM"

I purposely described this as a display formatting concern. I view this the same way as a computer storing a one byte binary value as 00000110 and formatting it to display as the number 6 for people to read. The underlying value did not change, only the display. 

The key point to UTC Time is that it is the *same time value no matter the Wall-Clock format.*  People in Mumbai are enjoying lunch while people in San Francisco are enjoying their late night, both at exactly UTC time 1602828000000.

## Scheduled pickup time

Getting back to our computer engineer, Gort has a solution for the pickup time sorting issue. He adds a column to store the equivalent server time with each dropoff. Now the comparison between dropoff times work, and the sort looks correct.

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

This unusual mapping happened because Gort's app only uses wall-clock time with no offset, even though the GPS sends offset information. For example, the GPS can send "2020-10-31 05:52 -05:00" but Gort's app truncated the offset.

## More time math challenges

For the same pickups time as before, Gort's app also gets delivery time. Drivers get paid for their time on the road, and Gort's app must calculate the elapsed time. 

| Pickup time | Pickup Location | Delivery time | Delivery Location|
|--|--|--|--|
| 2020-10-31 08:00| Chicago, IL | 2020-11-01 08:00| Houston, TX |

Gort's app calculates that this Chicago to Houston trip took 24 hours. But the client is upset again, the driver says the trip took 25 hours. Gort finds that the origin and destination are in the the same Time Zone, and is confused, if the driver never crossed a Time Zone, why is this answer wrong?

The source of the error is that daylight saving time ended on November 1, 2020, and everyone in America/Chicago Time Zone rolled back their clocks 1 hour, adding that hour overnight.

Thus the driver is correct, the time between "2020-10-31 08:00" and "2020-11-01 08:00" is 25 hours, in the America/Chicago Time Zone.

Gort adds a routine in his app to add one hour to all the calculated elapsed durations on November 1, 2020.  The client is happy again. Until they find a different delivery:


| Pickup time | Pickup Location | Delivery time | Delivery Location|
|--|--|--|--|
| 2020-10-31 08:00| Denver, CO | 2020-11-01 08:00| San Francisco, CA |
| 2020-10-31 08:00| Phoenix, AZ | 2020-11-01 08:00| San Francisco, CA |

The client reports that the first result should be 26 hours while the second is only 25 hours. Gort is in a frenzy, both Denver and Phoenix look like they are in Mountain Time Zone on a map. How could they be different?

Recall that Phoenix is in the America/Phoenix Time Zone, different from America/Denver. Gort was misled by the map which shows a legal time zone, not the actual area where people set their clocks the same way. 

What Gort needs in the app is the UTC Time for storage and math, combined with the offset for a display format.  

## UTC Time solves this

Briefly, here are the outcomes of the challenges above, when UTC Time is used instead.

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

The bottom line is that Wall Clock time, by itself, creates bugs when the values apply to different Time Zone areas, and the values require math operations, such as greater-than (sorting) and subtraction (time elapsed).

These errors are avoidable by using UTC Time instead, and treating the display of a clock time as a format for display, not as an actual value. 

However, people think using Wall Clock time. No one can look at time 1602828000000 and understand it.  At some point is must be converted to a readable format like "October 15, 2020 23:00 -07:00", and the computer engineer in San Francisco can read and understand the time.

The challenge is that the engineer reading the data may conclude that it "looks right", or they may overlook the offset of "-07:00", trying only to work with the clock time being displayed. 

Working with UTC Time requires the tech teams (engineering, QA) to shift how they think about time values. That's hard, since people read clocks since elementary school and rarely need to deal with time math. 

It is further hard because of the casual way people use the term Time Zone to describe an offset. It's easy to conclude that using Eastern Standard Time will always convert correctly, but that is the offset -05:00, and does not apply to the location during daylight saving time, when the offset is -04:00.


## An example to try in Javascript

This exercise is another easy way to see the impact of using
millisecond epoch time versus wall-clock time. Start with 
the representation of the time "2021-03-14T01:35:00-06:00" 
in Chicago (-6 hour offset)

 ```
> var chiDate = new Date("2021-03-14T01:35:00-06:00")
> chiDate.getTime()
< 1615707300000
> chiDate.toLocaleString("en-US", {timeZone: "America/Chicago"})
< "3/14/2021, 1:35:00 AM"
```

If this is formatted to the "America/Denver" time zone
then the wall-clock time output is one hour earlier

```
> var denDate = chiDate.toLocaleString("en-US", {timeZone: "America/Denver"})
> denDate
< "3/14/2021, 12:35:00 AM"
```

Now let's format in "America/New_York" locale, one hour 
later than Chicago

```
> var nyDate = chiDate.toLocaleString("en-US", {timeZone: "America/New_York"})
> nyDate
< "3/14/2021, 3:35:00 AM"
```

Woah, that is TWO hours ahead of the Chicago time. 
That's because the "America/New_York" time zone has
all set their wall clocks one hour ahead for daylight
savings time.

But if we check `chiDate`, it's still the same epoch time:

```
> chiDate.getTime()
< 1615707300000
```


## My Pet Peeve 1: Eliminate datetime type! 

Microsoft .NET and SQL Server both support a datetime type which cannot store offset information across time zone offsets.  The alternative is the datetimeoffset type, which defines an exact point in time (a UTC Time) and has offset information. 

In my opinion, the very existence of the datetime type has made it easier for developers to fall into the pitfalls described above. 

Even [Microsoft's own documentation](https://docs.microsoft.com/en-us/dotnet/standard/datetime/choosing-between-datetime) suggests that the datetimeoffset should be the default choice. 

Please, developers, do not use datetime. I understand datetimeoffset is sometimes harder to use and read, but it avoids bugs which are very difficult to solve later.

## My Pet Peeve 2: Do not use 'standard' to describe the Time Zone! 

It's common to see people use the word 'standard' when giving time zone context.  For example: "the server upgrade will start at 2020-10-12 19:00 Eastern Standard Time."  But that's probably wrong, because EST is a -05:00 offset, and the Eastern time zone in the U.S. observes daylight saving time on October 12, equivalent to -04:00 offset.

I promise, now that you have that description of 'standard' versus 'daylight' in the context of a displayed time, you will start to notice the apps that get it right, and those that make mistakes.


## Helpful Links

World clock converter, where the correct offset name is displayed: https://www.timeanddate.com/worldclock/converter.html

Another article that describes these ideas:
https://stackoverflow.com/questions/4331189/datetime-vs-datetimeoffset

These two sites list Time Zone abbreviations but they actually mean UTC Offset names:
https://www.timeanddate.com/time/zones/
https://www.timetemperature.com/abbreviations/united_states_time_zone_abbreviations.shtml




> Written with [StackEdit](https://stackedit.io/).
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTE4OTcwNTE1MjgsNTQ5MDQzNDMsLTU1MD
cwNTE5MSwzNDQ2Mjc4NTAsMTA1MDgwMTIxLDE4MzI4ODU1MDMs
MTM5NDEzMzI3NSw4Njg3MTI2NjMsMTA0MTk5NTM4MywtMzY2MT
I3OTYyLC02NTUyMTAyMTUsLTIzOTA2NTU0LC0xOTc0NTAzODU2
LC00NDc5MzA4MTksLTY3Njc2NzUwNCwxNjYyMDk2OTEyLC00NT
MzMTMzMzUsLTg1Njg2NTMwMSwtMTU2MTQ3NzE0MCwtMjY0OTE0
MjM2XX0=
-->