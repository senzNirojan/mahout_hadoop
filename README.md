# mahout_hadoop

Building a Recommender
To demonstrate how to build an analytic job with Mahout on EMR, we’ll build a movie recommender. We will start with ratings given to movie titles by users in the MovieLens data set, which was compiled by the GroupLens team, and will use the “recommenditembased” example to find most-recommended movies for each user.

1. Sign up for an AWS account.

2. Start up an EMR cluster (note the pricing and make sure to shut the cluster down afterward).
	Go to services in AWS console and create an EMR cluster.

3. ssh into the server
	ssh -i ~/msc_auth.pem hadoop@ec2-54-234-30-200.compute-1.amazonaws.com

4. Get the MovieLens data
	wget http://files.grouplens.org/datasets/movielens/ml-1m.zip
5. unzip ml-1m.zip

6. Convert ratings.dat, trade “::” for “,”, and take only the first three columns:
	cat ml-1m/ratings.dat | sed 's/::/,/g' | cut -f1-3 -d, > ratings.csv

7. Put ratings file into HDFS:
	hadoop fs -put ratings.csv /ratings.csv

8. Run the recommender job:
	mahout recommenditembased --input /ratings.csv --output recommendations --numRecommendations 10 --outputPathForSimilarityMatrix similarity-matrix --similarityClassname SIMILARITY_COSINE

9. Look for the results in the part-files containing the recommendations:
        hadoop fs -ls recommendations
        hadoop fs -cat recommendations/part-r-00000 | head
You should see a lookup file that looks something like this (your recommendations will be different since they are all 5.0-valued and we are only picking ten):

User ID	(Movie ID : Recommendation Strength) Tuples
35	[ 2067:5.0, 17:5.0, 1041:5.0, 2068:5.0, 2087:5.0, 1036:5.0, 900:5.0, 1:5.0, 2081:5.0, 3135:5.0 ]
70	[ 1682:5.0, 551:5.0, 1676:5.0, 1678:5.0, 2797:5.0, 17:5.0, 1:5.0, 1673:5.0, 2791:5.0, 2804:5.0 ]
105	[ 21:5.0, 3147:5.0, 6:5.0, 1019:5.0, 2100:5.0, 2105:5.0, 50:5.0, 1:5.0, 10:5.0, 32:5.0 ]
140	[ 3134:5.0, 1066:5.0, 2080:5.0, 1028:5.0, 21:5.0, 2100:5.0, 318:5.0, 1:5.0, 1035:5.0, 28:5.0 ]
175	[ 1916:5.0, 1921:5.0, 1912:5.0, 1914:5.0, 10:5.0, 11:5.0, 1200:5.0, 2:5.0, 6:5.0, 16:5.0 ]
210	[ 19:5.0, 22:5.0, 2:5.0, 16:5.0, 20:5.0, 21:5.0, 50:5.0, 1:5.0, 6:5.0, 25:5.0 ]
245	[ 2797:5.0, 3359:5.0, 1674:5.0, 2791:5.0, 1127:5.0, 1129:5.0, 356:5.0, 1:5.0, 1676:5.0, 3361:5.0 ]
280	[ 562:5.0, 1127:5.0, 1673:5.0, 1663:5.0, 551:5.0, 2797:5.0, 223:5.0, 1:5.0, 1674:5.0, 2243:5.0 ]
Where the first number is a user id, and the key-value pairs inside the brackets are movie-id:recommendation-strength tuples.

Building a Service
Next, we’ll use this lookup file in a simple web service that returns movie recommendations for any given user.

10. Get Twisted, and Klein and Redis modules for Python.
        sudo pip3 install twisted
        sudo pip3 install klein
        sudo pip3 install redis
Install Redis and start up the server.

11.
        wget http://download.redis.io/releases/redis-2.8.7.tar.gz
        tar xzf redis-2.8.7.tar.gz
        cd redis-2.8.7
        make
        ./src/redis-server &

12. Build the web service using the hello.py in this repository.
   

13. Start the web service.
	twistd -noy hello.py &

14. Test the web service with user id “37”:
	curl localhost:8080/37

You should see a response like this (again, your recommendations will differ):
The recommendations for user 37 are [7:5.0,2088:5.0,2080:5.0,1043:5.0,3107:5.0,2087:5.0,2078:5.0,3108:5.0,1042:5.0,1028:5.0]
When you’re finished, don’t forget to shut down the cluster in AWS console.
