+++
title = "Search Engine with Python"
+++
<h1>**Search Engine with Python**</h1>
<body>A good search engine need not be sophisticated, it is better to start simple to understand the workings of a good search engine.
Of-course the first thing we need is a good dataset for raw textual data for oyr mining purpose. I have used the 
kaggle [data-set](https://www.kaggle.com/jessicali9530/kuc-hackathon-winter-2018/home) which consists of patient reviews on medications.
This is in  _**csv**_  format so we use **pandas** dataframe to load it into python.

</body>
<h2>**Data Preprocessing:**</h2>
To make things easy we are going to match search with only the reviews available.The first challenge we face is data preprocessing.
In our raw data set (test and train data combine) there are about **215,000** records. Upon inspection there are unsueable rows with 
incomplete "condition" and "Drug name" columns. After dropping those we are left with **213,892** rows.

The reviews contain some html codes that are left as is and need to be converted to utf-8 encoded string with **htlm** library's 
unescape function. Once this is done now its time for text mining, we convert each review string into a list of words with 
the _**split**_ function for a string. We ensure that all words are in **lower case** and must be of minimum length 3. 

The **nltk** library is one of the best and easiest for natural language processing with python which includes **_stopwords.stop("english")_** which can be downloaded and the function is available in the **corpus** function on the module. We use this to remove stop words from our  reviews. The library also consists of **_WordNetLemmatizer()_** using which we create a lemmatizer object and use the **__lemmatize()__** function on each word to find it's root word. This ends the preprocessing step.

**<h2>Creating Vocabulary:</h2>**
<body>We use the inverted index technique which is popular in search engines nowadays. It is very fast as we only calculate similarity for top few reviews.
This is a crucial step, here we traverse through all the preprocessed reviews in our dataset to find unique words in the entire pool. For this we
use a python __dictionary__ as the use hash indexing which is fast. The **keys** of this dictionary are the words. The **value** consists of a _list_, whose first element is the **document frequency** of that word _(number of reviews the word occurs in)_. The second element of this _list_ is another _dictionay_ where **keys** are **ID's** of reviews in which a word occured, **values** are a _list_ with the index position(s)
of the word within the review.
Using this we can now calculate the weights of each word in every review to create posting list for all words. For this we use the **tf-idf** [term frequency-Inverted document frequency](https://medium.freecodecamp.org/how-to-process-textual-data-using-tf-idf-in-python-cd2bbc0a94a3)
The third element in the _list_ is the posting list for every word.
  **Prototype:**
  <pre>
  { word1 : [ docfreq, { docid1:[pos1, pos2, .....], docid2:[pos1, pos2, ....], ....... }, 
            **{ doc1:w1, doc2:w2, .... }** ]
   .
   . 
   .
  }
  </pre>
  Notice we use a position list to store every index of every occurence of a word in a document.
  We get **32742** unique words as features for our dataset.
  <pre>
  
  </pre>
</body> 

<h3> Calculating the tf-idf weights for each word </h3>
<body>
  1. After construction of the first stage of the vocabulary we now calculate the weights of each word.
  2. Term frequency of a word is given with respect to each review it appears in:
  <pre>
  word1-tf-review1 = number of times word1 appears in review1 / number of words in review1
  </pre>
  3. Inverse document frequency for a word is defined by:
  <pre>
  word1-idf = total number of reviews in the dataset / number of reviews word1 occurs in
  </pre>
  4. The weight of a word in a reviewis given by:
  <pre>
  word1-weight-review1 = (1+ log(word1-tf-review1)) / (word1-idf)
  </pre>
  
</body>

**<h2>Query processing and calculating similarity</h2>**

<script type="text/javascript" src="https://cdnjs.cloudflare.com/ajax/libs/mathjax/2.7.1/MathJax.js?config=TeX-AMS-MML_HTMLorMML">
</script>
Finally we use the input box we created in our application's search page to retrive user search query and make it undergo the same 
pre-processing as our reviews, converting to lower case, stop word removal and lemmatization.
We calculate the weights of words in the query using only term frequency. Now we retrieve the **top 10** reviews from the posting lists of each word in the search query. 
If a review appears in the top-10 elements of every query word, calculate cosine similarity score. 

$$ {sim(q,r)} = \\vec{q} \\cdot \\vec{d} = \\sum_{t\ \\text{in both q and r}} w\_{t,q} \\times w\_{t,r}.$$

Where **'q'** is the search query and **'r'** is a review vector. If  a review doesn't appear in the top-k elements of some query words, use the weight in the 10th element as the upper-bound on weight in vector. Hence, we can calculate the upper-bound score for using the query word's actual and upper-bound weights with respect to vector, as follows. 

$$ \\overline{sim(q,r)} = \\sum_{t\\in T_1} w\_{t,q} \\times w\_{t,r}.$$

$$  + \sum_{t\\in T_2} w\_{t,q} \\times \\overline{w\_{t,r}}.$$


In the above equation, first part has query words whose top-k elements contain review. Second part includes query words whose top-k elements do not contain the review. The weight in the 10-th element of word's postings list is used here. 

$$ \\overline{sim(q,r)} = \\sum_{t\\in q} w\_{t,q} \\times \\overline{w\_{t,r}}.$$

* We choose k as 20, this acts as a hyperparameter forthe search. Set k to be 20 as trial and error shows that this is optimal value.

* Let query be "My stomach hurts", After preprocessing the the vector representation will be ['stomach','hurt']

* Below we show how to algorithm calculate similarity of query.
<pre>
  word-weight-query = number of times word appears in the query / number of words in query


  1. retrive top 20 of posting list for each word in query
  2. find the list of all reviews. 
  3. for every review:
  4.   for every word:
  5.     if word exsists in the review:
  6.        score += word-weight-review * word-weight-query 
  7.     else:
  8.        score += word-weight-review20 * word-weight-query

</pre>

<h2> Challenges faced </h2>
1. Implementing the count of words in each word. Used the concept of positional indexing.
2. Getting the posting list for each word. It was computationally expensive to sort every posting list and keep them stored prior to the query similarity calculation. Sort only the posting list being retrived while calculating the similarity.

<h4> Contributions: </h4>
1. Implemented my own Inverted index from scratch using numpy and pandas.
2. Created A way to quickly retrive the posting lists for similarity calculation.
3. Improving efficiency by dropping words that appear too many times from the vocabulary.
4. Used **nltk's** **_WordNetLemmatizer_** for feature selection.

<h5> Referrences</h5>
* (https://nlp.stanford.edu/IR-book/html/htmledition/a-first-take-at-building-an-inverted-index-1.html)
* [Sample code given](https://colab.research.google.com/drive/1n1hUx-mO4EqhKyFmN--9pK5MhKR68MpB#scrollTo=8ILVjili5Xmu)


Continue reading to implement Multinomial Naive Bayes classifier.
[Part 2](https://sananara-aryabhata.netlify.com/post/first_search_engine/)

