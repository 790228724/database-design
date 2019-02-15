# principles

- use singular noun
- use uuid if possible
- use soft delete
- no null fields, except date

# Notes

- inner joins is faster that subquery most of the time
- apply logic in the database if you are going to have many different applications
- use optimized uuid for faster querying
- include the default auto incremented id


## Useful Statements


```sql
-- Sets a default created date
created_at datetime     NOT NULL DEFAULT CURRENT_TIMESTAMP

-- Sets a default updated date, and updates it whenever a row is updated
updated_at datetime     NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP

-- Adds a new constraint that checks if the first user id is smaller than the second.
CONSTRAINT check_one_way CHECK (user_id1 < user_id2)

-- Adds a constraint that checks if the combination of both columns is unique.
CONSTRAINT uq_user_id_1_user_id_2 UNIQUE (user_id1, user_id2)

-- Sets a foreign key constraint, and updates the foreign key when the primary key changes, or delete the row when it is deleted.
FOREIGN KEY (user_id1) REFERENCES user (id) ON UPDATE CASCADE ON DELETE CASCADE

-- Sets a foreign key constraint, and updates the foreign key when the primary key changes, or set the foreign key to null when it is deleted.
FOREIGN KEY (relationship) REFERENCES ref_relationship (status) ON UPDATE CASCADE ON DELETE SET NULL

-- Create a composite primary key from two columns.
PRIMARY KEY (user_id1, user_id2)
```


Sorting:

```sql
-- Sorting integer string in the correct order
SELECT * FROM <table> ORDER BY CAST(<column> AS unsigned)
```
## Data Type: IPV4 and IPV6

You have two possibilities (for an IPv4 address) :

- a `varchar(15)`, if your want to store the IP address as a string. e.g. `192.128.0.15` for instance
- an `integer (4 bytes)`, if you convert the IP address to an integer. e.g. `3229614095` for the IP I used before

```sql
`ipv4` INT UNSIGNED
INSERT INTO `table` (`ipv4`) VALUES (INET_ATON("127.0.0.1"));
SELECT INET_NTOA(`ipv4`) FROM `table`;
```

```sql
`ipv6` VARBINARY(16)
INSERT INTO `table` (`ipv6`) VALUES (INET6_ATON("127.0.0.1"));
SELECT INET6_NTOA(`ipv6`) FROM `table`;
```

To use a single column for both IPV4 and IPV6:

```sql
CREATE TABLE `sensor` (
  `ip` varbinary(16) NOT NULL DEFAULT '0x'
)
-- Insert IPv6.
insert into sensor (ip) values (INET6_ATON("2001:0db8:85a3:0000:0000:8a2e:0370:7334"));

-- Insert IPv4.
insert into sensor (ip) values (INET6_ATON("255.255.255.0"));

select INET6_NTOA(ip) from sensor;
+------------------------------+
| INET6_NTOA(ip)               |
+------------------------------+
| 2001:db8:85a3::8a2e:370:7334 |
| 255.255.255.0                |
+------------------------------+
2 rows in set (0.00 sec)
```

## Data Type: Country
Al Jumahiriyah al Arabiyah al Libiyah ash Shabiyah al Ishtirakiyah al Uzma also known as Libya is the world's longest country name at 74 characters with spaces and 63 characters without.

```sql
country varchar(74) NOT NULL DEFAULT ''
```

## Data Type: Address

```sql
address_line_1 VARCHAR(255) NOT NULL DEFAULT '',
address_line_2 VARCHAR(255) NOT NULL DEFAULT '',

-- Longest city name: Llanfairpwllgwyngyllgogerychwyrndrobwllllantysiliogogogoch (58 chars.)
city VARCHAR(58) NOT NULL DEFAULT '',

-- Longest state name: The State of Rhode Island and Providence Plantations (52 chars.)
state VARCHAR(56) NOT NULL DEFAULT '',
postal_code VARCHAR(16) NOT NULL DEFAULT '',
country VARCHAR(74) NOT NULL DEFAULT '',
```

References:
- postal code length https://stackoverflow.com/questions/325041/i-need-to-store-postal-codes-in-a-database-how-big-should-the-column-be

## Data Type: URL

```sql
url varchar(2083) NOT NULL DEFAULT '';
```

Checking can be done at the application side. If the length exceeded that of 2083, just warn the client or suggest them to use a url shortener.

References: https://stackoverflow.com/questions/219569/best-database-field-type-for-a-url


## Data Type: Email

https://stackoverflow.com/questions/8242567/acceptable-field-type-and-size-for-email-address

```sql
email VARCHAR(255) NOT NULL UNIQUE
```
## Data Type: Geolocation


## Data Type: TZ

Max length of 32, longest is `America/Argentina/ComodRivadavia`:

```sql
zoneinfo VARCHAR(32) COMMENT "String from zoneinfo [zoneinfo] time zone database representing the End-User's time zone. For example, Europe/Paris or America/Los_Angeles"
```
References: 
- https://stackoverflow.com/questions/12546312/max-length-of-tzname-field-timezone-identifier-name
- https://www.iana.org/time-zones

## Data Type: Locale

BCP47/RFC5646 section 4.4.1 recommends a 35 characters tag length:

```sql
locale VARCHAR(35) NOT NULL DEFAULT '' COMMENT "End-User's locale, represented as a BCP47 [RFC5646] language tag. This is typically an ISO 639-1 Alpha-2 [ISO639?1] language code in lowercase and an ISO 3166-1 Alpha-2 [ISO3166?1] country code in uppercase, separated by a dash. For example, en-US or fr-CA. As a compatibility note, some implementations have used an underscore as the separator rather than a dash, for example, en_US",
```

References:
- https://stackoverflow.com/questions/17848070/what-data-type-should-i-use-for-ietf-language-codes
- https://openid.net/specs/openid-connect-core-1_0.html#zoneinfo

## Data Type: Phone number

```sql
phone_number VARCHAR(32) NOT NULL DEFAULT '',
phone_number_verified BOOLEAN NOT NULL DEFAULT 0,
```
References:
- https://en.wikipedia.org/wiki/Telephone_numbering_plan
- https://boards.straightdope.com/sdmb/showthread.php?t=417024
https://stackoverflow.com/questions/723587/whats-the-longest-possible-worldwide-phone-number-i-should-consider-in-sql-varc


## One-to-One Relationship

Example of 1-to-1 relationship between `user` and `preference` table:

```
CREATE TABLE IF NOT EXISTS user (
  name VARCHAR(255),
  id INT UNSIGNED AUTO_INCREMENT,
  PRIMARY KEY (id)
);

CREATE TABLE IF NOT EXISTS preference (
  user_id INT UNSIGNED AUTO_INCREMENT,
  interest TEXT,
  -- ...other fields
  PRIMARY KEY (id),
  FOREIGN KEY (id) REFERENCES user(id)
);
```
