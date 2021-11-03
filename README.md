[![View My Profile](https://img.shields.io/badge/View-My_Profile-blue?logo=GitHub)](https://github.com/cholu6768) 

# The SQL Murder Mystery 

![picture](picture.png)

> This case can be found on https://mystery.knightlab.com/

# Table of Contents

- [🕵️‍♂️ About](#about)
- [📋 Schema Diagram](#schema-diagram)
- [🔍 Solving the case](#solving-the-case)
  - [Crime Scene Report](#crime-scene-report)
  - [Finding The Witnesses](#finding-the-witnesses)
  - [Witnesses Interviews](#witnesses-interviews)
  - [Analysis First Statement](#analysis-first-statement)
  - [Analysis Second Statement](#analysis-second-statement)
  - [Catching The Murderer](#catching-the-murderer)
  - [Murderer Statement](#murderer-statement)
  - [Finding The Real Villain](#finding-the-real-villain)
  - [Catching The Real Villain](#catching-the-real-villain)

# 🕵️‍♂️ About
A crime has taken place and the detective needs your help. The detective gave you the crime scene report, but you somehow lost it. You vaguely remember that the crime was a ​murder​ that occurred sometime on ​Jan.15, 2018​ and that it took place in ​SQL City​.

# 📋 Schema Diagram
A schema was provided to have a better understanding on where to look for clues.

![erd](erd.png)


# 🔍 Solving the case

## Crime Scene Report

First, let's retrive the corresponding crime scene report.

```sql
SELECT 
  date,
  type,
  description,
  city
FROM crime_scene_report
WHERE 
  city = 'SQL City' AND 
  type = 'murder' AND
  date = '20180115';
```    

| date     | type   | description                                                                                                                                                                               | city     |
|----------|--------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|----------|
| 20180115 | murder | Security footage shows that there were 2 witnesses. The first witness lives at the last house on "Northwestern Dr". The second witness, named Annabel, lives somewhere on "Franklin Ave". | SQL City |

## Finding The Witnesses

Now, let's find the full name and ``id`` of the first witness.

```sql
SELECT 
  id,
  name,
  address_number,
  address_street_name AS str_name
FROM person
WHERE str_name = 'Northwestern Dr'
ORDER BY address_number DESC
LIMIT 5;
```
| id    | name           | address_number | str_name        |
|-------|----------------|----------------|-----------------|
| 14887 | Morty Schapiro | 4919           | Northwestern Dr |

**The first witness is Morty Schapiro and has the ``id`` 14887.**

Next, let's look for the ``id`` and last name of the second witness.

```sql
SELECT 
  id,
  name,
  address_number,
  address_street_name AS str_name
FROM person
WHERE 
  str_name = 'Franklin Ave' AND
  name LIKE 'Annabel%';
```
| id    | name           | address_number | str_name      |
|-------|----------------|----------------|---------------|
| 16371 | Annabel Miller | 103            | Franklin Ave. |

**Her full name is Annabel Miller and her ``id`` is 16371.**

## Witnesses Interviews

Let's use the IDs of the witnesses to look for their interviews regarding the murder.

```sql
SELECT 
  person_id,
  transcript
FROM interview
WHERE 
  person_id = 14887 OR person_id = 16371;
```    

``Mr.Schapiro (First witness) said: "I heard a gunshot and then saw a man run out. He had a "Get Fit Now Gym" bag. The membership number on the bag started with "48Z". Only gold members have those bags. The man got into a car with a plate that included "H42W"."``

``Ms.Miller (Second witness) said: "I saw the murder happen, and I recognized the killer from my gym when I was working out last week on January the 9th."``

## Analysis First Statement

Let's focus on finding the information that Mr.Schapiro (First witness) stated.

First let's find the gym's member name whose id starts with '48Z'.

```sql
SELECT 
  id,
  person_id,
  name,
  membership_start_date,
  membership_status
FROM get_fit_now_member
WHERE id LIKE '48Z%';
```
| id    | person_id | name          | membership_start_date | membership_status |
|-------|-----------|---------------|-----------------------|-------------------|
| 48Z38 | 49550     | Tomas Baisley | 20170203              | silver            |
| 48Z7A | 28819     | Joe Germuska  | 20160305              | gold              |
| 48Z55 | 67318     | Jeremy Bowers | 20160101              | gold              |

**3 members were found whose ids start with '48Z'.** 

There were two **gold members**:
- **Joe Germuska** with membership id = 48Z7A and person id = 28819.
- **Jeremy Bowers** with membership id = 48Z55 and person id = 67318.

There was also a **silver member**:
- **Tomas Baisley** with membership id = 48Z38 and a person id = 49550.

Next, let's find the owner of the car whose car plate included 'H42W'.

```sql

SELECT 
  id,
  age,
  height,
  eye_color,
  hair_color,
  gender,
  plate_number,
  car_make,
  car_model
FROM drivers_license
WHERE plate_number LIKE '%H42W%';
```

| id     | age | height | eye_color | hair_color | gender | plate_number | car_make  | car_model |
|--------|-----|--------|-----------|------------|--------|--------------|-----------|-----------|
| 183779 | 21  | 65     | blue      | blonde     | female | H42W0X       | Toyota    | Prius     |
| 423327 | 30  | 70     | brown     | brown      | male   | 0H42W2       | Chevrolet | Spark LS  |
| 664760 | 21  | 71     | black     | black      | male   | 4H42WR       | Nissan    | Altima    |

**3 people with ``license id`` 183779, 423327, 664760 had a car plate number that included 'H42W'.**

Now, let's find the names using the license IDs found on the previous query. 

```sql
SELECT
  id,
  name,
  license_id,
  address_number,
  address_street_name AS str_name
FROM person
WHERE license_id IN (183779, 423327, 664760);
```

| id    | name           | license_id | address_number | str_name              |
|-------|----------------|------------|----------------|-----------------------|
| 51739 | Tushar Chandra | 664760     | 312            | Phi St                |
| 67318 | Jeremy Bowers  | 423327     | 530            | Washington Pl, Apt 3A |
| 78193 | Maxine Whitely | 183779     | 110            | Fisk Rd               |

**One of the ``license_id`` matched the name 'Jeremy Bowers' who was also found to be a gold gym member.**

## Analysis Second Statement

Let's look into the what Ms.Miller (second witness) said.

Based on what was found from the statement of Mr.Schapiro let's see if 'Jeremy Bowers' was in the gym when Ms.Miller was there on the 9th of january.

```sql
SELECT 
  checkin.membership_id,
  members.name,
  checkin.check_in_date,
  checkin.check_in_time,
  checkin.check_out_time
FROM get_fit_now_member AS members
INNER JOIN get_fit_now_check_in AS checkin
  ON members.id = checkin.membership_id
WHERE 
  check_in_date = '20180109' AND 
  name = 'Jeremy Bowers';
```    

| membership_id | name          | check_in_date | check_in_time | check_out_time |
|---------------|---------------|---------------|---------------|----------------|
| 48Z55         | Jeremy Bowers | 20180109      | 1530          | 1700           |

**Jeremy Bowers was indeed in the gym on the 9th of january.**

## Catching The Murderer

The statements given by the witnesses match Jeremy Bowers as the murderer since Mr.Bowers:
- Was a gym gold member and had a ``membership_id`` that started with '48Z'
- His car plate number included 'H42W'
- He was in the gym on the 9th of january of 2018.

Let's check if Jeremy Bowers is behind the murder

```sql
INSERT INTO solution VALUES (1, 'Jeremy Bowers');
        
SELECT value FROM solution;
```        

After checking the solution and putting the name of the suspect the follwing message was shown:

``"Congrats, you found the murderer! But wait, there's more... If you think you're up for a challenge, try querying the interview transcript of the murderer to find the real villain behind this crime. If you feel especially confident in your SQL skills, try to complete this final step with no more than 2 queries. Use this same INSERT statement with your new suspect to check your answer."``

Let's look into who is behind the murder!

## Murderer Statement 

First let's see the interview transcript from Jeremy Bowers (Murderer). 

```sql
SELECT 
  person.id,
  person.name,
  interview.transcript
FROM person
INNER JOIN interview
  ON person.id = interview.person_id
WHERE 
  person.name = 'Jeremy Bowers';
```    

``His statement was: "I was hired by a woman with a lot of money. I don't know her name but I know she's around 5'5" (65") or 5'7" (67"). She has red hair and she drives a Tesla Model S. I know that she attended the SQL Symphony Concert 3 times in December 2017."``

## Finding The Real Villain

The join of the  ``person``, ``drivers_license``, ``facebook_event_checkin`` and ``income`` tables, helped on getting the name, income and the people who assisted the SQL Symphony. The person's description given by the murderer's statement was also used to filter the data.

```sql
SELECT 
  t1.id,
  t1.name,
  t1.ssn,
  t4.annual_income,
  t3.event_name,
  t3.date
FROM person AS t1
INNER JOIN drivers_license AS t2
  ON t1.license_id = t2.id
INNER JOIN facebook_event_checkin AS t3
  ON t1.id = t3.person_id
INNER JOIN income AS t4
  ON t1.ssn = t4.ssn
WHERE 
  t2.car_make = 'Tesla' AND
  t2.car_model = 'Model S' AND
  t2.gender = 'female' AND 
  t2.hair_color = 'red'	
ORDER BY t1.id;
```

| id    | name             | ssn       | annual_income | event_name           | date     |
|-------|------------------|-----------|---------------|----------------------|----------|
| 99716 | Miranda Priestly | 987756388 | 310000        | SQL Symphony Concert | 20171206 |
| 99716 | Miranda Priestly | 987756388 | 310000        | SQL Symphony Concert | 20171212 |
| 99716 | Miranda Priestly | 987756388 | 310000        | SQL Symphony Concert | 20171229 |

**The output of the query showed only one person that is a female, has red hair, drives a "Tesla Model S", has a big income and assisted 3 times in December 2017 to the SQL Symphony Concert.**

**Her name is Miranda Priestly.**

## Catching The Real Villain

Let's check if Miranda Priestly is behind the murder!

```sql
INSERT INTO solution VALUES (1, 'Miranda Priestly');
        
SELECT value FROM solution;
```

After checking the solution and putting the name of the suspect the follwing message was shown:

``"Congrats, you found the brains behind the murder! Everyone in SQL City hails you as the greatest SQL detective of all time. Time to break out the champagne!"``