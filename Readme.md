# Creating alternative datasets for finance with Python — scraping, text-mining and sentiment analysis

![ScreenShot](/screenshots/sentiment_prop.png)

The fastest-growing category of data is unstructured, e.g. text and images. In finance many still rely — almost exclusively — on traditional, numeric time-series of prices and fundamental data. How can we access these growing sources of untapped, alternative data? And how do we make sense of millions of documents of text that no human can process in any reasonable amount of time?

This article demonstrates how you can source and analyse such data. As an example, we want to scrape freely-accessible news articles about the oil company Royal Dutch Shell. A convenient source is the newspaper The Business Times as it has no paywall and its archive is available going back a few years.

Once we have sourced and stored the articles we need to turn this unstructured data into a (numeric) format that algorithms can process. That’s what a natural language process (NLP) data pipeline does and the Python ecosystem has many open-source tools that help us do exactly that. Once the data is pre-processed and cleaned we can perform all kinds of analysis, including sentiment scoring.

1. The tool box: Python data science and text-mining packages
2. The data: Scraping news articles from the Business Times website
3. Building a Natural Language Processing (NLP) pipeline with Spacy
4. Text pre-processing — tokenisation, lemmatisation, stop-word removal
5. Named-entity recognition
6. Sentiment algorithms: one proprietary, and two out-of-the-box (NLTK, TextBlob)
7. Visualising and comparing sentiment scores
8. Create manual labels to check sentiment score accuracy

Please have a look at a more detailed description in this [Medium article](https://medium.com/@marceldietsch/creating-alternative-datasets-for-finance-with-python-scraping-text-mining-and-sentiment-1c505b778ac9)


## Getting Started

These instructions will get you a copy of the project up and running on your local machine for development and testing purposes. 

### Prerequisites

```
Python==3.6
BeautifulSoup4  
Pandas
Matplotlib
Seaborn

TextBlob
NLTK
spaCy==2.1
spacy language model
python -m spacy download en_core_web_lg
```

### Installing

Install the dependencies from the prerequisite list and then run the notebook.

Example: The scraper can be run independently of the rest of the script:

```
def scraper(keyword):
    """
    takes search term for Business Times (Singapore) news articles, 
    runs scraper through 10 pages of the archive and
    returns dictionary with date (key) and article (value)
    """
    
    date_sentiments = {}
    article_text = {}
    counter = 1

    for i in range(1,9):
        
        page = urlopen('https://www.businesstimes.com.sg/search/'+ keyword +'?page='+str(i)).read()
        soup = BeautifulSoup(page, features="html.parser")
        posts = soup.findAll("div", {"class": "media-body"})
        
        for post in posts:
            
            time.sleep(2)
            url = post.a['href']
            date = post.time.text
            print("Article:", counter, "|", date, "URL:", url[8:65] + "...")
            counter += 1
            
            try:
                link_page = urlopen(url).read()
            except:
                url = url[:-2]
                link_page = urlopen(url).read()
                
            link_soup = BeautifulSoup(link_page)
            sentences = link_soup.findAll("p")
            passage = ""
            
            for sentence in sentences:
                passage += sentence.text
            
            article_text.setdefault(date, []).append(passage)

    articles = {}
    
    for k,v in article_text.items():
        articles[datetime.strptime(k, '%d %b %Y').date() + timedelta(days=1)] = v 
    
    return articles
    
    
articles = scraper("shell")
```


## Visualisation examples

Visualise named-entities in your text:

```
displacy.render(spacy_nlp(str(sentences[:13])), jupyter=True, style='ent')
```

![ScreenShot](/screenshots/ner.png)

Visualise and compare sentiment distributions:

```
sns.set(style="white", palette="muted", color_codes=True)

plt.figure(figsize=(12,7))
ax = sns.kdeplot(df.NLTK_spacy)
ax2 = sns.kdeplot(df.Proprietary_spacy)
ax2 = sns.kdeplot(df.TextBlob_spacy)
plt.title("Sentiment algorithms - probability density functions", fontsize=19)
sns.despine(left=True)
```

![ScreenShot](/screenshots/distributions.png)


## Authors

* **Marcel Dietsch** - (https://github.com/marceld)

## License

This project is licensed under the MIT License 