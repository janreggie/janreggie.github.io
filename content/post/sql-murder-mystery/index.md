---
title: "SQL Murder Mystery"
type: post
date: 2020-09-21T13:29:38+08:00
publishdate: 2020-09-21
lastmod: 2020-09-21
draft: false
description: "The SQL Murder Mystery is a fascinating game for SQL users to solve a crime"
tags:
  - "sql"
  - "challenges"
categories:
  - "challenges"
---

The [SQL Murder Mystery](https://mystery.knightlab.com/) is quite an interesting game
wherein users are invited to solve a murder mystery.
This is a great way for me to hone my SQL skills
after completing learning SQL through [Select Star SQL](https://selectstarsql.com/).
In this post I will discuss my solution for finding out who did the murder,
and a small activity afterwards.

<!--more-->

## Details

A murder has occurred sometime on January 15, 2018 at SQL City.
You are given access to a database containg several relevant tables,
such as person data, drivers' licenses, and gym memberships.

{{< optfigure src="schema"
  alt="The schema for the various tables in SQL Murder Mystery"
  link="https://mystery.knightlab.com/schema.png"
  caption=`The schema. Why the residents decided to give up all of their information like this is beyond me.
    Fortunately, everything here is a work of fiction.` >}}

## My solution

```sql
WITH murder AS (
  -- determine details of the murder
  SELECT *
  FROM crime_scene_report
  WHERE date = 20180115
    AND TYPE = 'murder'
    AND city = 'SQL City'
),
first_witness AS (
  -- last house on Northwestern Dr belongs to Morty Schapiro
  SELECT *
  FROM person
  WHERE address_street_name = 'Northwestern Dr'
    AND address_number = (
      SELECT MAX(address_number)
      FROM person
    )
),
second_witness AS (
  -- name similar to "Annabel" and on Franklin Ave is Annabel Miller
  SELECT *
  FROM person
  WHERE name LIKE "%Annabel%"
    AND address_street_name = 'Franklin Ave'
),
witnesses AS (
  -- combine first_witness and second_witness
  SELECT person.*
  FROM person, first_witness, second_witness
  WHERE person.id = first_witness.id
    OR person.id = second_witness.id
),
witnesses_interview AS (
  -- what did they say? details:
  -- from witness 1
  -- - member of Get Fit Now
  -- - membership number of bag starts with 48Z and gold member
  -- - plate number like %H42W%
  -- from witness 2
  -- - worked out on 9 Jan
  SELECT witnesses.name, interview.*
  FROM interview
    JOIN witnesses ON interview.person_id = witnesses.id
),
with_48z AS (
  -- from witness 1's report
  SELECT *
  FROM get_fit_now_member
  WHERE id LIKE '48Z%'
    AND membership_status = 'gold'
),
checked_in_at_20180109 AS (
  -- from witness 2's report
  SELECT *
  FROM get_fit_now_check_in
  WHERE check_in_date = 20180109
),
suspect_lifters AS (
  -- from witness 1 and 2's reports
  SELECT with_48z.id AS membership_id, person.*
  FROM person
    JOIN with_48z ON person.id = with_48z.person_id
    JOIN checked_in_at_20180109 ON with_48z.id = checked_in_at_20180109.membership_id
),
suspect_drivers AS (
  -- from witness 2's report
  SELECT drivers_license.plate_number, person.*
  FROM person
    JOIN drivers_license ON person.license_id = drivers_license.id
      AND drivers_license.plate_number LIKE '%H42W%'
),
suspect AS (
  -- combining suspect_lifters and suspect_drivers leads us to one dude
  SELECT person.*
  FROM person
    JOIN suspect_lifters ON person.id = suspect_lifters.id
    JOIN suspect_drivers ON person.id = suspect_drivers.id
),
suspect_interview AS (
  -- now the plot thickens...
  -- I was hired by a woman with a lot of money.
  -- ... she's around 5'5" (65") or 5'7" (67").
  -- She has red hair and she drives a Tesla Model S.
  -- I know that she attended the SQL Symphony Concert 3 times in December 2017.
  SELECT person.name, interview.*
  FROM interview
    JOIN suspect ON interview.person_id = suspect.id
    JOIN person ON interview.person_id = person.id
),
employer_driver AS (
  -- from suspect's description: red hair, Tesla Model S, bet. 65 and 67
  SELECT *
  FROM drivers_license
  WHERE hair_color = 'red'
    AND car_make = 'Tesla'
    AND car_model = 'Model S'
    AND height >= 65
    AND height <= 67
),
employer_symphony AS (
  -- what are these people thinking using facebook_event_checkin?!
  -- anyhow, at 2017 december, of event name SQL Symphony Concert,
  -- and thrice (GROUP BY person_id HAVING count = 3)
  SELECT *, COUNT(*) AS count
  FROM facebook_event_checkin
  WHERE event_name = 'SQL Symphony Concert'
    AND date >= 20171201
    AND date <= 20171231
  GROUP BY person_id
  HAVING count = 3
),
employer AS (
  SELECT person.*
  FROM person
    JOIN employer_driver ON person.license_id = employer_driver.id
    JOIN employer_symphony ON person.id = employer_symphony.person_id
)
SELECT *
FROM suspect, employer
```

This all seems a bit overwhelming.
Let's break this down.

### Determining the details of the murder

```sql
SELECT *
FROM crime_scene_report
WHERE date = 20180115
  AND TYPE = 'murder'
  AND city = 'SQL City'
```

This helps us extract the details of the murder in question.
It returns the following query:

```none
date      | type   | description                                                           | city
20180115  | murder | Security footage shows that there were 2 witnesses.                   | SQL City
          |        | The first witness lives at the last house on "Northwestern Dr".       |
          |        | The second witness, named Annabel, lives somewhere on "Franklin Ave". |
```

Let us save this query as `murder` using `WITH murder AS (SELECT ...)`.

### Extracting witness testimony

#### Identifying the two witnesses

The first witness lives at the last house on "Northwestern Dr".
Assuming that the houses are lined up with increasing house number,
we can just grab the house with the largest address_number.

```sql
SELECT *
FROM person
WHERE address_street_name = 'Northwestern Dr'
  AND address_number = (
    SELECT MAX(address_number)
    FROM person
  )
```

This returns the following query, which we save as `first_witness`:

```none
id    | name            | license_id | address_number | address_street_name | ssn
14887 | Morty Schapiro  | 118009     | 4919           | Northwestern Dr     | 111564949
```

The second witness has a name that should contain "Anabel"
and lives at "Franklin Ave".

```sql
SELECT *
FROM person
WHERE name LIKE "%Annabel%"
  AND address_street_name = 'Franklin Ave'
```

This returns the following query, which we save as `second_witness`:

```none
id    | name           | license_id | address_number | address_street_name | ssn
16371 | Annabel Miller | 490173     | 103            | Franklin Ave        | 318771143
```

Let us combine the two so that we can grab their testimonies easily.
Let us save the result as `witnesses`:

```sql
SELECT person.*
FROM person, first_witness, second_witness
WHERE person.id = first_witness.id
  OR person.id = second_witness.id
```

```none
id    | name           | license_id | address_number | address_street_name | ssn
14887 | Morty Schapiro | 118009     | 4919           | Northwestern Dr     | 111564949
16371 | Annabel Miller | 490173     | 103            | Franklin Ave        | 318771143
```

#### Extracting their testimonies

Now that we have a list of witnesses,
we can now extract the testimony they have given using the `interview` table.
Let us save this query as `witnesses_interview`:

```sql
SELECT witnesses.name, interview.*
FROM interview
  JOIN witnesses ON interview.person_id = witnesses.id
```

```none
name           | person_id | transcript
Morty Schapiro | 14887     | I heard a gunshot and then saw a man run out.
               |           | He had a "Get Fit Now Gym" bag. The membership number on the bag
               |           | started with "48Z". Only gold members have those bags. The man got
               |           | into a car with a plate that included "H42W".
Annabel Miller | 16371     | I saw the murder happen, and I recognized the killer from my gym
               |           | when I was working out last week on January the 9th.
```

Looks like we have a good lead on our hands.
This of course, assumes that they are truthful.

### Finding the suspect

Using the `get_fit_now_member`, `get_fit_now_check_in`, and `drivers_license` tables,
we can narrow down the suspects based on the information provided.

#### Using "Get Fit Now" data

Let us find which ones have a membership number starting with `48Z`
and a `gold` membership status.
Let us save this as `with_48z`:

```sql
SELECT *
FROM get_fit_now_member
WHERE id LIKE '48Z%'
  AND membership_status = 'gold'
```

```none
id    | person_id | name          | membership_start_date | membership_status
48Z7A | 28819     | Joe Germuska  | 20160305              | gold
48Z55 | 67318     | Jeremy Bowers | 20160101              | gold
```

Using the information Miller provided,
let us now find which ones have been at the gym at the 9th of Jsanuary.
Let us save this as `checked_in_at_20180109`:

```sql
SELECT *
FROM get_fit_now_check_in
WHERE check_in_date = 20180109
```

```none
membership_id | check_in_date | check_in_time | check_out_time
X0643         | 20180109      | 957           | 1164
UK1F2         | 20180109      | 344           | 518
XTE42         | 20180109      | 486           | 1124
1AE2H         | 20180109      | 461           | 944
6LSTG         | 20180109      | 399           | 515
7MWHJ         | 20180109      | 273           | 885
GE5Q8         | 20180109      | 367           | 959
48Z7A         | 20180109      | 1600          | 1730
48Z55         | 20180109      | 1530          | 1700
90081         | 20180109      | 1600          | 1700
```

Now let us combine the data from `checked_in_at_20180109` and `with_48z`
and save it as `suspect_lifters`:

```sql
SELECT with_48z.id AS membership_id, person.*
FROM person
  JOIN with_48z ON person.id = with_48z.person_id
  JOIN checked_in_at_20180109 ON with_48z.id = checked_in_at_20180109.membership_id
```

```none
membership_id | id    | name          | license_id | address_number | address_street_name   | ssn
48Z7A         | 28819 | Joe Germuska  | 173289     | 111            | Fisk Rd               | 138909730
48Z55         | 67318 | Jeremy Bowers | 423327     | 530            | Washington Pl, Apt 3A | 871539279
```

Uh oh. Both of them have the same starting membership ID,
are both gold members, and checked in at that date.

#### Using driver's license data

Recall that Schapiro testified that the plate number he used has `H42W` in it.
Let us look for all people with that license number.

```sql
SELECT drivers_license.plate_number, person.*
FROM person
  JOIN drivers_license ON person.license_id = drivers_license.id
    AND drivers_license.plate_number LIKE '%H42W%'
```

Let us save this result as `suspect_drivers`:

```none
plate_number | id    | name           | license_id | address_number | address_street_name   | ssn
4H42WR       | 51739 | Tushar Chandra | 664760     | 312            | Phi St                | 137882671
0H42W2       | 67318 | Jeremy Bowers  | 423327     | 530            | Washington Pl, Apt 3A | 871539279
H42W0X       | 78193 | Maxine Whitely | 183779     | 110            | Fisk Rd               | 137882671
```

Using `suspect_lifters` and `suspect_drivers`,
let us find who the suspect is and save the result as `suspect`:

```sql
SELECT person.*
FROM person
  JOIN suspect_lifters ON person.id = suspect_lifters.id
  JOIN suspect_drivers ON person.id = suspect_drivers.id
```

```none
id    | name          | license_id  | address_number | address_street_name   | ssn
67318 | Jeremy Bowers | 423327      | 530            | Washington Pl, Apt 3A | 871539279
```

Bingo.
Let us put his name in the "Check your solution" section:

```sql
INSERT INTO solution VALUES (1, 'Jeremy Bowers');
SELECT value FROM solution;
```

```none
value
Congrats, you found the murderer! But wait, there's more... If you think you're up for a challenge,
try querying the interview transcript of the murderer to find the real villain behind this crime.
If you feel especially confident in your SQL skills, try to complete this final step with no more
than 2 queries. Use this same INSERT statement with your new suspect to check your answer.
```

Well... guess we're not done yet.

### Finding the employer

What did Bowers tell the coppers?

```sql
SELECT person.name, interview.*
FROM interview
  JOIN suspect ON interview.person_id = suspect.id
  JOIN person ON interview.person_id = person.id
```

```none
name          | person_id | transcript
Jeremy Bowers | 67318     | I was hired by a woman with a lot of money.
              |           | I don't know her name but I know she's around 5'5" (65") or 5'7" (67").
              |           | She has red hair and she drives a Tesla Model S.
              |           | I know that she attended the SQL Symphony Concert 3 times in December 2017.
```

Guess we have to look for the employer now.

The `drivers_license` table contains data regarding peoples' heights and what cars they drive.
We are looking for a person with red hair, between 65 to 67 inches of height, and has a red Tesla Model S.
Let us save this query as `employer_driver`:

```sql
SELECT *
FROM drivers_license
WHERE hair_color = 'red'
  AND car_make = 'Tesla'
  AND car_model = 'Model S'
  AND height >= 65
  AND height <= 67
```

```none
id     | age | height | eye_color | hair_color | gender | plate_number | car_make | car_model
202298 | 68  | 66     | green     | red        | female | 500123       | Tesla    | Model S
291182 | 65  | 66     | blue      | red        | female | 08CM64       | Tesla    | Model S
918773 | 48  | 65     | black     | red        | female | 917UU3       | Tesla    | Model S
```

We also know that the suspect attended an event named "SQL Symphony Concert 3"
thrice in December 2017.
Fortunately we can use the `facebook_event_checkin` table.
Let us save this query as `employer_symphony`:

```sql
SELECT *, COUNT(*) AS count
FROM facebook_event_checkin
WHERE event_name = 'SQL Symphony Concert'
  AND date >= 20171201
  AND date <= 20171231
GROUP BY person_id
HAVING count = 3
```

```none
person_id | event_id | event_name           | date     | count
24556     | 1143     | SQL Symphony Concert | 20171224 | 3
99716     | 1143     | SQL Symphony Concert | 20171229 | 3
```

Using `employer_symphony` and `employer_driver`,
let us now find who our employer is and save the result as `employer`:

```sql
SELECT person.*
FROM person
  JOIN employer_driver ON person.license_id = employer_driver.id
  JOIN employer_symphony ON person.id = employer_symphony.person_id
```

```none
id    | name             | license_id | address_number | address_street_name | ssn
99716 | Miranda Priestly | 202298     | 1883           | Golden Ave          | 987756388
```

And let us place the name in our "Check your solution" section:

```sql
INSERT INTO solution VALUES (1, 'Miranda Priestly');
SELECT value FROM solution;
```

```none
value
Congrats, you found the brains behind the murder! Everyone in SQL City hails you
as the greatest SQL detective of all time. Time to break out the champagne!
```

Well boys, we did it.
The employer is no more.
