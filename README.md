# data_sourcing_challenge
Assignment for AI/ML Bootcamp Module 6 Challenge

Our assignment was to prepare data for a recommendation system to help people find movie reviews and related movies.

---------------------------------------------------------------------

## Description

I was tasked to prepare some data for a recommendation system to help people find movie reviews and related movies. I extracted data from two different sources: The New York Times and The Movie Database and merged the data together.

This challenge has three parts that must be completed in order:
- Part 1: Access the New York Times API
- Part 2: Access The Movie Database API
- Part 3: Merge and Clean the Data for Export

## Getting Started

### Dependencies

- Python 3.10
- Python Requests Library
- Python JSON Library
- Python python-dotenv Library
- Jupyter Notebooks

### API Keys

- API keys are required to execute the New York Times and The Movie Database APIs
- You will need to create accounts, register for APIs keys, and add these keys to your .env file to successfully execute this notebook
- To get a New York Times API Key, please reference the [New York Times Get Started page](https://developer.nytimes.com/get-started "https://developer.nytimes.com/get-started") for directions
-  To get a The Movie Database (TMDB) API Key, please reference the [TMDB Getting Started page](https://developer.themoviedb.org/docs/getting-started "https://developer.themoviedb.org/docs/getting-started") for directions

### Installing

- Clone this repo to your environment
- Create a .env file with your New York Times and The Movie Database API keys and save it in the folder that you cloned the repo to.


### Executing program

- Open '**retrieve_movie_data.ipynb**' from your cloned repo folder in Jupyter Notebooks
- Step through the completed notebook to see my data preparation by clicking the "Run" button.
- The results are displayed after each step.

**Part 1: Access the New York Times API**
1. I consulted the New York Times Article Search API documentation to build my query URL using the variables included in the starter code.

- Fields used to build my query URL:
    > **Base URL** = "https://api.nytimes.com/svc/search/v2/articlesearch.json?"
    >
    > **filter_query** = 'section_name:"Movies" AND type_of_material:"Review" AND headline:"love"'
    > This filters for movie reviews with "love" in the headline.  This requires section_name to be "Movies" and the type_of_material to be be "Review".
    >
    > **sort** = "newest"
    > Sorts results by newest first
    >
    > **field_list** = "headline,web_url,snippet,source,keywords,pub_date,byline,word_count"
    > This identifies the data fields to return from the API call
    >
    > **begin_date** = "20130101"
    > **end_date** = "20230531"
    > This searches for reviews published between the begin and end date

2. I created an empty list called reviews_list to store the results from your API requests.

3.  The Article Search API limits results to 10 per page and I retrieved 200. To do this, I created a for loop to loop through 20 pages (starting from page 0). Inside the loop, I performed the following actions:

    - Extended the query_url created in Step 1 to include the page parameter "&page=page_number".
    - Made a GET request to retrieve the page of results, and stored the JSON data in a variable called reviews.
    - Added a 12-second interval between queries to stay within API query limits.

    - Important: The New York Times limits requests to 500 per day and 5 per minute.

    - Wrote a try-except clause that performs the following actions:

        - try: loop through the reviews["response"]["docs"] and append each review to the list, then print out the query page number (i.e. the number of times the loop has executed).

        - except: Print the page number that had no results then break from the loop.

4. Previewed the first five results in JSON format using json.dumps with the argument indent=4 to format the data.
5. Converted reviews_list to a Pandas DataFrame using json_normalize()
6. Extracted the movie title from the "headline.main" column and saved it to a new column "title" using the Pandas apply() method and the following lambda function:

> lambda st: st[st.find("\u2018")+1:st.find("\u2019 Review")] 

- This code takes the string in the cell and extracts the characters between the unicode quotation marks, as long as a space and the word "Review" follows the closing quotation mark.

7. Used the supplied extract_keywords function to convert the "keywords" column from a list of dictionaries to strings using the apply() method.
8. Created a list called titles from the "title" column using to_list(). These titles will be used in the query for The Movie Database.

**Part 2: Access The Movie Database API**
- I consulted the TMDB Search & Query API documentation to build my query URLs using the variables included in the starter code. I made two search requests to extract the information I needed, Search and Movie Details.

- The search query is used to find the movie ID from the search by title. Most of this query was included in the starter code, as follows, but I needed to include the movie title in the query.  I got the movies titles from the titles list that I created from the New York Times query above.

- Prepare The Movie Database query:
    > **url** = "https://api.themoviedb.org/3/search/movie?query="
    >  **tmdb_key_string** = "&api_key=" + tmdb_api_key
- The movie query is made after I get the movie ID.

- I used the titles list created in Part 1 to perform the queries of The Movie Database.

1. I created an empty list called tmdb_movies_list to store the results from the API requests. This will contain a list of dictionaries.
2. I created a variable called request_counter and initialized it with the value of 1. This counter does the following:
    - Increment by one every time you iterate through the titles list.
    - Use time.sleep(1) when it reaches a multiple of 50.
    - Print a message to indicate that the application is sleeping.

3. Looped through the titles list created from the movie reviews DataFrame, and performed the following actions:
    - Performed the actions outlined in Step 2.
    - Performed a GET request that sends the title to The Movie Database search and retrieves the JSON results.
    - Used a try clause that performs the following actions:
        - Collect the movie ID from the first result.
        - Make a GET request using the movie query (starting with https://api.themoviedb.org/3/movie/) and movie ID to retrieve the full movie details in JSON format.
        - Extract the genre names from the results into a list called genres.
        - Extract the spoken_languages' English name from the results into a list called spoken_languages.
        - Extract the production_countries' name from the results into a list called production_countries.
        - Create a dictionary with the following results: title, original_title, budget, original_language, homepage, overview, popularity, runtime, revenue, release_date, vote_average, vote_count, as well as the genres, spoken_languages, and production_countries lists I created.
        - Append this dictionary to tmdb_movies_list.
        - Print out the name of the movie and a message to indicate that the title was found.
    - Used the except clause to print out a statement if a movie is not found.

4. Previewed the first five results in JSON format using json.dumps with the argument indent=4 to format the data.
5. Converted the results to a DataFrame called tmdb_df with pd.DataFrame().

**Part 3: Merge and Clean the Data for Export**
- Now that I have collected the data from both APIs, I merged the two DataFrames and cleaned the data, then exported it for future use.

1. Merged the New York Times reviews and TMDB DataFrames on the title column.
2. The genres, spoken_languages, and production_countries columns were saved as lists, but the columns need to be converted be strings without the list characters ([, ], and '). To fix these columns, I performed the following actions:
    - Create a list of the columns that need fixing called columns_to_fix.
    - Create a list of characters to remove called characters_to_remove.
    - Loop through columns_to_fix and do the following:
        - Use astype() to convert the column to a string.
        - Loop through the characters_to_remove and use the Pandas str.replace() method to remove the character from the string.
    - Print the head of the updated DataFrame to confirm the list characters were removed.
3. Deleted any duplicate rows and reset the index.
4. Exported data to a CSV file, "merged_nyt_tmdb_df.csv", without the DataFrame's index.


## Help

- Please execute all steps in the notebook.  The results of above steps are used in subsequent steps. 


## Authors

- Author:  Tom Clemons

## Version History

- 0.1
    - Initial Release

## Acknowledgments

- New York Times Get Started information:  https://developer.nytimes.com/get-started
- New York Times Article Search API documentation:  https://developer.nytimes.com/docs/articlesearch-product/1/overview
- The Movie Database Getting Started information:  https://developer.themoviedb.org/docs/getting-started
- The Movie Database API documentation:  https://developer.themoviedb.org/docs/search-and-query-for-details