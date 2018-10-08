Sightline Visualization Discovery Platform
======================================

CSCI 475/476 Senior Design with Prof. Brian King


_The MQB Team_

Zhengri Fan,  Jiayu Huang, Henry Kwan, Xiaoying Pu

---
Bucknell University Summer 2017 Research Project with Prof. Evan Peck

Nick Simons, Adon Shapiro


***NOTE: *** **This extension is deprecated.  For any questions, please contact one of the contributors.**

Table of Contents
---
<!-- MarkdownTOC -->

- Abstract
- Project Organization and Architecture
    - Backend
    - Dependencies
    - Front End
- Optimization over the Baseline Iteration
    - Better Recommendations
- How to Compile and Install
    - Running the server
    - Installing and Using the Chrome Extension
    - How to use the extension
    - Maintenance and Troubleshooting
- How to Test
    -
- Repo Organization

<!-- /MarkdownTOC -->


# Abstract

Our client, Professor Evan Peck of the Department of Computer Science, requested to extend his Sightline visualization discovery platform. We, the MQB team, developed an intuitive Chrome extension that leverages the Sightline database and suggests relevant visualizations to provide a data-centric browsing experience. The backend of the extension applies natural language processing (NLP) to understand web content and ranks the visualization suggestions with a scoring method.

# Project Organization and Architecture

Overall, this project is mainly composed of a front end and a back end. Our REST API connects the two parts together.


## Backend
Our backend can be encapsulated as in Figure 1, in terms of functionality.


![](https://gitlab.bucknell.edu/xp002/csci475/raw/master/figures/backend.png)
_Figure 1. The three major components of our backend._

The backend system consists of three major components: Source Analyzer, Candidate Filter, and Front End Interface. Source Analyzer extracts the properties of the input, then the Candidate Filter will query the database based on those properties extracted, and it will generate the top candidates to recommend. Finally, the Front End Interface will send the filtered recommendations to the front end chrome extension.



### Source Analyzer:

We built the source extractor with the IBM Alchemy API. This API can be used to analyze the website to extract taxonomies and concepts in the website. The input for the source extractor would be the URL of user’s current page. The output would be a list of concepts and taxonomies associated with the input URL. The IBM interface does the heavy lifting of natural language processing, thus affording us with better backend latency.

Related function calls:


```
url_extractor()
queryTaxonomy()
taxonomy_extractor()
concept_extractor()
```


### Candidate Filter:

After we have acquired the taxonomies and concepts from the Source Analyzer, we use these items as conditions to filter our visualization database. The visualization database is a standard PostgreSQL database that can be queried in standard SQL command. The database stores concepts and taxonomies associated with each visualization. We filter the database by taxonomy first, and then we check if the resulting candidate pool is small enough to be presented. If we get too many candidates, we will score each visualization based on their concepts similarity, compared to the original web page. Each concept is essentially a word, so we compare the concepts from candidates with those from the website with the NLTK Wordnet. Each score we assign is the sum of similarities across all concepts of one particular candidate. With all candidates scored, the top five candidates are our final output to the front end.

Related function calls:

```
filterdict()
compare_concept()
process_list()
```



### Front End Filter:

We use the Flask framework to connect our backend and frontend. The backend extractor and filter are wrapped into python functions and can be easily accessed with a URL (/extract) associated with the flask framework. In this way, we are able to let the chrome extension front end pass information to the backend easily.

Related definitions:

```
@app.route("/extract")
def extract():
```

## Dependencies

Currently, the backend is hosted on a virtual machine provided by ECST (ecst@bucknell.edu). The address of the machine is 172.20.20.20, and the backend API is on port 5004. Given the aforementioned setup, http://172.20.20.20:5004/extract?url=google.com can be used to test the backend connection. This virtual machine will be deleted after Zhengri (zf002@bucknell.edu) graduates, so we provide only the information on how to recreate the setup of the machine.

There are two main things on the virtual machine: the backend server and the database.


Our server mainly depends on several Python packages:

-   watson\_developer\_cloud: IBM cloud services
-   alchemyapi: Alchemy API now deprecated!)
-   psycopg2: database queries
-   nltk: natural language processing
-   sematch.semantic.similarity: WordNet similarity
-   flask: web microframework


By using pip to install those dependencies listed in the server code, we currently have the following python packages installed on the virtual machine. Pip will resolve and install most of these dependencies.

-   appdirs==1.4.2
-   backports.ssl-match-hostname==3.4.0.2
-   beautifulsoup4==4.5.3
-   boto==2.44.0
-   chardet==2.2.1
-   Cheetah==2.4.4
-   click==6.7
-   configobj==4.7.2
-   decorator==3.4.0
-   Flask==0.12
-   html5lib==0.999999999
-   iniparse==0.4
-   IPy==0.75
-   isodate==0.5.4
-   itsdangerous==0.24
-   Jinja2==2.9.5
-   jsonpatch==1.2
-   jsonpointer==1.9
-   kitchen==1.1.1
-   Markdown==2.4.1
-   MarkupSafe==0.23
-   networkx==1.11
-   nltk==3.2
-   numpy==1.11.0
-   packaging==16.8
-   perf==0.1
-   Pillow==2.0.0
-   policycoreutils-default-encoding==0.1
-   prettytable==0.7.2
-   psycopg2==2.6.2
-   pyasn1==0.1.9
-   pycurl==7.19.0
-   Pygments==1.4
-   pygobject==3.14.0
-   pygpgme==0.3
-   PyGreSQL==4.2
-   pyliblzma==0.5.3
-   pyparsing==1.5.7
-   pysolr==3.6.0
-   pyudev==0.15
-   pyxattr==0.5.1
-   PyYAML==3.10
-   rdflib==4.0.1
-   requests==2.13.0
-   rsa==3.4.1
-   scikit-learn==0.17.1
-   scipy==0.13.2
-   sematch==1.0.3
-   seobject==0.1
-   sepolicy==1.1
-   six==1.9.0
-   SPARQLWrapper==1.5.2
-   urlgrabber==3.10
-   urllib3==1.10.2
-   watson-developer-cloud==0.25.0
-   webencodings==0.5
-   Werkzeug==0.11.15
-   xmltodict==0.10.2
-   yum-metadata-parser==1.1.4


The production version Sightline database is currently hosted at Amazon Web Service (AWS). For the database URL, username, and password, please contact professor Peck for authorization and detail.

Our database on the backend is not the production database of sightlinevis.com but a copy of it. On the virtual machine, the following information will help access the database:

-   Database name: postgres
-   User: postgres
-   Password: 00000000
-   Host: localhost
-   Port: 5432

In addition, the AlchemyAPI requires an API key. The API key is stored as an entry in config.json.


Currently, our API key has expired. Any attempt to access the /extract API now will result in watson_developer_cloud.watson_developer_cloud_service.WatsonException.

## Front End

Our Chrome Extension was built mainly using JavaScript. The strict design, what we see as HTML and CSS, is using Twitter’s Bootstrap 3. We chose this because it allowed us to lay out graphs and captions in an organized and columned format. In order to connect the backend to the front end, we used JavaScript--namely JQuery. As our extension focuses on a recommendation system, we needed to grab the handful of visualizations recommended by our system and display them onto the extension. JQuery was especially useful for this because it allowed us to get an element by a specific ID tag. Based on the current web page the user is viewing, we are able to use our backend recommendation system to suggest visualizations similar to the main topics mentioned on the web page. After retrieving the suggested visualizations, we are able to display them in a nice manner in our chrome extension along with the title for each respective element. We added hyperlinks to each title so that each visualization title links to the original web page on which the visualization is located.


# Optimization over the Baseline Iteration

Since we built upon a previous iteration of this Chrome Extension, we made improvements in terms of recommendation quality, latency/performance, and portability.


## Better Recommendations

Our recommendation algorithm is more sophisticated than the baseline version; the dataflow of the system is shown in Figure 2. Since the recommendation is based on comparing concepts or taxonomies, using more concepts has the potential to achieve a better result. Therefore, we have made a concept extrapolator that could expand on any given concept extracted from the web page that the user is currently browsing. As a result, we were  able to have up to 2x of concepts for comparison. In addition, the baseline version narrowed down the results with a filtering method, and hence the results are not ranked by how relevance they are. In order to address this deficiency, we implemented a scoring method so that the higher the score, the more relevant the visualization is. The scoring system is shown in Figure 3.

![](figures/CSCI_476_1.jpg)
_Figure 2. Dataflow of our recommendation algorithm._

![](figures/mtx.png)
_Figure 3. The scoring matrix._

In the matrix in Figure 3, the first column contains concepts extracted or extended from the page that the user is currently browsing. The first row contains the concepts of the visualization that are considered as relevant. As we can see from the previous matrix, each empty slot denotes to a score that is calculated based on the semantic similarity of the two concepts, and the relevance score that is returned from the API and the database for the web page and the visualization respectively. After each individual score is computed, they are summed to form a final score that would be used to justify how “relevant” the visualization is to the page.

`filterdict()` contains the implementation of the algorithm described in Figure 3.


# How to Compile and Install

Pull the repo at `git@gitlab.bucknell.edu:xp002/csci475.git`.

## Running the server

Some configurations are stored in config.json for easy access. The developer can change:

1. The host address of the backend.
2. The port number of the backend.
3. The AlchemyAPI key.

After configuration, run the python code process_service.py, and the developer should be able to access the server with this URL: <host>:<port>/extract?url=<google.com>

## Installing and Using the Chrome Extension

__WARNING__: Before distributing the extension, modify the host address and port number in getImage.js, line 46.

### Steps

1. Go to the repository and download the extension into a single folder on your local machine.
2. Open Chrome, go to address chrome://extensions/
3. On the “Extensions” page, click “Load unpacked extension …”
4. Select the folder you just downloaded (Figure 4).
5. The extension is now successfully installed (Figure 5).

__WARNING__:


Chrome may want to disable “developer mode”. Click “Cancel” on the dialog box to continue to use the extension (Figure 6).

![](figures/select_folder.png)
_Figure 4. Select the folder you just downloaded when loading unpacked extension._

![](figures/Extensions.png)
_Figure 5. chrome://extensions/ page when the extension is successfully installed._

![](figures/warning.png)
_Figure 6. Possible Chrome dialog box to disable developer mode._


## How to use the extension
You are only one click away from seeing visualization recommendations from our Sightline visualization collection.
When browsing a web page, click on the Chrome icon.
The extension pops up and loads for a couple of seconds (Figure 7).
Recommendation is shown (Figure 8).


![](figures/new_load.png)

_Figure 7. Loading screen._


![](figures/after.png)
_Figure 8. Extension loaded, showing recommendations._



## Maintenance and Troubleshooting
The Chrome extension depends on a backend API service. If and when the service is offline, the extension will not load. To check if the service is in working order:

1. Copy and paste the following URL into your search box: http://172.20.20.20:5004/extract?url=google.com (or whatever host and port you are currently using.)
2. If no text (JSON) shows up, the service is offline. Troubleshoot the backend.



### Warnings

- The extension will not load on the “New Tab” page in Chrome.
- If the extension does not load for a long time, the backend may no longer be online. See the maintenance section for troubleshooting advice.
- The API service is on bucknell.edu local network only. Please make sure that you are connected the Bucknell network via ethernet, Wi-Fi, or VPN.


# How to Perform User Studies

Nick Simons and Adon Shapiro developed a method for performing user studies.  Users will compare results from four conditions: Automatic Google, Automatic Sightline, Manual Google, and Manual Sightline.  We envision user studies in which participants will interact with a 3-monitor system.  One monitor will display the automatic conditions, one will display the manual conditions, and one will display a random Wikipedia page which will serve as the basis for the searches.

## Manual Conditions

After a participant has viewed the presented Wikipedia page, they may perform manual searches on Google and the Sightline database for further information and visualizations on the Wikipedia page's topic.

### Manual Google

The Manual Google Condition will consist of a Chrome window in which the participant will be able to complete a regular Google Search for information relevant to the topic of the Wikipedia page they viewed.  Participants will be able to click and view the Google results.

### Manual Sightline

This condition will consist of a Chrome window in which the participant will be able to complete a search on the Sightline Database for information relevant to the topic of the Wikipedia page they viewed.  Participants will again be able to click and view the results.

## Automatic conditions

Both manual conditions will be displayed on the second monitor, and are identical in appearance.  When the user clicks the SendURL extension icon in Chrome, both of the automatic conditions will load results.  Users will be able to click and view the results for each automatic condition.

## Installing and Using the SendURL User Study Chrome Extension

### Steps

1. Go to the sendURL/extension folder of the repository and download the extension into a single folder on your local machine.
2. Open Chrome, go to address chrome://extensions/
3. On the “Extensions” page, click “Load unpacked extension …”
4. Select the folder you just downloaded.
5. The extension is now successfully installed.

__WARNING__:


Chrome may want to disable “developer mode”. Click “Cancel” on the dialog box to continue to use the extension.

## How to use the sendURL extension to run user studies
### Steps
1. From the virtual machine, run the server (See "Running the server" in "How to Compile and Install")
2. On your local machine, enter the server directory for the sendURL extension
```
cd sightline-discovery/sendURL/server
```
3. From the current directory, run the python file getURL.py ***in Python 2.7***. Two empty Firefox windows should appear. Arrange them on the screens as you see fit.
  - Note: You may need to use pip to install some dependencies here.
  - Note: If nothing appears in the Firefox windows when you click the extension icon, visit address localhost:5000.  Then, click "ADVANCED", and click "Proceed to localhost (unsafe)". Clicking the sendURL extension icon should now cause results to load in the Firefox windows.
4. On your local machine, in your regular Chrome browser, visit a desired web page.
  -Note: We suggest you run the python file rand_wp.py to visit a randomly chosen, popular Wikipedia article.
5. When you have located the desired web page, click the sendURL extension icon (it should appear as the Google Chrome logo).  Wait for the links to appear in the two Firefox windows.
6. Allow the user to click the links in the Firefox windows and briefly read through the resulting pages.  Record the user's reactions using whatever method you deem appropriate.

# Repo Organization

```
.
├── Chrome-Extension-Bootstrap (front end related)
│   ├── README.md
│   ├── css
│   │   ├── bootstrap.min.css
│   │   └── custom.css
│   ├── fonts
│   │   ... 
│   │  
│   ├── img
│   │   ... 
│   │  
│   ├── js
│   │   ├── bootstrap.min.js
│   │   ├── getImage.js
│   │   ├── jquery-2.0.3.min.js
│   │   ├── loader.js
│   │   └── loading.js
│   ├── manifest.json
│   └── popup.html
├── README.md
├── figures (figures for README.md)
│   ├── ...
├── baseline_iteration (the baseline iteration of the extension)
│   └── sightline_extension
│       └── scripts
├── docs (documentations)
│   ...
├── sendURL
|   ├── extension
│   └── server
└── source (back end source code)
    ├── backend (server source code)
    │   ├── alchemyapi.py
    │   ├── api_key.txt
    │   ├── config.json
    │   ├── icon.png
    │   ├── process_service.py
    │   ├── requirements.txt
    │   └── test_service.py
    ├── data (experiment data)
    │   ├── sightlinevis.json
    │   └── urls.txt
    ├── python_postgres_test (script for database connection)
    │   └── test.py
    └── similarity_research (experiment with similarity)
        ├── out.txt
        ├── sim_env
        └── similarity_prototype.py
```
