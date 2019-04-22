<h1>NBA Twitter Analysis</h1>
<p>
The project given to us was to demonstrate an Extract-Transform-Load process using Python against a database.  The initial considerations were two-fold.  Firstly, we had to find a some appropriate datasets.  We decided on three related datasets from different sources:
<p>
<ol>
<li>The “Social Power NBA” dataset from Kaggle</li>
<li>A game data API endpoint from stats.nba.com</li>
<li>The Twitter API</li>
</ol>
<p>
Secondly, and relatedly, we had to decide what types of questions we wanted analysts be able to inquire of the database we were compiling.  Ultimately, the main questions we wanted to be able to ask of our gestalt dataset were along the lines of what types relationships existed between a player’s activity on the court and their activity on twitter.  We were interested in being able to see how frequently a particular player’s twitter account was mentioned and what sorts of correlations that had with their on-court performance and personal twitter activity.
<p>
  Rather than extract all the data, transform all the data, and load it all at once, we structured our code to run a single extraction/transformation to produce a bridge table.  There was too much data available to pragmatically pull the other data sets and join them.  Instead, the bridge table was used as a list of values to call against the other two datasets on a daily basis, and append any new results to the database.  Initial data transforms were then performed on the API call results before being loaded into a mysql environment, as we would be using relational databases throughout the process.  The establishment of the database and API queries required a number of prerequisite actions
<br>
    <ul>
        <li>Running "pip install nba_api" within a git bash terminal</li>
		<li>Applying for (and receiving) a developer account with Twitter for API access.</li>
		<li>Creating a local file named "resources.py" used to house Twitter API access information <b>twitteruser</b> and <b>twitterpass</b></li>
		<li>Adding the path to the local MySQL environment to "resources.py" under the variable name <b>mysqlstr</b> to facilitate exchanges between Python and MySQL</li>
	</ul>
  
<p>
  The Social Power NBA dataset from Kaggle was our starting point.  This dataset came as a *.csv file containing a sample of 100 NBA players, and had some personal stats from the 2016-2017 season, along with their NBA player ID and twitter handle (if they had one).  This dataset was going to be the linchpin for our ETL process, as it would allow us to bridge the NBA data with the Twitter data.  The *.csv required some cleaning and preparation to ease its use in the python environment, as not all columns would be necessary, we eliminated players who didn’t have a twitter handle, and we wanted to promote column name continuity across all datasets to facilitate the bridge. 
<p>
 Player Stats Bridge Table Field Definitions
<br>
    <ul>
        <li><b>PLAYER_ID</b> - Primary key, unique numeric identifier for an individual player in the NBA</li>
        <li><b>PLAYER_NAME</b> - Proper capitalized first name and last name</li>
        <li><b>TEAM_ABBREVIATION</b> - Standardized NBA three letter abberviation for player's team</li>
        <li><b>TWITTER_HANDLE</b> - Player's username on Twitter</li>
        <li><b>ACTIVE_TWITTER_LAST_YEAR</b> - Boolean value, Whether the player published any content in 2017</li>
    </ul>    
<p>
  We then proceeded to use this database as a list for performing API calls on the other two datasets. This allowed us to minimize calling unnecessary data through APIs that otherwise would have been filtered out and discarded while performing joins on databases in memory.
<p>
  The <b>PLAYER_ID</b> column was iterated against the NBA stats API to pull game data from the previous day. The prerequisite installation of nba_api allows the importation of pre-existing python modules to facilitate access to the NBA API. Various statistics regarding the players performance within games were pulled through the NBA stats API into a local dataframe, reorganized, and then sent to the local mySQL environment.
<p>
 Player Game Stats Table Field Definitions
<br>
    <ul>
        <li><b>PLAYER_ID</b> - Part 1 of composite primary key, unique numeric identifier for an individual player in the NBA</li>
        <li><b>GAME_DATE</b> - Part 2 of composite primary key, date of game played</li>
        <li><b>WL</b> - Whether the game was ultimately won or lost</li>
        <li><b>PTS</b> - Number of points scored by a particular player during a game</li>
        <li><b>PLUS_MINUS</b> - Total cumulative point difference while a particular player is actively playing during a game</li>
    </ul>
<p>
  Likewise, a series of calls using the <b>TWITTER_HANDLE</b> column were run against the Twitter API to pull tweet data regarding the given list of players.  Running the Twitter API required applying for and receiving a developer account from Twitter.  The Twitter API calls each returned a significant quantity of data that required extensive examination and filtering to narrow the content down to just the fields that would be relevant to the analyses we would expect to be run on the final database.  We wanted to rerturn all Tweets associated with a particular player, but the fields had to be organized in such a way that a many-to-many relationship could be supported.  Tweets could come from players (tweet) or simply contain their @username (mention).  Additionally, a single tweet could potentially contain the usernames of multiple NBA players.  To maintain the data integrity of tweets regarding individual users, a composite primary key was constructed using the <b>TWEETID</b> and <b>PLAYER_ID</b> fields.  For purposes of this project, we were not concerned with the contents of the tweets themselves.
<p>
Player Mentions Table Field Definitions
<br>
    <ul>
        <li><b>CREATE_DATE</b> - Date Tweet was published</li>
        <li><b>CREATE_TIME</b> - Time Tweet was published</li>
        <li><b>TWITTER_USER</b> - Twitter username that published a tweet</li>
        <li><b>RETWEETS</b> - How many times a Tweet was republished</li>
        <li><b>FAVORITES</b> - How many times a Tweet was "liked" by users</li>
        <li><b>PLAYER_ID</b> - Part 1 of composite primary key, imported from Player Stats Bridge Table and used in API query</li>
        <li><b>TWEETID</b> - Part 2 of composite primary key, unique numeric identifier assinged by Twitter to a published Tweet</li>
    </ul>
<p>
  Because of the cumulative nature of the API calls, this final database would allow examination of Twitter activity regarding a player (not just tweets from the player themselves) that could be contrasted against a number of criteria.  Examples of potential analyses are listed below:
<ul>
  <li>Number of tweets about a player during/after a particular game</li>
  <li>Trends over time (such as a season, or year) regarding player performance vs. public mindshare</li> 
</ul>
