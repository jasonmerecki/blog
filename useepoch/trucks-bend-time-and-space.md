

# Trucks Bend Time And Space!
*Or, why time is hard in computer science.*

## First, some definitions

**Wall-Clock time:** The way people are used to measuring time through the day. When people view "11:30 AM" on the wall clock they conclude it's mid-day and time for lunch, while viewing "11:00 PM" indicates it is night time and possibly time to sleep. 

**Time Zone:** A geographic area where people have agreed to set their calendars and clocks in the same way.  Most use the Gregorian calendar, and set their clocks so that "11:30 AM" is roughly the middle of daylight hours. At certain times of the year, people in the Time Zone may or may not change their clocks by 1 hour, to follow daylight savings time (or British Summer Time if you prefer). 

**UTC Time:** Computers measure time by the elapsed milliseconds starting on a fixed point in time, called the Epoch, which is based on January 1, 1970, 12:30 AM in Greenwich Mean Time. The resulting value is the Coordinated Universal Time (UTC). On Friday October 16 at 6:00 AM GMT, there were 1602828000000 milliseconds elapsed since the Epoch, thus the UTC time.

**UTC Offset:** The value that allows computers to take UTC Time and a Time Zone, and convert it to Wall-Clock time for people in that Time Zone to read.  It is expressed in hours and minutes, such that +05:30 means add 5 hours and 30 minutes to the UTC Wall-Clock time.

**UTC Offset Name:** Since people think of time offsets relative to the Time Zone, there are names for the offsets which match to the Time Zone name. In the above examples, the offset in India displays as India Standard Time (IST) while in San Francisco it displays as Pacific Daylight Time (PDT).

Since these offset names are also called "Time Zones" it can create confusion. Don't be fooled! [Read this blog](https://spin.atomicobject.com/2016/07/06/time-zones-offsets/) which describes the difference well. 


## Putting it together

I purposely described this as a conversion. It is the same as a computer storing a one byte binary value as 00000110 and converting it to 6 for people to read.

By example, a UTC time value of 1602828000000 and a Time Zone of India Time (i.e. Asia/Kolkata) goes through these steps:

 - Convert UTC Time 1602828000000 to GMT Wall-Clock time "Friday October 16 6:00 AM" 
 - Lookup the offset for Asia/Kolkata for that time of year, resulting in  +05:30 
 - Add the offset and display the result "October 16, 2020 11:30 AM"

The same UTC time converted with US-Pacific (America/Los_Angeles) will lookup an offset of -07:00 due to daylight savings time, and convert to "Thursday October 15, 11:00 PM". 

The key point to UTC time is that it is the *same time value no matter the Wall-Clock conversion.*  People in Mumbai are enjoying lunch while people in San Francisco are enjoying their late night, at UTC time 1602828000000.


# Gort goes to work

Let's say a computer engineer named Gort has been hired to write software for a package delivery company. He is used to thinking in Wall-Clock time and tends to 





## Helpful sites

World clock converter: https://www.timeanddate.com/worldclock/converter.html

Another way to describe these ideas
https://stackoverflow.com/questions/4331189/datetime-vs-datetimeoffset


> Written with [StackEdit](https://stackedit.io/).
<!--stackedit_data:
eyJoaXN0b3J5IjpbMTE2NTI5ODAwNywtMTk4NzMzMDIxOF19
-->