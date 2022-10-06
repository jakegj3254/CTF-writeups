# Summary 
In this challenge, the goal is to grab a flag from a website game database. In this case the entry point is the login page that occurs after finishing a game, there is a SQL injection attack that can be used to grab any info from the database, in this case we want the info the user "Mr. Flag", for fairly obvious reasons(it's literally in the name).

# Steps 
1. First things first, looking through the provided code, in the file scores.sql, we can confirm that flag is the password of the user Mr. Flag, since his password is listed as osu{redacted} showing that it is in the form of a flag.
2. Trying basic SQL injection tests, such as inputting a single quote, we can see that the login page is indeed vulnerable to SQL injection since we get a SQL error back with that input.
3. To make things easier for myself, I used Burp Suite to copy the POST request to a file so it can be read back when I need to.
4. In this case I used a tool called sqlmap to make injection easier, I used the previous POST file and passed it to the tool with the -r parameter.
5. After making sure sqlmap worked, I checked the source code to find which database and column I'm looking for and sent the command  ```sqlmap -r POST.txt  -D scores -C password --dump``` where -D specified the database, -C specified the column and --dump printed the info in that column.
6. After that ran, I got back a list of passwords for each user, giving me the flag needed to complete the challenge (full output can be viewed below)
```
+-------------------------------------------------+
| password                                        |
+-------------------------------------------------+
| ball_industry_large_beyond                      |
| bank_addition_high_string                       |
| birds_addition_twelve_cloth                     |
| cried_bread_tear_agreed                         |
| future_many_maybe_compound                      |
| osu{Py7h0n_F_s7r1n9S_4Re_8E4u71FuL_4ND_4M421N9} |
| plant_hours_dead_ring                           |
| property_plains_brown_shore                     |
| same_held_December_maybe                        |
| thus_etching_insects_evening                    |
| voice_party_rhythm_weight                       |
+-------------------------------------------------+

```
