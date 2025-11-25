
I connected to the ftp port as `anonymous` and I got in!!!

![](attachments/Pasted%20image%2020251125151959.png)


and I did list the files and I saw a `note.txt` so I got it:

![](attachments/Pasted%20image%2020251125152100.png)

and then I read the file and the contents were some credentials and password `cd73502828457d15655bbd7a63fb0bc8`:

```
Hello Heath !
Grimmie has setup the test website for the new academy.
I told him not to use the same password everywhere, he will change it ASAP.


I couldn't create a user via the admin panel, so instead I inserted directly into the database with the following command:

INSERT INTO `students` (`StudentRegno`, `studentPhoto`, `password`, `studentName`, `pincode`, `session`, `department`, `semester`, `cgpa`, `creationdate`, `updationDate`) VALUES
('10201321', '', 'cd73502828457d15655bbd7a63fb0bc8', 'Rum Ham', '777777', '', '', '', '7.60', '2021-05-29 14:36:56', '');

The StudentRegno number is what you use for login.


Le me know what you think of this open-source project, it's from 2020 so it should be secure... right ?
We can always adapt it to our needs.

-jdelta
```

Searching the Internet I found that this hashing is `MD5` so I decrypted the hash and the password is `student`