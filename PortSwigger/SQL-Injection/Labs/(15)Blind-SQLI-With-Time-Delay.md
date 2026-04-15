this lab contain blind sqli with time delay

first opened the lab in burp suit and sent the packet to the repeater
first noticed that anything you inject says 200 ok no errors

then apended this to the tracking id.: TrackingId=x'||pg_sleep(10)--

after that noticed that it takes 10s to load and process querry.

now just add a payload like  TrackingId=x'|| (select '') pg_sleep(10)-- and it solved the lab.
