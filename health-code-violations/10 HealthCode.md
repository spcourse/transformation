
# Health code inspections

For this assignment we'll be working with restaurant health code inspection
data from the city of Seattle. It lists which restaurants were inspected when,
and how severe any health code violations were. This might range from simple
things like not having thermometers installed in the fridge and or a lack of
labeling on food containers, to more serious concerns like not separating raw
and cooked foods, or not keeping the establishment clear of rodents and
insects.

Download the data files for this assignment [here](Seattle_Health_Code.zip).
Note this file is quite large, so downloading it might take a while.

Get started by loading the data in `Health_Code_Violations_Seattle.csv` and
take a first look at the type of data it contains. If you want to print the
data frame, make sure to only print the [head](https://pandas.pydata.org/pandas-docs/stable/reference/api/pandas.DataFrame.head.html)
as the file is reasonably large. For this whole assignment, you should take
some care with what parts of the data you try to print and inspect, as is it
can take quite a while to print all of it.

We'll combine this data with restaurant reviews from *Yelp* to see if we can
find any indicators that a restaurant has serious health code violations. The
research question we'll try and answer with this combined data is:

* Are there any words whose presence in a *Yelp* review might indicate that
there is a serious health code violation present at a restaurant?

Given such a list of words, we could then automatically select certain
restaurants for inspection, which could help the *Washington State Department
of Health* reduce the number of serious health code violations.

## Yelp Data

The health code inspections for Seattle you've just loaded from the CSV will be
one part of the data. The other part will be the Yelp reviews for restaurants
from the same area. To answer our research question, we're going to try and
match these 2 different data sources, so find reviews and inspection results
that are about the *same* restaurant.

The reviews we will be using are a part of the [Yelp open dataset](https://www.yelp.com/dataset),
which is a subset of *Yelp* data that is made publicly available. All this Yelp
data is in *JSON* format, which you might not be familiar with yet. JSON is a
very common format on the web and is probably the second most common format you
will encounter for data, after *CSV*. Read a short introduction on JSON
[here](https://www.w3schools.com/whatis/whatis_json.asp) if you've never worked
with the format before.

Take a look at the [documentation](https://www.yelp.com/dataset/documentation/main)
for the dataset to get an idea of the types of data it contains exactly. The
goal we have here, based on our research question, is to match health code
inspections with reviews. However, `review.json` only contains `business_id`'s
for each review, which we cannot directly link to the inspections. The
`business_id`'s can be found in another file though, `business.json`, that also
contains the more descriptive name and address information for the business.
Linking each review to their respective business will therefore be a good first
step.


### Reading the data

Python has a built-in module for processing JSON files, unsurprisingly called
`json`. You can load JSON objects from a string format using [loads()](https://docs.python.org/3.8/library/json.html#json.loads).
The result will be some combination of dictionaries and lists, both familiar
Python datastructures, which take the place of *objects* and *arrays* in JSON
respectively.

Open the `business.json` file with a text editor and inspect what the structure
is exactly. Write some code to read this file and process the JSON objects
using `loads()` from the `json` module. If you need a refresher how to read
files in Python, take a look [here](https://docs.python.org/3/tutorial/inputoutput.html#reading-and-writing-files).

As we've discussed earlier, we'll need to match these businesses to reviews,
which means searching for each business by `business_id`. Given this dataset is
really quite large, programming this search efficiently will be crucial, so
we'll use a dictionary to speed up the search. As we'll be searching for the
`business_id`'s, we'll be using those as the dictionary **keys**. The
**values** should be the JSON objects containing all the other information
about the business.

Thinking ahead, we'll want to combine all the reviews for a business in one
place, so add one more attribute to the JSON object of each business, called
`'reviews'`, which should be an empty list (we'll fill it later). The final
structure should look something like:

    {'nxeMvqQDJ5ocA4toqRE3CQ': {'business_id': 'nxeMvqQDJ5ocA4toqRE3CQ',
                                'name': 'Ximaica',
                                ...,
                                'categories': ['Restaurants'],
                                'reviews': []},
     'oyxW3ZP60Z9b7F3LciaCjQ': {'business_id': 'oyxW3ZP60Z9b7F3LciaCjQ',
                                'name': 'XO Bistro',
                                ...,
                                'categories': ['French', 'Restaurants'],
                                'reviews': []},
     '0iyzWPc4C58sAJUAAE-BIg': {'business_id': '0iyzWPc4C58sAJUAAE-BIg',
                                'name': 'Xplosive Mobile Food Truck',
                                ...,
                                'categories': ['Food', 'Vietnamese', 'Food Trucks', 'Filipino', 'Restaurants'],
                                'reviews': []},
     ...}

Remember, *don't print the whole data frame!*

> #### Sidenote: Considerations for structuring data
>
> Before filling this dictionary with reviews, we'll take a brief
> aside to discuss different ways to represent this (or any other) data, and why
> you might choose one method over the other. The main consideration here is
> between representing your data in a flat table-like structure, like a pandas
> *DataFrame*, or a hierarchical structure consisting of dictionaries and lists,
> like the *JSON* structure. Knowing both methods will be useful when choosing
> the structures for you own data processing projects.
> <br/><br/>
> The `business.json` data seems like it is actually very close to data you might
> represent in a *DataFrame*. In fact, if the `'categories'` attribute wasn't a
> part of the data, it would be natural to interpret every attribute as a
> column name and every new line as a new row in the DataFrame. Pandas even comes
> with a built-in function to convert json data to a DataFrame: [read_json](https://pandas.pydata.org/pandas-docs/stable/reference/api/pandas.read_json.html).
> <br/><br/>
> One of the problems here is that we can't easily convert the part of the data
> which is hierarchical, e.g. objects or lists contained in other objects (like
> `'categories'`), as there is no clear conversion to a flat table format. However,
> we could just choose to drop the `'categories'` part of the data, as it isn't
> directly relevant to our research question, and then we could use some of the
> nice pandas features to process the data. Specifically, merging data from
> different sources can easily be done in pandas: [merge DataFrames](https://pandas.pydata.org/pandas-docs/stable/user_guide/merging.html#database-style-dataframe-or-named-series-joining-merging)
> <br/><br/>
> Merging allows you to combine DataFrames based on values in a specific column,
> making news rows consisting of each of the matched pairs, which would instantly
> solve the problem of matching the `business_id`'s of the reviews and of the
> business descriptions together. Having a hierarchical representation has its advantages
> too though, as all relevant data can be stored together; the reviews can just
> be put in a list contained *within* each business object (the empty `reviews` list you
> just added), whereas in a DataFrame we'd have to perform computationally
> expensive `groupby()` operations every time we'd want to do something with
> all the reviews for a business.
> <br/><br/>
> There are trade-offs to both structures and a valid argument could be made for
> using either structure here. However, after this merge, we'll want to also merge in the
> inspection results, which would become a very complex DataFrame with *a lot* of
> duplicate information. So in our case, here we will already make the choice to utilize a
> dictionary representation, based on the other transformations we will need
> to do with the data later on.

### Merging the JSON reviews

Now, lets actually merge the reviews into the business dictionary. Read the
`review.json` file into your program, reusing some of the code from loading
`business.json`. When you've loaded a review object, search for the
corresponding business in your business dictionary and add the review to the
list of reviews there. The merged results should end up looking something like:

    {'nxeMvqQDJ5ocA4toqRE3CQ': {'business_id': 'nxeMvqQDJ5ocA4toqRE3CQ',
                                'name': 'Ximaica',
                                ...,
                                'categories': ['Restaurants'],
                                'reviews': [{'review_id': 'XmAqfWFa_GyBj5nmU_ETXA',
                                             'business_id': 'nxeMvqQDJ5ocA4toqRE3CQ',
                                             ...,
                                             'stars': 3.0,
                                             'text': "Jamaican themed bar. DJ spins reggae ..."},
                                            {'review_id': 'xkSh71JlbLm9K9trwPJd7A',
                                             'business_id': 'nxeMvqQDJ5ocA4toqRE3CQ',
                                             ...,
                                             'stars': 2.0,
                                             'text': 'This place is closed. The bar there ..."},
                                            {'review_id': '7qlD758wI3g4vK8RETGebQ',
                                             'business_id': 'nxeMvqQDJ5ocA4toqRE3CQ',
                                             'stars': 5.0,
                                             'text': "Sucks that this place is closed ..."},
                                            ...]},
     'oyxW3ZP60Z9b7F3LciaCjQ': {'business_id': 'oyxW3ZP60Z9b7F3LciaCjQ',
                                'name': 'XO Bistro',
                                ...,
                                'categories': ['French', 'Restaurants'],
                                'reviews': [{'review_id': 'taAal2VzHOXJHK4w0APmYQ',
                                             'business_id': 'oyxW3ZP60Z9b7F3LciaCjQ',
                                             ...,
                                             'stars': 5.0,
                                             'text': "Lovely food and top-notch service ..."},
                                            {'review_id': 'TtoNrYmmGWqZ4RGDEv0MNA',
                                             'business_id': 'oyxW3ZP60Z9b7F3LciaCjQ',
                                             ...,
                                             'stars': 4.0,
                                             'text': "I visited this place a few times ..."},
                                            {'review_id': '7Ni9DMjJ3zZ9kq2jARmmyQ',
                                             'business_id': 'oyxW3ZP60Z9b7F3LciaCjQ',
                                             ...,
                                             'stars': 5.0,
                                             'text': "Chalk one up for this low-key, ..."},
                                            ...]},
    ...}


## Matching the inspections with reviews

Next we'll get started with the harder part of combining this data: merging the
inspection results into this same structure. Take a quick look at the file
containing all the inspections, which you have already loaded at the start of this
assignment. The most logical way to match these inspection results to the
reviews is through using the name of the business, but since these names are
written by different inspectors and reviewers, it might be hard to match them
*exactly*. We could also include information like address to try and get a more
accurate match, but the address information in the inspection file seem to be
missing often, so that probably won't improve results.

The business names in the inspection results look like they are formatted a
little differently too, so we'll need to do some conversions to ensure they
match the names used in the business descriptions. The names appear to be in
uppercase, so that'll definitely be something to fix. Also, it is probably a
good idea to remove any nonalphabetic characters, as those are very likely to
be used inconsistently in different data sources. Write a function called
`clean_name()`, which takes a restaurant name, converts it to all lowercase and
removes any nonalphabetic characters.

When merging in the inspection results with the review data, we'll be
repeatedly be searching for a restaurant with a specific name, and not its
`business_id`. So while doing these conversions, it would also be a good idea
to restructure the dictionary containing the data so we can easily search based
on the name of the restaurant. The *values* of this new dictionary should be the
same as in the old dictionary, but the *keys* should the cleaned restaurant
names, instead of the id's. Also, to clean up the data a little more, remove
all restaurants that don't have any reviews, as they won't help answer our
research question. Finally, add a new attribute `'inspections'`, containing an
empty list, in each object, just like you did for the reviews, where all the
inspection results can be merged into.

> #### Sidenote: Efficiency of transformations
>
> Making a whole new dictionary with different keys might seem inefficient and
> tedious, but given that we will have to try and find every business by name, it
> is actually much faster. This approach will be two $$O(N)$$ loops; one to rebuild
> the dictionary and one to search for every name, as one search itself will now
> be $$O(1)$$ in the new dictionary. Compare that to the version using nested
> loops to match the names, which would be $$O(N^2)$$, so *extremely* slow for
> data this size. This transformation is therefore part restructuring (changing
> the keys by which the dictionary is indexed) and part content modification
> (converting the restaurant names to the cleaned format), with the sole aim of
> reducing the complexity of the next transformation.

The restructured dictionary should end up looking something like:

    {'youngs restaurant': {'business_id': 'IbRT1K1TQ7mFXjlIZ9onUw',
                           'name': "Young's Restaurant",
                           ...,
                           'categories':  ['American (Traditional)', 'Chinese', 'Restaurants'],
                           'reviews':     [{'review_id': 'K90lbzJj_HUVxqkhp87bIA',
                                            'business_id': 'IbRT1K1TQ7mFXjlIZ9onUw',
                                            ...,
                                            'stars': 5.0,
                                            'text': "One of my favorite breakfast/brunch ..."},
                                           {'review_id': 'qTcdlvOHiI84_5e6mfe2QQ',
                                            'business_id': 'IbRT1K1TQ7mFXjlIZ9onUw',
                                            ...,
                                            'stars': 3.0,
                                            'text': "Honestly I wanted to try this place ..."},
                                           ...],
                           'inspections': []},
     'yu shan':           {'business_id': '0rqp9CbHsZZEw1UOUHInWg',
                           'name': 'Yu Shan',
                           ...,
                           'categories':  ['Chinese', 'Restaurants'],
                           'reviews':     [{'review_id': 'KOwWTvfTcl7jCbu8xoWxmw',
                                            'business_id': '0rqp9CbHsZZEw1UOUHInWg',
                                            ...,
                                            'stars': 5.0,
                                            'text': "This five star is for in house only ..."},
                                           {'review_id': '1cQGC96CIWC3HhULg_F6Dw',
                                            'business_id': '0rqp9CbHsZZEw1UOUHInWg',
                                            ...,
                                            'stars': 1.0,
                                            'text': "I ordered this food for delivery ..."},
                                           ...],
                           'inspections': []},
     ...}


### Merging in the inspections

Now apply the same `clean_name()` function to every name in the inspections
DataFrame. Use [map()](https://pandas.pydata.org/pandas-docs/stable/reference/api/pandas.Series.map.html)
to apply the function to entire `'Name'` column at once and store the result in
that same column. Using the same `clean_name()` function to clean up both sources
of data should ensure consistent results for this transformation, so names that
started out the same should still match after these transformations are applied.

Loop over the inspections DataFrame using [iterrows()](https://pandas.pydata.org/pandas-docs/stable/reference/api/pandas.DataFrame.iterrows.html)
and search for the (cleaned) name in your restaurant dictionary. Add the
inspection to the list of inspections for that restaurant. Transform the row
*Series* to a dictionary using [to_dict()](https://pandas.pydata.org/pandas-docs/stable/reference/api/pandas.Series.to_dict.html),
to make the data consistent in type.

The final result of combining all these data files in single structure should
look something like:

    {'youngs restaurant': {'business_id': 'IbRT1K1TQ7mFXjlIZ9onUw',
                           'name': "Young's Restaurant",
                           ...,
                           'categories':  ['American (Traditional)', 'Chinese', 'Restaurants'],
                           'reviews':     [{'review_id': 'K90lbzJj_HUVxqkhp87bIA',
                                            'business_id': 'IbRT1K1TQ7mFXjlIZ9onUw',
                                            ...,
                                            'stars': 5.0,
                                            'text': "One of my favorite breakfast/brunch ..."},
                                           {'review_id': 'qTcdlvOHiI84_5e6mfe2QQ',
                                            'business_id': 'IbRT1K1TQ7mFXjlIZ9onUw',
                                            ...,
                                            'stars': 3.0,
                                            'text': "Honestly I wanted to try this place ..."},
                                           ...],
                           'inspections': [{'Name': 'youngs restaurant',
                                            'Program Identifier': "YOUNG'S RESTAURANT",
                                            ...,
                                            'Violation_Record_ID': nan,
                                            'Grade': 1.0},
                                           {'Name': 'youngs restaurant',
                                            'Program Identifier': "YOUNG'S RESTAURANT",
                                            ...,
                                            'Violation_Record_ID': 'IVKWHBZBH',
                                            'Grade': 1.0},
                                           ...]},
     'yu shan':           {'business_id': '0rqp9CbHsZZEw1UOUHInWg',
                           'name': 'Yu Shan',
                           ...,
                           'categories':  ['Chinese', 'Restaurants'],
                           'reviews':     [{'review_id': 'KOwWTvfTcl7jCbu8xoWxmw',
                                            'business_id': '0rqp9CbHsZZEw1UOUHInWg',
                                            ...,
                                            'stars': 5.0,
                                            'text': "This five star is for in house only ..."},
                                           {'review_id': '1cQGC96CIWC3HhULg_F6Dw',
                                            'business_id': '0rqp9CbHsZZEw1UOUHInWg',
                                            ...,
                                            'stars': 1.0,
                                            'text': "I ordered this food for delivery ..."},
                                           ...],
                           'inspections': [{'Name': 'yu shan',
                                            'Program Identifier': 'YU SHAN',
                                            ...,
                                            'Violation_Record_ID': nan,
                                            'Grade': 1.0},
                                           {'Name': 'yu shan',
                                            'Program Identifier': 'YU SHAN',
                                            ...,
                                            'Violation_Record_ID': 'IV53U6A9T',
                                            'Grade': 1.0},
                                           ...]},
     ...}

## Processing the combined data

Now that we have all the data combined in a single structure, we can finally
start processing it. As the reviews are all just raw text data, we'll
definitely need to do some processing. For this we'll use the `nltk`
package, which stands for Natural Language ToolKit. Try to import the package
and see if it is already installed on your machine. If not, follow the
instructions [here](https://www.nltk.org/install.html).

We'll use a couple of very basic `nltk` features for processing the text;
tokenization and stop words. Tokenization splits the text into words, which is
essentially splitting on *spaces*, but also takes things like punctuation into
account. Also, some words are so common we want to filter them from
processing altogether, like the word *"the"*. These words are called
*stopwords* and `nltk` has standard lists for many languages. Include the
following code to load the list for English:

    import nltk

    nltk.download('punkt')
    nltk.download('stopwords')

    stop_words = set(nltk.corpus.stopwords.words("english"))


> #### Sidenote: Sets
>
> Since we only want to search if each word is contained in the list of
> stopwords, we'll use a **set** instead of a list. Using a set here converts
> each search from $$O(N)$$ to $$O(1)$$.

No need to write any further code for this step, here we've just set up the
tools we'll be needing for the next steps.

### Processing the violations

To answer our research question *"Are there any words whose presence in a Yelp
review might indicate that there is a serious health code violation present at
a restaurant?"*, we'll need to distinguish between restaurants that are
generally clean and ones that have serious health code violations. These
restaurants are most likely to have reviews with distinguishing words, as
something like unlabeled food in the fridge is unlikely to directly affect the
customer reviews. Take a look at the data containing the inspection results and
see if you can come up with any good criteria for filtering these results.

While defining your own criteria is a useful exercise, lets start out with
using a predefined set, so we can refer to the same results for the next few
steps. What I ended up selecting was just filtering on `'Inspection Score'`,
which seemed descriptive on how good or bad the inspection was, and also easy
enough to filter. If a restaurant has had several inspections, I just used the
maximum score, so the worst inspection the restaurant has ever gotten, as that
seemed like it would give the best indication of health code standards.

Looking at the type of violations listed in the inspection results, I ended up
settling on; any restaurant with a maximum score under $$20$$ is a generally
clean restaurant, while a maximum score over $$70$$ indicates serious health
code violations. This also results in nice distribution of approximately 120
restaurants labeled as clean and 120 as serious violations.

Loop over the dictionary containing all the data and divide the businesses and
their reviews into two lists; one list containing all the businesses considered
clean and one with businesses who had serious violations. You can ignore the
businesses that do not meet either criteria or do not have any inspection
results for now. You should end up with approximately 120 businesses in each
list.

### Processing the reviews

Next we'll actually process those reviews in both lists. Use the `nltk`
function [word_tokenize()](https://www.nltk.org/api/nltk.tokenize.html#module-nltk.tokenize)
to split the text of each review into words. Convert all words to lowercase and
remove any "words" that contain nonalphabetic characters (e.g. punctuation) or
words that are a part of the set of stopwords.

The processed result should still consist of 2 lists, 1 for clean restaurants
and 1 for those with violations. Within both these lists, all the processed
words for a one business should be merged to into a single list. So, *both lists
should consist of multiple lists of words, each containing all the processed
words in all reviews for one specific restaurant.*

### Counting word occurrences

Now let start with simply counting how often words occur in the clean
restaurant reviews and how often they occur in the reviews of those with
violations. Start out by combining all the lists with words for each business
to a single list, which is called flattening the list. There are many ways to
flatten a list in Python, I prefer using the [chain()](https://docs.python.org/3/library/itertools.html#itertools.chain)
function, but any flattening method will do. Do not discard/overwrite the
unflattened versions of the lists, as we *might* need them later on. The end
result of this transformation should be two massive flat lists of words, one
for each type of restaurant.

Next, use the built-in [Counter](https://docs.python.org/3/library/collections.html#counter-objects)
object to count how often elements occur in a list. Then select the 25
[most_common()](https://docs.python.org/3/library/collections.html#collections.Counter.most_common)
words for both lists and print them.

### Defining some better metrics

It seems as though there is pretty generic formula for Yelp reviews, as the set
of most common words appears to be nearly *identical* in both case. This could
have quite a few different causes; it could be because we're not filtering for
the serious violations correctly, or that a single serious violation does not
imply all bad reviews. For now we'll assume both of those are *not* the issue
and that some people (or bots) writing Yelp reviews are just not that creative.
So, instead we will define a better ordering metric than just the most common
word and try distinguish the two types of restaurant that way.

As there are so many commonalities between the word lists, a logical next step
would be to try to find words common in reviews of restaurants with violations,
but uncommon in reviews of clean restaurants. For now, we'll just focus on the
words that *do* actually occur in *both* lists, so we'll need to construct a
set containing these common words. As you may have already noticed, `Counter`
objects are actually just dictionaries with a couple extra features.
Additionally, the `.keys()` function returns a set-like [view of the dictionary](https://docs.python.org/release/3.3.0/library/stdtypes.html#dict-views),
so computing the [intersection()](https://docs.python.org/3/library/stdtypes.html#frozenset.intersection)
of words occurring in both *key-sets* should actually be pretty easy.

Using this set of words occurring in both `Counter` objects, we'll build a new
dictionary where we normalize the count of every word by dividing it by the
total count for that type of restaurant, making the values for both types
more comparable. Then we'll use the ratio between the normalized counts of both
types to assign a final score to each word.

$$score\_word = \dfrac{\frac{violations\_word\_count}{total\_violations\_count}}{\frac{clean\_word\_count}{total\_clean\_count}}$$

Dividing by the normalized count of how often a word occurs in reviews of
restaurants with *clean* inspection results, should give words that occur
in those restaurant *less* a higher score. This combined rating for each word,
based on the counts for both lists, can then be sorted to find the top 50 words
with the highest score. Use the built-in function [sorted()](https://docs.python.org/3/library/functions.html#sorted)
to construct this top 50 of words and print the results.

> #### Sidenote: Origins of this metric
>
> Dividing by the normalized count of how often a word occurs in reviews of
> restaurants with *clean* inspection results, might seem like a somewhat
> arbitrary way to filter out common words. In essence, it is, and the only
> real indication we have of it making any sense, is if it actually produces a
> better list of words. However, it does also have some theoretical foundation,
> which might be interesting to note. The metric was heavily inspired by
> [tf-idf](https://en.wikipedia.org/wiki/Tf%E2%80%93idf), which is a widely
> used metric in information retrieval for improving search results.

### Tweaking the metric further

The results from this metric seem to mostly produce names of dishes or even
specific restaurants that correspond to serious violations. What we would
actually like in our top 50 words are general trends among the *different*
restaurants. For this new metric we can reuse most of the code from the
previous metric, we'll only need to adjust how exactly words are counted.

So, instead of counting how many times a word occurs overall in either list, we
should *count in how many of the different business reviews it occurs*. The
easiest way to do that is to take the *unflattened* list of reviews, remove
duplicate words for each business, and then flatten the results. Applying a
`Counter` to this flattened list should return in how many different
restaurants' reviews this word is mentioned. The rest of the metric can be left
exactly as is, just use this new count dictionary to compute the score for each
word, sort and print the top 50 words.

### Final improvements

This metric definitely isn't perfect, but it produces some interesting words in
the top 50 list, most notably *inspector/inspection* and *cockroach/roach*.
There are plenty more improvements possible to get the list of words even more
representative:

* Improve the definition of *clean* and *severe violations*, so the reviews are
separated more accurately.

* Use the reviews from restaurants without any inspections as general model for
common food terms or specific dishes, so those can be filtered from the top 50
distinguishing words.

* Use the exact time window of the inspection to filter reviews that are more
likely to be related to results of the inspection.

* Tweak the word scoring in some other way and see if this improves the results.

Implement any one of these extension, or come up with your own improvements and
print the final top 50 of most indicative words.

> Disclaimer: not having "cockroach/roach" and "inspection/inspection" does not _immediately_ mean that something's wrong with your code. Try checking with your TA if you're not sure.


## BONUS: Yelp API

Now that you have a good list of words to monitor for in *Yelp* reviews, you
can build a program that does exactly that! Yelp also offers *API* end points
where you can just send requests for specific data and get some *JSON* data
back. From there building a monitoring system is quite straightforward,
especially considering you already wrote code to load the JSON objects and
process the review text.

For a general introduction on what an API is and how to use it, read the blog
post [here](https://www.dataquest.io/blog/python-api-tutorial/). Then take a
look the documentation for the Yelp [api](https://www.yelp.com/developers/documentation/v3),
specifically the end points [search](https://www.yelp.com/developers/documentation/v3/business_search)
and [reviews](https://www.yelp.com/developers/documentation/v3/business_reviews).

Write a program to actively monitor Yelp for new restaurants in Seattle which
get reviews that contain words from your top most indicative words for health
code violations and add these to a list for future inspections by the
*Washington State Department of Health*.
