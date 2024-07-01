<h1>PokemonScrapingProject</h1>
<p>These scrapers pull information regarding Pokemon from <a href="https://www.serebii.net/">Serebii</a> and <a href="https://pokemondb.net/">PokemonDB</a>, and split it into separate tables for Pokedex, moves, learnsets, and evolution data, repeating for each generation of mainline games. The purpose was to get the data into a format where it can be used for whatever data analyses I or anyone who happens upon it want to do. To that end, I've included the output data files here for anyone who would rather work just with them.</p>
<h2>Resources</h2>
<ul>
  <li><a href="https://www.serebii.net/pokedex/">Serebii Pokedex</a></li>
  <li><a href="https://www.serebii.net/attackdex-rby/">Serebii Attackdex</a></li>
  <li><a href="https://pokemondb.net/evolution/level">PokemonDB Evolution Guide</a></li>
</ul>
<p>Note: Both Serebii link leads specifically to Gen 1 hubs, while the PokemonDB link leads to the page for Pokemon of all generations who evolve via level up. If you dig into the files, you can see that I pulled from the Serebii pages for Pokedex and Attackdex for each generation individually, and pulled from the pages for each evolution method before filtering for just the Pokemon available in a given generation.</p>
<h2>Required Libraries</h2>
<ul>
  <li><a href="https://scrapy.org/">Scrapy</a></li>
  <li><a href="https://pandas.pydata.org/">Pandas</a></li>
  <li><a href="https://openpyxl.readthedocs.io/en/stable/">OpenPyXL</a></li>
  <li><a href="https://www.sqlalchemy.org/">SQLAlchemy</a></li>
</ul>
<h2>Setup</h2>
<p>Begin by installing Python v3.11.8 and the required libraries (above) using your preferred method. I used Anaconda for Python and am running on Windows 11, so subsequent instructions will be tailored to that. I also recommend installing an IDE if you wish to view or edit the scripts, though you can just use Notepad for that purpose.</p>
<p>If you wish to save this data to a PostgreSQL server, which the code already allows for, you will also need to install and set up PostgreSQL. The <code>create_engine</code> command in <code>sqlalchemy</code> takes on this form:</p>
<pre><code>create_engine('[database type]://[username]:[password]@[host]:[port]/[database name]')</code></pre>
<p>Match the <code>username</code>, <code>password</code>, <code>host</code>, and <code>port</code> used in the included files (postgres, psql_password, localhost, and 5432, respectively) while setting up your server, or change them to match your own in each spider file. You will also need to create the databases (pkmn_g1, pkmn_g2, pkmn_g3, pkmn_g4, pkmn_g5, pkmn_g6, pkmn_g7, pkmn_g8, and pkmn_g9) in pgAdmin before running the spiders, otherwise it will throw an error. </p>
<p>Next, create a virtual environment with Python and all required packages using your preferred method (I used the Anaconda prompt). Enter the following commands, plugging in your desired name for <code>[projectname]</code>:</p>
<pre><code>conda create -n [projectname] python=3.11.8 anaconda
conda activate [projectname]</code></pre>
<p>Installing Anaconda should handle all the required libraries, but in case future versions decide to get rid of one of them, here's how you can check:</p>
<pre><code>conda list [library]</code></pre>
<p>Plug in <code>sqlalchemy</code>, <code>scrapy</code>, <code>pandas</code>, and <code>openpyxl</code> for <code>[library]</code>. If they are installed, you should see something like this:</p>
<pre><code># Name                    Version                   Build  Channel
sqlalchemy                2.0.25          py311h2bbff1b_0</code></pre>
<p>If any of them are missing, you can install them manually with this command:</p>
<pre><code>conda install [library]</code></pre>
<p>Once certain you've installed all required libraries, run the following command to create the scrapy project, again using your desired name for <code>[projectname]</code>:</p>
<pre><code>scrapy startproject [projectname]</code></pre>
<p>To check that this worked as intended, run the following commands:</p>
<pre><code>cd [projectname]
tree /F</code></pre>
<p>This should result in something like this:</p>
<pre><code>Folder PATH listing
Volume serial number is [system-dependent hexadecimal value]
C:.
│   scrapy.cfg
│
└───[projectname]
    │   items.py
    │   middlewares.py
    │   pipelines.py
    │   settings.py
    │   __init__.py
    │
    └───spiders
            __init__.py</code></pre>
<p>From here, you could either start your own project or continue using this one. This isn't intended to be a full scrapy tutorial, so I'll leave it to you to figure out where to go from here if you want to start your own project. I have linked some resources I found helpful at the bottom of this ReadMe.</p>
<h2>Running the Project</h2>
<p>To proceed with the included scraper, place the included files into the second-level [projectname] folder, which contains <code>items.py</code>, the spiders folder, etc. Then, back in Anaconda Prompt, proceed to that same folder. The prompt should read something like <code>([projectname]) C:\Users\[username]\[projectname]\[projectname]></code>. From here, you can run the command:</p>
<pre><code>scrapy crawl [scrapernamevariable]</code></pre>
<p><code>[scrapernamevariable]</code> is the <code>name</code> variable near the start of each Spider class. For example, in the Gen 9 Evolutions scraper (<code>pkmn_g9_evo.py</code>):</p>
<pre><code>class PkmnSpider(scrapy.Spider):
    name = 'pkmn_g9_evo'</code></pre>
<p>For simplicity, I gave each spider script the same name as the spider within it, i.e <code>pkmn_g4_pkmn.py</code> contains the spider pkmn_g4_pkmn, which you run using the command</p>
<pre><code>scrapy crawl pkmn_g4_pkmn</code></pre>
<p>As noted above, if you are going to use PostgreSQL, you need to create the databases in pgAdmind first or the script will throw an error. It will also throw an error if you have already run the script and generated the intended table within the database. If you are saving in multiple methods, the other others should turn out fine should either of these happen. Additionally, <code>pkmn_g8_moves</code> should throw 25 errors (26 if you add on one of the aforementioned Postgres errors). This is a known issue, and one I address below in my notes on my approach.</p>
<h2>Possible Future Updates</h2>
<p>I have a lot of plans for this data. First, I didn't finish getting the learnsets for each generation. I'd like to get those done at some point. Starting in Gen 5, I simply stopped bothering with splitting each Ability into a separate column. I'd like to get that done, and to have them separated by Form. That seemed like a whole can of worms, so I put it off. In Gen 7, I didn't really bother with Z moves because the way they are written on Serebii is not very conducive to easy scraping. In Gen 8, I let the moves exclusive to Legends: Arceus throw errors (the 25 mentioned above) because I was simply didn't think it was worthwhile to fix them given that L:A doesn't have multiplayer. I'd also like to include some stuff for Pokemon contests, where certain moves have greater appeal if they're used after other moves. Last, I'd like to add something tell whether a Pokemon is a starter, pseudo-legendary, legendary, mythical, ultra beast, paradox, or just a standard mon. These are all features that weren't necessary for the analysis I had planned, so I elected not to bother with them for now. At some point, I'd like to get them all working.</p>
<p>Beyond those things, there are a number of shortcomings in my code. I could add more comments explaining things. I've violated DRY in quite a few ways, so I'd like to rewrite all of these so that it's just four files using argparse to pull functions and such for each gen. Then I'd like to package those further into a single script that runs once and outputs everything the way I have it now.</p>
<p>I plan to build further tools building off this data. I want to make a spreadsheet in which you write up your team composition (mons, moves, EVs, etc.) and it tells you how good your coverage is. I want to make a script optimizing Pokemon contest builds (this was actually how I got the idea to make a web scraper in the first place, though the idea ballooned from there).</p>
<h2>A Note on My Approach to Pokemon Forms and Other Matters</h2>
<p>As I went through the evolution data, I realized a few things. First, I realized that, in many cases, a Pokemon's various forms are really only different in aesthetic, and that I would end up having several redundant entries if I wanted to cover, say, Alcremie's 63 forms. My intention in doing this project was to perform various analyses on the data I've acquired, and I realized that having these redundant entries could unduly influence the results. Then I realized that if I wanted to minimize this effect appropriately, I would have to give some thought to where exactly the line is. After much thought, I concluded that there are two criteria for whether a form should be treated as distinct:</p>
<ul>
	<li>Does the form change anything other than appearance? Types, stats, abilities, etc.</li>
	<li>Is the form change more or less permanent?</li>
</ul>
<p>In the case of the Burmy line, Burmy's various forms change only its appearance, and can change within the space of a battle. The female evolution Wormadam, however, sees a different secondary type as well as different stats between its various forms, which <i>are</i> permanent. For a Pokemon like Arceus, which sees a type change with its different forms, I concluded that this is insufficiently permanent because you change its form by having it hold a different item. It is a simple enough matter to change out the plate between battles with, say, the Elite Four. For something like Deoxys, however, which sees significant changes in stats between forms, the form change is locked to which version you are playing in Gen 3, where it was introduced. In subsequent generations, this was changed to where you could alter Deoxys' form by interacting with shards of a meteor. These are kept in a fixed location, and thus you wouldn't be able to change Deoxys' form between battles with the Elite Four. In the case of Alcremie, all stats, abilities, types, etc. are identical between forms, but the evolution method varies slightly. In the interest of avoiding redundant entries in the Pokedex, I figured it was better to leave this explanation in the form of the summary given by PokemonDB: "Spin around holding a Sweet item." I may edit that entry in the future to provide more detail as to what factors determine the resulting form.</p>
<p>Mega Evolutions do not meet the permanence criteria, but they are sufficiently distinct mechanically as to warrant separate entries, and are easily censored from analysis data given their common form names. Dynamax and Gigantamax forms simply double current and max HP values and swap out moves for their Max versions, so it made more sense just to list each Pokemon's Max moves in their learnsets.</p>
<p>Another thing I noted while working with evolution data is that, with a handful of exceptions, Pokemon generally evolve using one of the following three methods:</p>
<ul>
	<li>Level Up</li>
	<li>Use Item</li>
	<li>Trade</li>
</ul>
<p>Though there are just these three main methods that trigger the evolution, there many different criteria that must be met for the evolution to proceed when the trigger is activated. Friendship evolutions, for example happens upon level up once a Pokemon's friendship has reached the specified threshold. A standard level up evolution happens when a Pokemon levels up once the Pokemon's level has reached the specified threshold. While the two differ in criteria, they are both triggered by leveling up. I haven't dug into the game's code, so I can't say if this quite how it works, but I'd expect that when Pokemon level up, certain checks are triggered which determine if the evolution thresholds for level, friendship, etc. have been met, and that the evolution sequence is triggered if they are. Since many Pokemon have compound evolution conditions like "level up while holding a specific item at a specific time of day" or "use a specific item during a certain weather condition," it made sense to me to write things this way.</p>
<h2>Recommendations for Learning Scrapy</h2>
<p>When learning things for this project, I made most use of the following:</p>
<ul>
	<li>Web Scraping in Python on <a href="https://app.datacamp.com/">DataCamp</a>. This covers a lot of the fundamentals, and gives a good overview of how to use XPath and CSS selectors.</li>
	<li>John Watson Rooney's <a href="https://www.youtube.com/c/JohnWatsonRooney">YouTube channel</a>. He goes through step by step showing how to do projects.</li>
</ul>
<p>If you encounter difficulty with setting up your selectors, I've found that ChatGPT is really helpful with breaking down what you've written and suggesting troubleshooting methods. Never take what it tells you as gospel, though. Just as people can make mistakes, ChatGPT can make them as well. Trust, but verify.</p>
