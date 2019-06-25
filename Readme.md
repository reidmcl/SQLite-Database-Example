Reid McLaughlin

###This provides an outline of how to create a database in SQLite. Specifically this database provides information on all matches of the 2013-14 English Premier League season. 
First, download SQLite on to your computer. Download the two CSV files that will be used as data sources. Please see the database diagram that outlines the structure of the database.

**First** - create a "New Database" in DB Browser for SQLite. Title it whatever you want - perhaps something like "EPL_13-14_Season" 

**Second** - import the two CSV files. File -> Import -> Table from CSV file

13-14 EPL Data - Import CSV file as a table named “EPL_DATA”
PremiershipFootballGrounds.CSV - Import CSV files as a table named “PremiershipFootballGrounds”

**Third** - The follow commands create the database's four tables. Run them in the "Execute SQL" tab in DB Browser for SQLite.

Creation of 4 tables:

`CREATE TABLE Matchs 
	(MatchID INTEGER NOT NULL,
	HomeTeamID	INTEGER NOT NULL,
	AwayTeamID	INTEGER NOT NULL,
	FTHomeTeamGoals INTEGER NOT NULL,
	FTAwayTeamGoals	INTEGER NOT NULL,
	FullTimeResult TEXT NOT NULL,
	RefereeID	INTEGER NOT NULL,
	B365HomeWinOdds REAL NOT NULL,
	B365AwayWinOdds REAL NOT NULL,
	B365DrawOdds REAL NOT NULL,
	MatchDate NUMERIC NOT NULL,
	Primary Key (MatchID),
	FOREIGN KEY (HomeTeamID) REFERENCES Teams(TeamID),
	FOREIGN KEY (AwayTeamID) REFERENCES Teams(TeamID),
	FOREIGN KEY (RefereeID) REFERENCES Referees(RefereeID)
);`

﻿`CREATE TABLE Referees (
	RefereeID	INTEGER NOT NULL,
	RefereeName TEXT NOT NULL,
	PRIMARY KEY(RefereeID)
);`


﻿`CREATE TABLE Teams (
	TeamID INTEGER NOT NULL,
	Name TEXT,
	Stadium TEXT,
	LatD	INTEGER,
	LatM	INTEGER,
	LatS	INTEGER,
	LonD	INTEGER,
	LonM	INTEGER,
	LonS	INTEGER,
	PRIMARY KEY(TeamID)
);`

﻿`CREATE TABLE TeamMatchStats (
	MatchID INTEGER NOT NULL,
	TeamID INTEGER NOT NULL,
	TeamShots INTEGER NOT NULL,
	TeamShotsOnTarget INTEGER NOT NULL,
	TeamYellows INTEGER NOT NULL,
	TeamReds INTEGER NOT NULL,
	FOREIGN KEY (MatchID) REFERENCES Matches(MatchID),
	FOREIGN KEY (TeamID) REFERENCES Teams(TeamID)
);`


﻿**Fourth** - The follow commands should be carried out in the "Execute SQL" tab in the DB Browser for SQLite. 
There are 5 queries that transfer data from the CSV tables into the database tables that have just been created:

`INSERT INTO Referees (RefereeName)
SELECT DISTINCT RefereeName FROM EPL_DATA;`

`INSERT INTO Teams (Name, Stadium, LatD, LatM, LatS, LonD, LonM, LonS)
SELECT DISTINCT Team, Stadium, LatD, LatM, LatS, LonD, LonM, LonS FROM PremiershipFootballGrounds;`

`INSERT INTO Matches (MatchID, HomeTeamID, AwayTeamID, FTHomeTeamGoals, FTAwayTeamGoals, FullTimeResult, RefereeID, B365HomeWinOdds, B365AwayWinOdds, B365DrawOdds, MatchDate)
SELECT  
EPL_DATA.MatchID, Team1.TeamID, Team2.TeamID, 
FTHomeTeamGoals, FTAwayTeamGoals, FullTimeResult, Referees.RefereeID, B365H, B365A, B365D, Date
FROM EPL_DATA
JOIN Teams as Team1
ON Team1.Name = EPL_DATA.HomeTeam
JOIN Teams as Team2
On Team2.Name= EPL_DATA.AwayTeam
NATURAL JOIN Referees;`

Inserting HomeTeam Stats for each match in TeamMatchStats

`﻿INSERT INTO TeamMatchStats (MatchID, TeamID, TeamShots, TeamShotsOnTarget, TeamYellows, TeamReds)
SELECT  
EPL_DATA.MatchID, Team1.TeamID, HomeTeamShots, HomeTeamShotsTarget, HomeTeamYellows, HomeTeamReds
FROM EPL_DATA
JOIN Teams as Team1
ON Team1.Name = EPL_DATA.HomeTeam;`

Inserting AwayTeam Stats for each match in TeamMatchStats

﻿`INSERT INTO TeamMatchStats (MatchID, TeamID, TeamShots, TeamShotsOnTarget, TeamYellows, TeamReds)
SELECT  
EPL_DATA.MatchID, Team2.TeamID, AwayTeamShots, AwayTeamShotsTarget, AwayTeamYellows, AwayTeamReds
FROM EPL_DATA
JOIN Teams as Team2
ON Team2.Name = EPL_DATA.AwayTeam;`


**Fifth** - Delete the two tables (“EPL_DATA” and “PremiershipFootballGrounds”) that were created directly from the corresponding CSV files. 


Extra: 
___________________________________________________
Some example questions, their queries, and their outputs:

1. Which team had the most shots on target throughout the course of the season? (join)

﻿`SELECT Name 
FROM Teams NATURAL JOIN TeamMatchStats
GROUP BY TeamID
ORDER BY SUM(TeamShotsOnTarget) 
DESC 
LIMIT 1;`

Liverpool

2. Which referee oversaw the most goals scored throughout the season and how many goals did they oversee? (join) 

`﻿SELECT RefereeName, SUM(FTHomeTeamGoals + FTAwayTeamGoals) AS Goals_Overseen FROM
MATCHES NATURAL JOIN REFEREES
GROUP BY RefereeID 
ORDER BY SUM(FTHomeTeamGoals + FTAwayTeamGoals) 
DESC 
LIMIT 1;`

﻿P Dowd	86

3. Which were the 3 matchups in the matches that had the lowest odds of ending in a draw? (this means highest figure since we are in the gambling realm)? (join) 

﻿`SELECT Team1.Name, Team2.Name FROM Matches
JOIN Teams as Team1
ON Team1.TeamID = Matches.HomeTeamID
JOIN Teams as Team2
On Team2.TeamID= Matches.AwayTeamID
ORDER BY B365DrawOdds
DESC
LIMIT 3;`

﻿Manchester City	Cardiff City
Manchester City	Crystal Palace
Manchester City	Aston Villa


4. Which day of the season saw the match with the highest combined number of yellow and red cards and how many cards were given out in that match? (Join)

﻿`SELECT Matches.MatchDate, Violence 
FROM (SELECT *, SUM (TeamYellows + TeamReds) AS Violence 
FROM TeamMatchStats
GROUP BY MatchID
ORDER BY SUM (TeamYellows + TeamReds)
DESC
LIMIT 1)
NATURAL JOIN Matches;`

﻿17/08/13	10