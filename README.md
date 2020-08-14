# Yelp-Recommendation-System(PySpark)
## Yelp Data 
In this assignment, we generated the review data from the original Yelp review dataset with some filters such as the condition: "state"=="CA". We randomly took 80% of 
data for training, 10% of data for testing, and 10% of the data as the blind dataset.
### Task1: Min-Hash + LSH
#### Implement the Min-Hash and Locality Sensitive Hashing algorithms with Jaccard similarity to find similar business pairs in the train_review.json file:
1. focus on the 0 or 1 ratings rather than the actual ratings/stars in the reviews. Specifically, if a user has rated a business, the user's contribution in the characteristic
matrix is 1. If the user hasn't rated the business, the contribution is 0. Your task is to identify business pairs whose Jaccard similarity is >= 0.05.
2. Define any collection of hash functions that you think would result in a consistent permutation of the row entries of the characteristic matrix.
3. Build the signature matrix using Min-Hash
4. Divide the matrix into b bands with r rows each, where bXr=n(n is the number of hash functions). You need to set b and r properly to balance the number of candidates and the 
computational cost.
5. Two businesses become a candidate pair if their signatures are identical in at least one band.
6. Verify the candidate pairs using their original Jaccard similarity
#### Output file:
You must write a business pair and its similarity in the JSON format using exactly the same tags as example. Each line represents for a business pair ("b1","b2").
There us no need to have an output for ("b2","b1")
example: {"b1":"cYwJ...","b2":"Fid2...","sim":0.0324...}
         {"b1":"7zec...","b2":"1Vvx...","sim":0.0188...}
#### Execution example:
spark-submit task1.py <input_file> <output_file> 

### Task2: Content-based Recommendation System
Buld a content-based recommendation system by generationg profiles from review texts for users and businesses in the train review set. Then you will use the 
system/model to predict if a user prefers to review a given business, i.e., computing the cosine similarity between the user and item profile vectors.
#### training process: contruct business and user profiles as the model
1. Concatenating all the review texts for the business as the document and parsing the document, such as removing the punctuations, numbers, and stopwords. Also, you can remove
extremelt rare words to reduce the vocabulary size, i.e., the cound is less than 0.0001% of the total words.
2. Measuring word importance using TF-IDF, i.e., term frequency*inverse doc frequency
3. Using top 200 words with highest TF-IDF scores to describe the document
4. Creating a Boolean vector with these significant words as the business profile
5. Creating a Boolean vector for representing the user profile by aggregating the profiles of the items that the user has reviewed
#### predicting process: 
1. Estimate if a user would prefer to review a business by computing the consine distance between the profile vectors
2. The (user,business) pair will be considered as a valid pair if their cosine similarity is >=0.01
#### Output file:
model format: there is no strict format for the content-based model
predict format: write the results in the JSON format using exactly the same tags as the example. Each line represents for a predicted pair of ("user_id","business_id")
exmaple: {"user_id":"1vXJ...","business_id":"Zzvff...","sim":0.6123...}
         {"user_id":"2svf...","business_id":"JAmQC...","sim":0.3421...}
#### Execution example:
1. spark-submit task2train.py <train_file><model_file><stopwords>
2. spark-submit task2predict.py <test_file><model_file><output_file>

### Task3: Collaborative Filtering Recommendation System
Buld a collaborative filtering recommendation systems with train reviews and use the models to predict the ratings for a pair of user and business
#### case1: item-based CF recommendation system
1. During training process, build a model by computing the Pearson correlation for the business pairs that have at least three co-rated users.
2. During the predicting process, use the model to predict the rating for a given pair of user and business.
3. Must use at most N business neighbors that are most similar to the target business for prediction
#### case2: user-based CF recommendation system with Min-Hash LSH
1. During training process, since the number of potential user pairs might be too large to compute, combine the Min-Hash and LSH algrithms in the user-based CF recommendation system.
2. Identify user pairs who are similar using their co-rated businesses without considering their rating scores(similar to Task 1). This process reduces the number of user pairs you need to compare for the final Pearson correlation score
3. Compute the Pearson correaltion ofr the user pair candidates that have Jaccard similarity >= 0.01 and at least three co-rated businesses. The predicting process is similar to Case 1
#### Output file:
model format: must write the model in the JSON format using exactly the same tags as the example. Each line represents for a business pair ("b1","b2") for item-based model or a suer pair("u1","u2")
for user-based model. The is no need to have ("b2","b2") or ("u2","u1")
example:
1. item-based: {"b1":"eZc...","b2":"fB4ff...","sim":0.354...}
               {"b1":"1vX...","b2":"HhVmD...","sim":0.620...}
2. user-based: {"u1":"eZc...","b2":"fB4ff...","sim":0.354...}
               {"u2":"1vX...","u2":"HhVmD...","sim":0.620...}
predict format: write a business pair and its similarity in the JSON format using exactly the same tags as the example. Each line represents for a predicted pair of ("user_id","business_id")
example: {"user_id":"...","business_id":"...","stars":3.607...}
         {"user_id":"...","business_id","...","stars":1.442...}
#### Execution example:
spark-submit task3train.py <train_file><model_file><cf_type>
spark-submit task3predict.py <train_file><test_file><model_file><output_file><cf_type>

