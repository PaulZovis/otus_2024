# Строим модель данных

```
CREATE TABLE `dct_city` (
	`city_id` INT(11) NOT NULL AUTO_INCREMENT,
	`city_name` VARCHAR(400) NULL DEFAULT NULL COLLATE 'utf8mb4_0900_ai_ci',
	`city_desc` VARCHAR(1000) NULL DEFAULT NULL COLLATE 'utf8mb4_0900_ai_ci',
	PRIMARY KEY (`city_id`) USING BTREE,
	UNIQUE INDEX `city_name` (`city_name`) USING BTREE
)
COLLATE='utf8mb4_0900_ai_ci'
ENGINE=InnoDB
;

CREATE TABLE `dct_country` (
	`country_id` INT(11) NOT NULL AUTO_INCREMENT,
	`country_name` VARCHAR(200) NOT NULL COLLATE 'utf8mb4_0900_ai_ci',
	`country_desc` VARCHAR(1000) NULL DEFAULT NULL COLLATE 'utf8mb4_0900_ai_ci',
	PRIMARY KEY (`country_id`) USING BTREE,
	UNIQUE INDEX `country_name` (`country_name`) USING BTREE
)
COLLATE='utf8mb4_0900_ai_ci'
ENGINE=InnoDB
;

CREATE TABLE `dct_gender` (
	`gender_id` INT(11) NOT NULL AUTO_INCREMENT,
	`gender_name` VARCHAR(10) NULL DEFAULT NULL COLLATE 'utf8mb4_0900_ai_ci',
	`gender_desc` VARCHAR(1000) NULL DEFAULT NULL COLLATE 'utf8mb4_0900_ai_ci',
	PRIMARY KEY (`gender_id`) USING BTREE,
	UNIQUE INDEX `gender_name` (`gender_name`) USING BTREE
)
COLLATE='utf8mb4_0900_ai_ci'
ENGINE=InnoDB
;

CREATE TABLE `dct_language` (
	`language_id` INT(11) NOT NULL AUTO_INCREMENT,
	`language_name` VARCHAR(200) NOT NULL COLLATE 'utf8mb4_0900_ai_ci',
	`language_desc` VARCHAR(1000) NULL DEFAULT NULL COLLATE 'utf8mb4_0900_ai_ci',
	PRIMARY KEY (`language_id`) USING BTREE,
	UNIQUE INDEX `language_name` (`language_name`) USING BTREE
)
COLLATE='utf8mb4_0900_ai_ci'
ENGINE=InnoDB
;

CREATE TABLE `dct_region` (
	`region_id` INT(11) NOT NULL AUTO_INCREMENT,
	`country_id` INT(11) NOT NULL,
	`region_name` VARCHAR(100) NULL DEFAULT NULL COLLATE 'utf8mb4_0900_ai_ci',
	PRIMARY KEY (`region_id`) USING BTREE,
	INDEX `country_id` (`country_id`) USING BTREE,
	CONSTRAINT `dct_region_ibfk_1` FOREIGN KEY (`country_id`) REFERENCES `dct_country` (`country_id`) ON UPDATE NO ACTION ON DELETE NO ACTION
)
COLLATE='utf8mb4_0900_ai_ci'
ENGINE=InnoDB
;

CREATE TABLE `dct_street` (
	`street_id` INT(11) NOT NULL AUTO_INCREMENT,
	`street_name` VARCHAR(400) NULL DEFAULT NULL COLLATE 'utf8mb4_0900_ai_ci',
	`street_desc` VARCHAR(1000) NULL DEFAULT NULL COLLATE 'utf8mb4_0900_ai_ci',
	PRIMARY KEY (`street_id`) USING BTREE,
	UNIQUE INDEX `street_name` (`street_name`) USING BTREE
)
COLLATE='utf8mb4_0900_ai_ci'
ENGINE=InnoDB
;

CREATE TABLE `dct_title` (
	`title_id` INT(11) NOT NULL AUTO_INCREMENT,
	`title_name` VARCHAR(10) NULL DEFAULT NULL COLLATE 'utf8mb4_0900_ai_ci',
	`title_desc` VARCHAR(1000) NULL DEFAULT NULL COLLATE 'utf8mb4_0900_ai_ci',
	PRIMARY KEY (`title_id`) USING BTREE,
	UNIQUE INDEX `title_name` (`title_name`) USING BTREE
)
COLLATE='utf8mb4_0900_ai_ci'
ENGINE=InnoDB
;

CREATE TABLE `users` (
	`user_id` INT(11) NOT NULL AUTO_INCREMENT,
	`title_id` INT(11) NOT NULL,
	`first_name` VARCHAR(100) NULL DEFAULT NULL COLLATE 'utf8mb4_0900_ai_ci',
	`last_name` VARCHAR(100) NULL DEFAULT NULL COLLATE 'utf8mb4_0900_ai_ci',
	`birth_date` DATE NULL DEFAULT NULL,
	`gender_id` INT(11) NOT NULL,
	`marital_status` VARCHAR(100) NULL DEFAULT NULL COLLATE 'utf8mb4_0900_ai_ci',
	`language_id` INT(11) NOT NULL,
	PRIMARY KEY (`user_id`) USING BTREE,
	INDEX `title_id` (`title_id`) USING BTREE,
	INDEX `gender_id` (`gender_id`) USING BTREE,
	INDEX `language_id` (`language_id`) USING BTREE,
	CONSTRAINT `users_ibfk_1` FOREIGN KEY (`title_id`) REFERENCES `dct_title` (`title_id`) ON UPDATE NO ACTION ON DELETE NO ACTION,
	CONSTRAINT `users_ibfk_2` FOREIGN KEY (`gender_id`) REFERENCES `dct_gender` (`gender_id`) ON UPDATE NO ACTION ON DELETE NO ACTION,
	CONSTRAINT `users_ibfk_3` FOREIGN KEY (`language_id`) REFERENCES `dct_language` (`language_id`) ON UPDATE NO ACTION ON DELETE NO ACTION
)
COLLATE='utf8mb4_0900_ai_ci'
ENGINE=InnoDB
;

CREATE TABLE `addresses` (
	`address_id` INT(11) NOT NULL AUTO_INCREMENT,
	`user_id` INT(11) NOT NULL,
	`country_id` INT(11) NOT NULL,
	`region_id` INT(11) NULL DEFAULT NULL,
	`city_id` INT(11) NOT NULL,
	`street` VARCHAR(200) NULL DEFAULT NULL COLLATE 'utf8mb4_0900_ai_ci',
	`building_number` VARCHAR(50) NULL DEFAULT NULL COLLATE 'utf8mb4_0900_ai_ci',
	`postal_code` VARCHAR(20) NULL DEFAULT NULL COLLATE 'utf8mb4_0900_ai_ci',
	PRIMARY KEY (`address_id`) USING BTREE,
	INDEX `user_id` (`user_id`) USING BTREE,
	INDEX `country_id` (`country_id`) USING BTREE,
	INDEX `region_id` (`region_id`) USING BTREE,
	INDEX `city_id` (`city_id`) USING BTREE,
	CONSTRAINT `addresses_ibfk_1` FOREIGN KEY (`user_id`) REFERENCES `users` (`user_id`) ON UPDATE NO ACTION ON DELETE NO ACTION,
	CONSTRAINT `addresses_ibfk_2` FOREIGN KEY (`country_id`) REFERENCES `dct_country` (`country_id`) ON UPDATE NO ACTION ON DELETE NO ACTION,
	CONSTRAINT `addresses_ibfk_3` FOREIGN KEY (`region_id`) REFERENCES `dct_region` (`region_id`) ON UPDATE NO ACTION ON DELETE NO ACTION,
	CONSTRAINT `addresses_ibfk_4` FOREIGN KEY (`city_id`) REFERENCES `dct_city` (`city_id`) ON UPDATE NO ACTION ON DELETE NO ACTION
)
COLLATE='utf8mb4_0900_ai_ci'
ENGINE=InnoDB
;
```

+------------------+       +------------------+
|    dct_country   |       |     dct_city    |
+------------------+       +------------------+
| country_id (PK)  |<-----| city_id (PK)     |
| country_name     |       | city_name       |
| country_desc     |       | city_desc       |
+------------------+       +------------------+

+------------------+       +------------------+
|    dct_region    |       |     dct_gender  |
+------------------+       +------------------+
| region_id (PK)   |       | gender_id (PK)  |
| country_id (FK)  |<-----| gender_name      |
| region_name      |       | gender_desc     |
+------------------+       +------------------+

+------------------+       +------------------+
|      users       |       |     dct_title   |
+------------------+       +------------------+
| user_id (PK)     |       | title_id (PK)   |
| title_id (FK)    |<-----| title_name      |
| gender_id (FK)   |       | title_desc      |
| language_id (FK) |       +------------------+
+------------------+

+------------------+       +------------------+
|    addresses     |       |  dct_language   |
+------------------+       +------------------+
| address_id (PK)  |       | language_id (PK)|
| user_id (FK)     |<-----| language_name   |
| country_id (FK)  |       | language_desc   |
| region_id (FK)   |       +------------------+
| city_id (FK)     |
+------------------+
