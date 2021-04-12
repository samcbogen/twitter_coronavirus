# Coronavirus Twitter Analysis

In this project, I scanned all geotagged tweets sent in 2020 to monitor for the spread of the coronavirus on social media.


**Main Goals:**

1. work with large scale datasets
1. work with multilingual text
1. use the MapReduce divide-and-conquer paradigm to create parallel code

## Background

Approximately 500 million tweets are sent everyday.
Of those tweets, about 1% are *geotagged*.
That is, the user's device includes location information about where the tweets were sent from.
The lambda server's `/data-fast/twitter\ 2020` folder contains all geotagged tweets that were sent in 2020.
In total, there are about 1.1 billion tweets in this dataset.
We can calculate the amount of disk space used by the dataset with the `du` command as follows:
```
$ du -h /data-fast/twitter\ 2020
```

The tweets are stored as follows.
The tweets for each day are stored in a zip file `geoTwitterYY-MM-DD.zip`,
and inside this zip file are 24 text files, one for each hour of the day.
Each text file contains a single tweet per line in JSON format.
JSON is a popular format for storing data that is closely related to python dictionaries.

Vim is able to open compressed zip files,
and I encourage you to use vim to explore the dataset.
For example, run the command
```
$ vim /data-fast/twitter\ 2020/geoTwitter20-01-01.zip
```
Or you can get a "pretty printed" interface with a command like
```
$ unzip -p /data-fast/twitter\ 2020/geoTwitter20-01-01.zip | head -n1 | python3 -m json.tool | vim -
```

You will follow the [MapReduce](https://en.wikipedia.org/wiki/MapReduce) procedure to analyze these tweets.
MapReduce is a famous procedure for large scale parallel processing that is widely used in industry.
It is a 3 step procedure summarized in the following image:

<img src=mapreduce.png width=100% />


**Runtime:**

The simplest and most common scenario is that the map procedure takes time O(n) and the reduce procedure takes time O(1).
If you have p<<n processors, then the overall runtime will be O(n/p).
This means that:
1. doubling the amount of data will cause the analysis to take twice as long;
1. doubling the number of processors will cause the analysis to take half as long;
1. if you want to add more data and keep the processing time the same, then you need to add a proportional number of processors.

More complex runtimes are possible.
Merge sort over MapReduce is the classic example. 
Here, mapping is equivalent to sorting and so takes time O(n log n),
and reducing is a call to the `_reduce` function that takes time O(n).
But they are both rare in practice and require careful math to describe,
so we will ignore them.
In the merge sort example, it requires p=n processors just to reduce the runtime down to O(n)...
that's a lot of additional computing power for very little gain,
and so is impractical.


## Project Tasks

1. I modified the `map.py` file so that it tracks the usage of the hashtags on both a language and country level.
   This required creating a variable `counter_country` similar to the variable `counter_lang`, 
   and working this variable in the `#search hashtags` section of the code.
   The output of running `map.py` included two files, one that ends in `.lang` for the lanuage dictionary,
   and one that ends in `.country` for the country dictionary.

1. Once the `map.py` file was modified to track results for each country,
   I ran the map file on all the tweets in the `/data-fast/twitter\ 2020` folder.
   In order to do this, I created a shell script `run_maps.sh` that looped over each file in the dataset and runs `map.py` on that file. I used the `nohup` command to ensure that the program ran after I disconnected, and I used the `&` operator to make sure that all of the commands in `map.py` ran in parallel. 

1. After my modified `map.py` ran on all the files,
   I had a large number of files in your `outputs` folder.
   I then used the `reduce.py` file to combine all of the `.lang` files into a single file,
   and all of the `.country` files into a single different file.
   Next, I used the `visualize.py` file to count the total number of occurrences of each of the hashtags. This file displays the output from running the `map.py` file.

   For each hashtag, I created an output file in my repo using output redirection with the following template:
   ```
   $ ./src/visualize.py --input_path=PATH --key=HASHTAG | head > visuals/HASHTAG
   ```
   
## Results

Ultimately, as can be seen in the json files, it is clear that there was a spike in the usage of the tracked hashtags in March 2020, particularly around March 10, which makes sense as it is right around the time when the first COVID-19 lockdown was instituted in the United States. The hashtags' usage was moderate before that, likely because COVID-19 was already prevalent in China prior to arriving in the United States. 


