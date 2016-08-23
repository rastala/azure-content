<properties
	pageTitle="Create text analytics models in Azure Machine Learning Studio | Microsoft Azure"
	description="How to create text analytics models in Azure Machine Learning Studio using modules for text preprocessing, N-grams or feature hashing"
	services="machine-learning"
	documentationCenter=""
	authors="rastala"
	manager=""
	editor=""/>

<tags
	ms.service="machine-learning"
	ms.workload="data-services"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="article"
	ms.date="08/19/2016"
	ms.author="roastala" />


#Create text analytics models in Azure Machine Learning Studio

You can use Azure Machine Learning to build and operationalize text analytics models, for example to perform document classification or sentiment analysis.

In a text analytics experiment, you would typically:

 1. Clean and preprocess text dataset
 2. Extract numeric feature vectors from pre-processed text
 3. Train classification or regression model
 4. Score and validate the model
 5. Deploy the model to production

In this tutorial, you learn these steps as we walk through a sentiment analysis model using ["Book Reviews from Amazon" dataset] (https://azure.microsoft.com/en-us/documentation/articles/machine-learning-use-sample-datasets). This dataset consists of review scores (1,2 or 4,5) and a free-form text. The goal of our is to predict the the review score: low (1,2) or high (4,5).

The experiments covered in this tutorial are available in Cortana Intelligence Gallery:

[Predict Book Reviews] (https://gallery.cortanaintelligence.com/Experiment/Predict-Book-Reviews-1)

[Predict Book Reviews - Predictive Experiment] (https://gallery.cortanaintelligence.com/Experiment/Predict-Book-Reviews-Predictive-Experiment-1)

## Step 1: Clean and preprocess text dataset

We begin the experiment by dividing the review scores into categorical low and high buckets to formulate the problem as two-class classification. We do this step using Edit Metadata and Group Categorical Values modules.

![Create Label] (./media/machine-learning-text-analytics-module-tutorial/create-label.png)

Then, we clean the text using Preprocess Text module. The cleaning will reduce the noise in the dataset, help you find the most important features, and improve the accuracy of the final model. We remove stopwords - very common words such as "the" or "a" - as well as numbers, special characters, duplicated characters such as "coool", email addresses and URLs. We also convert the text to lowercase, lemmatize the words and detect sentence boundaries that will be indicated by "|||" symbol in in pre-processed text.

![Preprocess Text] (./media/machine-learning-text-analytics-module-tutorial/preprocess-text.png)

Optionally, you can apply custom list of stopwords, or custom C# syntax regular expression to replace substrings, as well as remove words by part of speech: nouns, verbs or adjectives.

After the preprocessing is complete, we split the data into train and test sets.

## Step 2: Extract numeric feature vectors from pre-processed text

Machine learning algorithms typically expect numeric feature vectors as input instead of free-form text. We use Extract N-Gram Features module to transform the text data to such format. This module takes a column of whitespace-separated words and computes a dictionary of words, or N-grams of words, that appear in your dataset. Then, it counts how many times each word, or N-gram, appears in each record, and creates new feature vectors from those counts. In this tutorial, we set N-gram size to 2, so our feature vectors include single words and combinations of two subsequent words.

![Extract N-grams] (./media/machine-learning-text-analytics-module-tutorial/extract-ngrams.png)

We apply TF*IDF (Term Frequency Inverse Document Frequency) weighting to N-gram counts. This approach adds weight of words that appear frequently in a single record but are rare across the entire dataset. Other options include binary, TF and graph weighing.

Such text features often have a very high dimensionality. For example, if your corpus has 100,000 unique words, your feature space would have 100,000 dimensions, or more if N-grams are used. The Extract N-Gram Features module gives you a set of options to reduce the dimensionality. You can choose to exclude words that are very short, very long, or too uncommon or too frequent to have significant predictive value. In this tutorial, we exclude N-grams that appear in fewer than 5 records or in more than 80% of records.

Also, you can use feature selection to select only those features that are the most correlated with your prediction target. We use Chi-Squared feature selection to select 1000 features. You can view the vocabulary of selected words or N-grams by clicking the right output of Extract N-grams moddule.

As an alternative approach to using Extract N-Gram Features, you can use Feature Hashing module. Note though that Feature Hashing does not have build-in feature selection capabilities, or TF*IDF weighing.

## Step 3: Train classification or regression model

Now the text has been transformed to numeric feature columns. The dataset still contains string columns from previous stages, so we use Select Columns in Dataset to exclude them.

We then use Two-Class Logistic Regression to predict our target: high or low review score. Note that at this point the text analytics problem has been transformed into a regular classification problem, and you can use the tools available in Azure Machine Learning to improve the model. For example, you can experiment with different classifiers to find out how accurate results they give, or use hyperparameter tuning to improve the accuracy.

![Train and Score] (./media/machine-learning-text-analytics-module-tutorial/scoring-text.png)

## Step 4: Score and validate the model

To validate the model, we score it against the test dataset and evaluate the accuracy. However, note that the model learned the vocabulary of N-grams and their weights from the test dataset. We should use that vocabulary and those weights when extracting features from test data, as opposed to creating the vocabulary anew. To do this, we add Extract N-Gram Features module to the scoring branch of the experiment, connect the output vocabulary from training branch, and set the vocabulary mode to read-only. We also disable the filtering of N-grams by frequency by setting the minimum to 1 instance and maximum to 100%, and turn the feature selection off.

After the text column in test data has been transformed to numeric feature columns, we exclude the string columns from previous stages like in training branch, and then use Score Model module to make predictions and Evaluate Model module to evaluate the accuracy.

## Step 5: Deploy the model to production

The model is almost ready to be deployed to production. When deployed as web service, it should take free-form text string as input, and return a prediction "high" or "low", using the learned N-gram vocabulary to transform the text to features, and trained logistic regression model to make a prediction from those features. 

To accomplish this, we first save the N-gram vocabulary as dataset, and the trained logistic regression model from the training branch of the experiment. Then, we save the experiment using "Save As" to create a new experiment graph for predictive model. We remove the Split Data module and the training branch from the experiment, and connect the previously saved N-gram vocabulary and model to Extract N-Gram Features and Score Model modules, respectively. We also remove the Evaluate Model module.

As a final step of setting up the predictive experiment, we insert Select Columns in Dataset module before Preprocess Text module to remove the label column, and Score Model module we unselect "Append score column to dataset" option. That way, the web service will not request the label it is trying to predict, and it will not echo the input features in response.

![Predictive Experiment] (./media/machine-learning-text-analytics-module-tutorial/predictive-text.png)

Now we have an experiment that can be published as a web service and called using request-response or batch execution APIs.

## Additional information

[MSDN documentaton on text analytics modules] (https://msdn.microsoft.com/en-US/library/azure/dn905886.aspx)
