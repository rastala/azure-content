<properties
	pageTitle="Create text analytics models in Azure Machine Learning Studio | Microsoft Azure"
	description="How to create text analytics models in Azure Machine Learning Studio using modules for text preprocessing, N-grams, feature hashing and topic modeling"
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

You can use Azure Machine Learning to build and operationalize text analytics models, for example to perform document classification or sentiment analysis, or to cluster your text dataset into topics.

In a text analytics experiment, you would typically:

 1. Clean and preprocess text dataset
 2. Extract numeric feature vectors from pre-processed text
 3. Train classification or regression model
 4. Score and validate the model
 5. Deploy the model to production

In this tutorial, you learn these steps as we walk through a sentiment analysis model using "Book Reviews from Amazon" dataset. This dataset consists of review scores (1,2 or 4,5) and a free-form text. The goal of our is to predict the the review score: low (1,2) or high (4,5).

## Step 1: Clean and preprocess text dataset

We begin the experiment by dividing the review scores into categorical low and high buckets to formulate the problem as two-class classification. We do this step using Edit Metadata and Group Categorical Values modules.

Then, we clean the text using Preprocess Text module. The cleaning will reduce the noise in the dataset, help you find the most important features, and improve the accuracy of the final model. We remove stopwords - very common words such as "the" or "a" - as well as numbers, special characters, duplicated characters such as "coool", email addresses and URLs. We also convert the text to lowercase, lemmatize the words and detect sentence boundaries that will be indicated by "|||" symbol in in pre-processed text.

Optionally, you can apply custom list of stopwords, or custom C# syntax regular expression to replace substrings, as well as remove words by part of speech: nouns, verbs or adjectives.

## Step 2: Extracte numeric feature vectors from pre-processed text

Machine learning algorithms typically expect numeric feature vectors as input instead of free-form text. We use Extract N-Gram Features module to transform the text data to such format. This module takes a column of whitespace-separated words, computes a dictionary of words, or N-grams of words, and maps each word, or N-gram to distinct column.

## Step 3: Train classification or regression model

## Step 4: Score and validate the model

## Step 5: Deploy the model to production

## Multi-language support and language detection

## Topic modeling
