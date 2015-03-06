---
layout: page
title: Using a shell variable with jq
---

I have been learning the basics of [jq](http://stedolan.github.io/jq/). I ran into a scenario 
where I wanted to use jq but refer to a shell variable in a bash script. 

It turns out that this case is neatly described in the jq manual:

>`--arg name value:`
>
>This option passes a value to the jq program as a predefined variable. If you run jq with `--arg foo bar`, then `$foo` is available in the program and has the value "bar".

Clear enough, but I understand things much better after I've tried them out, so I will share with you my toy example. 

Suppose you have a json with a list of some movies, and you want to see if since more and more remakes
occured, if the franchise had begun to go downhill. For example consider homealone.json:

```
{
    "Films": [
        {
            "Title": "Home Alone",
            "Year": "1990",
            "Plot": "An 8-year old troublemaker must protect his home from a pair of burglars when he is accidentally left home alone by his family during Christmas vacation.",
            "imdbRating": "7.4"
        },
        {
            "Title": "Home Alone 2",
            "Year": "1992",
            "Plot": "N/A",
            "imdbRating": "5.7"
        },
        {
            "Title": "Home Alone 3",
            "Year": "1997",
            "Plot": "Alex Pruitt, a young boy of nine living in Chicago, fend off thieves who seek a top-secret chip in his toy car to support a North Korean terrorist organization's next deed.",
            "imdbRating": "4.2"
        },
        {
            "Title": "Home Alone 4",
            "Year": "2002",
            "Plot": "Kevin McCallister's parents have split up. Now living with his mom, he decides to spend Christmas with his dad at the mansion of his father's rich girlfriend, Natalie. Meanwhile robber Marv...",
            "imdbRating": "2.3"
        },
        {
            "Title": "Home Alone 5",
            "Year": "2012",
            "Plot": "N/A",
            "imdbRating": "3.9"
        }
    ]
}
```

Below is a script I used to print out the title and the rating for each movie. I think there is certainly a more efficient way to extract this information, but the purpose of making this script was to just get a feel for using shell variables with jq. 

```bash
#!/bin/bash
###################################################
# Toy script to process with jq a json file with 
# a list of movies (where each movie has a Title 
# and an imdbRating) and print the title and rating 
# of each to the console.
####################################################

FILMFILE=homealone.json
usage() { echo "Usage: $0 [-f filmfile -- defaults to homealone.json ]" 1>&2; exit 1; }

while getopts ":f:" o; do
  case "${o}" in
    f) FILMFILE=${OPTARG}
       ;;
    *) usage 
       ;;   
  esac
done
shift $((OPTIND-1))

NUM_FILMS=$(cat $FILMFILE | jq ".Films | length")
declare -i COUNTER
COUNTER=0
while [[ ${COUNTER} -lt $NUM_FILMS ]] ; do
  RATING=$(cat $FILMFILE | jq --arg jqCounter $COUNTER '.Films[$jqCounter|tonumber].imdbRating') 
  MOVIE_TITLE=$(cat $FILMFILE | jq --arg jqCounter $COUNTER '.Films[$jqCounter|tonumber].Title') 
  echo "Rating of $MOVIE_TITLE is $RATING"

  let COUNTER=COUNTER+1 
done
```

Which shows me:

```
lma$ ./ratingfetcher.sh 
Rating of "Home Alone" is "7.4"
Rating of "Home Alone 2" is "5.7"
Rating of "Home Alone 3" is "4.2"
Rating of "Home Alone 4" is "2.3"
Rating of "Home Alone 5" is "3.9"
```

Conclusion: Home Alone is no exception to the remakes not measuring up to the original. 