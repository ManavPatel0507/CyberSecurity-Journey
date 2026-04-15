in this lab the errors were visible on the webpage hence it can be exploited.

first we opened the lab in burpsuit and sent the get request to the repeator then sent the querry again to check and it worked.

then add ' to the end of the reacking id and it gave use error. and appended the comment stntax -- then it worked again. also tried '' withoit comments syntex it also worked

now as it is a postgresql database it has a trait thet it verbose erroe message that includes the value as well eg: ERROR: invalid input syntax for type integer: "administrator".

so we can take advantage of this and make it throw error using type casting so it gives us the value from the database.

so the first thing we did is: TrackingId=ogAZZfxtOKUELbuJ' AND CAST((SELECT 1) AS int)--
this however gave error stating AND condition must be a boolean expression. 

so we simply did this and added a comparesion operator to the querry: TrackingId=ogAZZfxtOKUELbuJ' AND 1=CAST((SELECT 1) AS int)--

now this works, hence we can now retrive username from the table: TrackingId=ogAZZfxtOKUELbuJ' AND 1=CAST((SELECT username FROM users) AS int)--

however, this gave error saying that the tracking id length is too long so the comment(--) syntax is removed from the querry itself.

so to counter this we remove the tracking id itself and then ran it again and it gave another error that there are more than 1 rows being returned so it cant show that so we add linit 1

TrackingId=' AND 1=CAST((SELECT username FROM users LIMIT 1) AS int)--

it worked and it gave the 1st username which was administrator. the error message was: ERROR: invalid input syntax for type integer: "administrator"

now we can modify this querry to return the password for the administrator: TrackingId=' AND 1=CAST((SELECT password FROM users LIMIT 1) AS int)--

we got the password and now logined into the webpage as admin and solved the lab. 

