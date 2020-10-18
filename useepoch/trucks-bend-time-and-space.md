# Trucks Bend Time And Space!
*Or, why time is hard in computer science.*

## First, some definitions

**Wall-Clock time:** The way people are used to measuring time through the day. When people view "11:30 AM" on the wall clock they conclude it's mid-day and time for lunch, while viewing "11:00 PM" indicates it is night time and possibly time to sleep. 

**Time Zone:** A geographic area where people have agreed to set their calendars and clocks in the same way.  Most use the Gregorian calendar, and set their clocks so that "11:30 AM" is roughly the middle of daylight hours. At certain times of the year, people in the Time Zone may or may not change their clocks by 1 hour, to follow Daylight Savings Time (or British Summer Time if you prefer). 

**UTC Time:** Computers measure time by the elapsed milliseconds starting from a fixed point in time, called the Epoch, which is based on January 1, 1970, 12:00 AM in Greenwich Mean Time. The resulting value is the Coordinated Universal Time (UTC). For example, on Friday October 16 at 6:00 AM GMT, there were 1602828000000 milliseconds elapsed since the Epoch, thus the UTC Time.

**UTC Offset:** The value that allows computers to take UTC Time and a Time Zone, and format it to a Wall-Clock display time for people in a specific Time Zone to read.  The offset is expressed in hours and minutes, such that +05:30 means add 5 hours and 30 minutes to the UTC Time which will format into a local Wall-Clock time.

**UTC Offset Name:** Since people think of time offsets relative to the Time Zone, there are names for the offsets which match to the Time Zone name. At the UTC time above, Friday October 16 6:00 AM GMT, the offset in India is +05:30 and displays as India Standard Time (IST) while in San Francisco the offset is -07:00, and displays as Pacific Daylight Time (PDT).

Since these offset names are also called "Time Zones" in conversation, it can create confusion. Don't be fooled! Offsets are NOT Time Zones. [Read this blog](https://spin.atomicobject.com/2016/07/06/time-zones-offsets/) which describes the difference well. 


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


# Gort goes to work

Let's say a computer engineer named Gort has been hired to write software for a package delivery company. He is used to thinking in Wall-Clock time, and chooses to store the date and time values in a Wall-Clock time manner.
These examples will show formatted time in "YYYY-MM-DD HH:SS" and use 24-hour time.

## Scheduled pickup time

The client will have package pickup and dropoff times around North America. The incoming schedule data Gort receives looks like this:

| Pickup time |  |
|--|--|
| 2020-10-31 08:00|  |
| 2020-10-31 08:30|  |
| 2020-10-31 09:00|  |

The client wants Gort to display the pickup times sorted from earliest to latest, so they can coordinate their drivers. Gort sorts, adds the location to the output and gets this result:

| Pickup time | Location |
|--|--|
| 2020-10-31 08:00| Chicago, IL |
| 2020-10-31 08:30| San Francisco, CA |
| 2020-10-31 09:00| Dallas, TX |

The client is upset. The pickup in San Francisco is in Pacific time zone, and 08:30 in that location is after the 09:00 pickup in Dallas. But sorting the Wall-Clock time only knows the Wall-Clock time for comparison. 

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

The client asks Gort to plot these on a map to show the driver's route and progress, with this result:

![Mapped GPS pings](https://github.com/jasonmerecki/blog/blob/main/useepoch/GortMap01a.png)

It looks like this delivery company has trucks that bend time and space! 

Actually, the truck has crossed the time zone line. South of the Time Zone line, pings 3, 4, and 5 happened where clocks were set back 1 hour from those north of the line.

![Time zone line in blue](https://github.com/jasonmerecki/blog/blob/main/useepoch/GortMap01b.png)

This happened because, Gort's app only uses wall-clock time with no offset, even though the GPS sends offset information. For example, the GPS can send "2020-10-31 05:52 -05:00" but Gort's app truncated the offset.

## More time math challenges

For the same pickup time as before, Gort's app also gets delivery time. Drivers get paid for their time on the road. 

| Pickup time | Pickup Location | Delivery time | Delivery Location|
|--|--|--|--|
| 2020-10-31 08:00| Chicago, IL | 2020-11-01 08:00| Houston, TX |

Gort's app calculates that the trip took 24 hours. But the client is upset again, the driver says the trip took 25 hours. Gort finds that the origin and destination are in the the same Time Zone, and is confused, if the driver never crossed a Time Zone, why is this answer wrong?

Because Daylight Savings Time ended on November 1, 2020, and everyone in Central Time Zone rolled back their clocks 1 hour, adding that hour overnight.

## What to do 

The bottom line is that Wall Clock time creates bugs when the values apply to different Time Zone areas and require math operations, such as greater-than (sorting) and subtraction (time elapsed).

These are avoidable by using UTC Time instead, and treating the display of a clock time as a format for display, not as an actual value. 


## Helpful sites

World clock converter: https://www.timeanddate.com/worldclock/converter.html

Another way to describe these ideas
https://stackoverflow.com/questions/4331189/datetime-vs-datetimeoffset


> Written with [StackEdit](https://stackedit.io/).
<!--stackedit_data:
eyJoaXN0b3J5IjpbMTk0Nzc2MjI4LDEyODY5NTUxMjUsMzc0NT
UwODA2LDgyNjg2MTYwMCwxNTg5MDM3NzUxLDI0NTA0OTA1NSw1
OTM2ODIwLDExNzcwOTk2OTQsMTg5OTkzNjc2MSwxODk5OTM2Nz
YxLDE1MTQzMzcwNDUsLTg3NDI1MDE5LC03MDE0OTU5MTIsLTEy
MjcyNDg0NzYsMzI1ODcwNzUxLDE0MzgyMDUzOTYsLTEyNzE1Mz
IwNjQsODEyODA4MTQyLC0xOTg3MzMwMjE4XX0=
-->