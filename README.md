# Graphinnati

A PHP and MySQL web app for exploring census statistics about Cincinnati's 50 neighborhoods. Signed-in users tick the neighborhoods they care about, pick a statistic (population, gender, race or education level) and get an instant bar graph or pie chart drawn with CanvasJS.

The neighborhood data ships with the repo in `X.csv` and comes from the Open Cincy data portal, according to the site's home page copy.

## Features

- **Account sign-up and login**: `createAccount.php` collects first name, last name, phone, username and password; passwords are hashed with `password_hash` (bcrypt) and checked with `password_verify` in `handleLogin.php`. Sessions track the signed-in user, and `logout.php` ends the session.
- **Protected graph builder** (`data_page.php`): redirects to the login page unless you're signed in.
  - 50 neighborhood checkboxes, from Avondale to Winton Hills, with per-row and "select all" helpers.
  - 16 statistics to graph: total, male and female population; six race categories; and seven education levels from "less than 9th grade" to "college graduate".
  - Bar graph or pie chart output, animated and rendered client-side with [CanvasJS](https://canvasjs.com/).
- **Home page** (`index.php`) with an image carousel of sample graphs and a short pitch for the tool.
- **Contact form** (`contact.php`) that emails name, email, phone and message to the site owner with PHP `mail()`.

## How it works

1. `data_page.php` connects to a local MySQL database named `Graphinnati` and runs `SELECT * FROM Stats`.
2. PHP turns each statistic column into an array and writes it into the page with `json_encode`, so all the data is available to JavaScript up front.
3. When you click **Create Graph**, `createGraph()` reads the checked neighborhoods and the chosen statistic, builds a CanvasJS data series, and calls `createBarGraph()` or `createPieChart()` to render it into `#chartContainer`.

## Tech stack

- PHP 5 using the legacy `mysql_*` extension, with sessions for auth
- MySQL (database `Graphinnati`, tables `Accounts` and `Stats`)
- Bootstrap 3, jQuery 1.11, bootstrap-multiselect, bootstrap-select and bootstrap-datetimepicker
- CanvasJS for charts (`js/canvasjs.min.js`)

## Getting started

### Prerequisites

- **PHP 5.5 or 5.6.** The code calls `mysql_connect` and friends, which were removed in PHP 7, so it won't run on a modern PHP without porting to `mysqli` or PDO.
- MySQL listening on the socket `/tmp/mysql.sock` with user `root` and an empty password (these values are hard-coded in `handleLogin.php`, `insertProfile.php` and `data_page.php`).

### Database setup

The repo has no SQL schema file. These tables match the columns the code reads and writes:

```sql
CREATE DATABASE Graphinnati;
USE Graphinnati;

CREATE TABLE Accounts (
  FirstName   VARCHAR(100),
  LastName    VARCHAR(100),
  PhoneNumber VARCHAR(30),
  Username    VARCHAR(100) PRIMARY KEY,
  Password    VARCHAR(255)   -- bcrypt hash
);

CREATE TABLE Stats (
  NeighborhoodID   INT PRIMARY KEY,
  Neighborhood     VARCHAR(100),
  Population       INT, MalePopulation INT, FemalePopulation INT,
  RaceWhite INT, RaceBlack INT, RaceIndian INT, RaceAsian INT, RacePacific INT, RaceOther INT,
  EducationA INT, EducationB INT, EducationC INT, EducationD INT,
  EducationE INT, EducationF INT, EducationG INT
);
```

Then load `X.csv` into `Stats`. The file uses old Mac-style `\r` line endings, so tell MySQL about them:

```sql
LOAD DATA LOCAL INFILE 'X.csv' INTO TABLE Stats
  FIELDS TERMINATED BY ','
  LINES TERMINATED BY '\r'
  IGNORE 1 LINES;
```

Row order matters: the checkboxes on `data_page.php` use values 1 to 50 that index into the arrays in `NeighborhoodID` order.

### Running

```bash
git clone https://github.com/trey-a-hope/graphinnati.git
cd graphinnati
php -S localhost:8000
```

Open <http://localhost:8000/index.php>, create an account, sign in and go to **Create Graph**.

## Project structure

```
graphinnati/
├── index.php          # Home page: carousel and feature blurbs
├── login.php          # Sign-in form → handleLogin.php
├── handleLogin.php    # Verifies the bcrypt password and starts the session
├── createAccount.php  # Sign-up form → insertProfile.php
├── insertProfile.php  # Inserts a new row into Accounts
├── logout.php         # Destroys the session
├── data_page.php      # Graph builder: loads Stats, renders charts with CanvasJS
├── contact.php        # Contact form that sends email with mail()
├── X.csv              # Neighborhood census data (50 rows)
├── css/, js/          # Bootstrap, plugins, CanvasJS and custom styles
└── images/            # Backgrounds, logo and sample graph screenshots
```

## Known limitations

- SQL queries are built by string concatenation with raw `$_POST` values, so the login and sign-up forms are open to SQL injection. Use prepared statements before putting this online.
- Database credentials are hard-coded in three files instead of one config file.
- Status messages are passed as raw HTML in the `?message=` query string and echoed back, which allows reflected XSS.
- Dreamweaver sync files (`css/_notes`, `js/_notes`) are committed and can be removed.
- There's no license file in the repo.
