# Building Recommender Systems with Machine Learing and AI

## Section 1: Getting Started

	+ Youtube's candidate generation:
		```
			top-N <--- Knn index <---video vectors---- softmax ---> class probabilities
					^				^	
					|				|				
				user vector 				| 		
					|				|				
					|__________ ReLU _______________|
					 	 	^
					 	 	|
					 __________ ReLU ____________
					 	 	^
					 	 	|
					 __________ ReLU ____________	
					 	 	^
					 	 	|
		watch vector  search vector	 geographic  age  gender  ...
		     ^		 ^
		     |		 |
		  average	average

		  |||||..||	|||||..||
	   video watches   search tokens		 
	   ```

	+ (one) anatomy of a top-N recommender
		```
		individual interests --> candidate generation <--> item similarities
									|
									candidate ranking
									|
									filtering
									|
									output
		```
	+ autoencoders for recommendations ("autorec")
		```
		R1i	R2i	R3i ... Rmi		(+1)
			M1i M2i...
		R1i R2i R3i ... Rmi     (+1)
		```
	+ Frameworks: Tensorflow, DSSTNE, Sagemaker, ApacheSpark.

	+ Install Anaconda: 
		+ Notes: To overcome the PermissionError issue, after creating the RecSys environment, open its terminal, then run the command `nohup sudo spyder &` to get through this error. You may do the same if you use JupyterLab or others, just replace spyder by that application. 

	+ Getting Started
		+ What is a recommender system?
			+ RecSys is NOT a system that recommends arbitrary values, that describes machine learning in general.
				+ For example,
					+ A system that recommends prices for a house you're selling is NOT a recommender system.
					+ A system that recommends whether a transaction is fraudulent is NOT a recommender system.
				+ These are general ML problems, where you'd apply techniques such as regression, deep learning, xgboost, etc.

			+ A recommender system that predicts ratings or preferences a user might give to an item. RecSys recommends on things based on the people's past behaviors. Often these are sorted and presented as top-N recommendations, aka, recommender engines or platforms.

			+ Customers don't want to see your ability to predict their rating for an item, they just want to see things they're likely to love.


			+ implicit ratings: purchase data, video viewing data, click data. (by product of user's natural behavior)
			+ explicit ratings: star reviews. (ask user reviewing on something)


		+ `GettingStarted.py`				
			+ Use `SVD` singular value decomposition. [explained](https://www.youtube.com/watch?v=P5mlg91as1c)
				+ A[m x n] = U[m x r] * sigma{r x r}(V[n x r])^T
					+ A[m x n]: (rows x cols)		
						+ documents x terms(different words)
							* a given document (in a row) contains a list of terms/words (in columns)
						+ users x movies:
							* a given user (in a row) watches a list of movies (in columns)  
					+ U: Left singular vectors (i.e. user-to-concept similarity matrix)
						+ m x r matrix (m documents, r concepts)
					+ sigma: singular values (i.e. its diagonal elements: strength of each concept)
						+ r x r diagonal matrix (strength of each 'concept' in r)
						+ r: rank of the matrix A
					+ V: right singular vectors (i.e. movie-to-concept similarity matrix)
						+ n x r matrix (n terms, r concepts)

					A = U*sigma*V^T = sum{i}sigma_i*u_i*v_i

				+ [SVD case study](https://www.youtube.com/watch?v=K38wVcdNuFc)

				+ Compare the SVD algorithm to KNNBasic:
					+ test MAE score: SVD(0.2731) KNN(0.6871)
					+ test RSME score: SVD(0.3340) KNN(0.9138)

					+ SVD (user 81)
						We recommend:
							Gladiator (1992)
							Lord of the Rings: The Fellowship of the Ring, The (2001)
							To Kill a Mockingbird (1962)
							Ghost in the Shell (KÃ´kaku kidÃ´tai) (1995)
							Godfather: Part II, The (1974)
							Seven Samurai (Shichinin no samurai) (1954)
							African Queen, The (1951)
							Memento (2000)
							Band of Brothers (2001)
							General, The (1926)

					+ KNN (user 81) 
						We recommend:
							One Magic Christmas (1985)
							Art of War, The (2000)
							Taste of Cherry (Ta'm e guilass) (1997)
							King Is Alive, The (2000)
							Innocence (2000)
							MaelstrÃ¶m (2000)
							Faust (1926)
							Seconds (1966)
							Amazing Grace (2006)
							Unvanquished, The (Aparajito) (1957)

					+ Take-away note: Apparently, SVD performs better than KNN in this scenario.


## Section 2: Intro to Python 

## Section 3: Evaluating Recommender Systems

	+ Train/test/crossvalidation
		+ full data -> train and test
		+ trainset -> machine learning -> fit 
		+ use trained model for testing on test data.

		+ K-fold cross validation
			+ Bagging: full data -> fold i-th -> ML -> measure accuracy -> take average 

	+ Accurate metrics RMSE/MAE
		+ MAE: lower is better
			sum{1..n}|yi - xi|/n
		+ RMSE: lower is better
			+ it penalizes you more when your prediction is way off, and penalizes you less when you are reasonably close. It inflates the penalty for larger errors.

	+ Top-N hit rate - many ways
		+ evaluating top-n recommenders:
			+ Hit rate: you generate top-n recommendations for all of the users in your test set, if one of top-n recommendations is rated, it's a hit. Thus, hit rate = count of all hits per total users 
		+ leave-one-out cross validation:
			+ compute the top-n recommendations for each user in our training data, 
			+ then intentionally remove one of those items from that user's training data. 
			+ then test our recommender system's ability to recommend that item that was left out in the top-n results it creates for that user in the testing phase.
		+ Notes: hit rate with leave-one-out are working better with a very large dataset.

		+ Average reciprocal hit rate (ARHR):
			+ sum{1..n}(1/rank(i))/users
			+ it measures our ability to recommend items that actually appeared in a user's top-n highest rated movies, it gives more weight to these hits when they appear near the top of the top-n list.

		+ cumulative hit rate (cHR):
			+ throw away hits if our predicted rating is below some threshold. The idea is that we shouldn't get credit for recommending items to a user that we think they won't actually enjoy.

		+ rating hit rate (rHR)
			+ break it down by predicted rating score. The idea is that we recommend movies that they actually liked and breaking down the distribution gives you some sense of how well you're doing in more detail.
		+ Take-away: RMSE and hit rate are not always related.

	+ Coverage, Diversity, and Novelty Metrics
		+ Coverage: 
			+ the percentage of possible recommendations that your system is able to provide.
				`% of <user,item> pair that can be predicted.`
			+ Can be important to watch bcos it gives you a sense of how quickly new items in your catalog will start to appear in recommendations. i.e., when a new book comes out on Amazon, it won't appear in recommendations until at least few people buy it. Therefore, establishing patterns with the purchase of other items. Until those patterns exist, that new book will reduce Amazon's coverage metric.

		+ Diversity:
			+ How broad a variety of items your recommender system is putting in front of people. Low diversity indicates it recommends next books in the series that you've started reading, but doesn't recommend books from different authors, or movies related to what you've read/watched. 
			+ We can use similarity scores to measure diversity.
			+ If we look at the similarity scores of every possible pair in a list of top-n recommendations, we can average them to get a measure of how similar the recommended items in the list are to each other, called S.
			+ Diversity = 1 - S
				+ S: avg similarity between recommendation pairs.

		+ Novelty:
			+ mean popularity rank of recommended items.



	+ Churn, Responsiveness, and A/B Tests
		+ How often do recommendations change?
		+ perceived quality: rate your recommendations.
		+ The results of online A/B tests are the metric matters more than anything. 

	+ Review ways to measure your recommender
	+ Recommender Metrics
		+ Surprise package is about making rating predictions, and we need a method to get top-n recommendations out of it. 
	+ Test Metrics
		+ run the test.
			```
			Loading movie ratings...

			Computing movie popularity ranks so we can measure novelty later...

			Computing item similarities so we can measure diversity later...
			Estimating biases using als...
			Computing the pearson_baseline similarity matrix...
			Done computing similarity matrix.

			Building recommendation model...

			Computing recommendations...

			Evaluating accuracy of model...
			RMSE:  0.9033701087151801
			MAE:  0.6977882196132263

			Evaluating top-10 recommendations...
			Computing recommendations with leave-one-out...
			Predict ratings for left-out set...
			Predict all missing ratings...
			Compute top 10 recs per user...

			Hit Rate:  0.029806259314456036

			rHR (Hit Rate by Rating value): 
			3.5 0.017241379310344827
			4.0 0.0425531914893617
			4.5 0.020833333333333332
			5.0 0.06802721088435375

			cHR (Cumulative Hit Rate, rating >= 4):  0.04960835509138381

			ARHR (Average Reciprocal Hit Rank):  0.0111560570576964

			Computing complete recommendations, no hold outs...

			User coverage:  0.9552906110283159 [this is good]
			Computing the pearson_baseline similarity matrix...
			Done computing similarity matrix.

			Diversity:  0.9665208258150911 [this is not good, too high]

			Novelty (average popularity rank):  491.5767777960256
									[this is not good, too high][as long tail in distribution]				
			```
	+ Measure the performance of SVD recommender.
		

## Section 4: A Recommender Engine Framework 


## Section 5: Content-Based Filtering 

## Section 6: Neighborhood-Based Collaborative Filtering 

## Section 7: Matrix Factorization Methods 

## Section 8: Intro to Deep Learning 

## Section 9: Deep Learning for Recommender Systems

## Section 10: Scaling it Up 

## Section 11: Real-World Challenges of Recommender Systems 

## Section 12: Case Studies

## Section 13: Hybrid Approach 

## Section 14: Wrap Up
