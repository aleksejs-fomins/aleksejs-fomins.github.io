---
title: "A critique of psychometrics"
categories:
  - Meta
tags:
  - Psychometrics
  - Science
  - Opinion
author_profile: false
header:
  overlay_color: "#000"
  overlay_filter: "0.5"
  overlay_image: /assets/images/eskay-lim-PK6NDtsYy00-unsplash.jpg
  actions:
    - label: "Learn More"
      url: "/terms/"
  caption: "Photo credit: [**Unsplash**](https://unsplash.com)"
---

Recently I have briefly worked in psychometrics, on the task of measuring a concept related to human well-being. Reading in detail about a dozen publications in the field has left me utterly puzzled. I am unable to establish a bridge between the scientific method I was trained in as a physicist, and the practices employed in psychometric research. In this post I attempt to structure my confused thoughts, and present, as clearly as I can, the questions that I am unable to find the answer to. For some of the questions I struggle to see how they can have an answer

## What is psychometrics

## What is the typical procedure employed in psychometrics

There is a large demand for measuring quantities related to human health and intelligence. This may include, for example, general health, severity of a mental disorder, general intelligence (e.g. IQ), or skill in a certain specialized discipline, such as calculus or art history of the 16th century. It is then the task of a psychometrist to construct and test a metric that would measure the quantity of interest. As far as I have understood, modern psychometrists use the following procedure to construct a measure they are interested in:

1. Existence of a **latent construct** is assumed - a scalar latent variable which quantifies the concept of interest. This variable is assumed to be monotonic: lower or greater value of this variable would imply that the person is more or less depressed, more or less healthy, more or less intelligent, depending on the measurement objective.
2. A **survey** of a few dosen questions is assembled to assess *assess the latent construct*. In other words, it is assumed that the answers to the questions in such a survey are in a monotonic relationship with the latent variable. For example, the question "Rate 1-10 how hard it is for you to get out of bed in the morning" could be used to assess depressiveness, under the assumption that higher depressiveness would generally result in higher difficulty of getting out of bed, and thus higher-valued answer to this question.
  * Multiple questions are used, as individual answers may be noisy, and individual questions may be selective only in some range of the latent variable. For example, a question to compute "2+2*2" would be useful to detect people who have not studied at primary school, but would not be useful at distinguishing the skill of trained mathematicians, as all of them would answer the question correctly, given enough motivation to do so.
3. The researcher would assume a **statistical model**, linking a person's latent construct value to the probabilities of different answers that person might choose when filling the survey. A typical model used for this purpose is the [Rasch model](https://en.wikipedia.org/wiki/Rasch_model) and its extensions. Given the person's latent construct variable, and a difficulty parameter of the question, the model would provide a probability distribution of all possible answers to that question by that person.
4. A survey is conducted, **collecting answers** from multiple participants. Question answers from all participants are collected into a single dataset.
5. The selected **model is fitted** to the dataset, simultaneously estimating latent variables for each person, and question-specific parameters (such as difficulty).
6. The individual **latent variables are reported** on the same scale. Since, for example, a depressiveness of 5.3 may be hard to interpret, the latent values are frequently reported as quantiles within the general population. This may be written as "person A is among the 10% most depressive people", or "among the 20% least depressive people".

## How and how precisely are the measurement targets typically defined?

In order to measure something, it needs to first be defined. AFAIK, concepts of interest to psychometricians, such as depression, health, or intelligence have reasonably vague definitions. How do psychometricians justify trying to make precise measurements of quantities that do not have precise definitions?

## How is unidimensionality of the construct assessed? What are the typical assessement outcomes?

In order for it to be meaningful to measure the latent construct with a scalar variable, it must be demonstrated that that the construct is unidimensional. In other words, a single scalar variable is sufficient to describe individual differences in regard to this construct. Mathematicians have a handy tool for checking if objects can be placed on a monotonic unidimensional scale: if, for any three objects, it is possible to order those objects in the order from lowest to highest value of the latent variable, then the variable is unidimensional. For example, obesity as measured by percent body fat is a unidimensional construct - any number of people can be unambiguously sorted with regards to such a metric. However, diet quality is not a unidimensional construct. It is not possible to objectively order people who have severe caloric deficiency, iron deficiency and B-vitamin deficiency respectively in the order from best to worst diet quality. This indicates that calories, minerals and vitamins are different dimensions of diet quality, and diet quality is thus a multidimensional construct.

## How is the validity of the questions assessed? What are the typical assessment outcomes?

This question really has two parts:
1. Are the answers to the question at all related to the latent construct? Do these quantities co-vary?
2. How strongly is the question related to the latent construct? How strongly are they confounded by sources unrelated to the latent construct?

I am tempted to immediately answer that the validity of such questions cannot be assessed, as it is unclear to me, how a question can be demonstrated to measure a certain quantity, when that quantity is loosely defined at best, or defined through the very same questions at worst. However, the field of psychometrics has invested a large amount of effort in establishing good practices on how such questions are to be selected and tuned, so we might as well take a look.



Next, let us have a look how psychometrics deals with confounding variables. First, let me indulge a little in brainstorming what these confounding variables could be. Some people might not answer the questions truthfully. Some people may be in a bad mood. Some people may feel differently about their experience in different parts of the day. Some may not be able to describe the experience well or consistently. Note that these are mostly related to the measurement of health, and to a far lesser extent to the measurement of intelligence. It is not unreasonable to assume that having only one correct answer and a strong incentive to answer correctly would naturally act to reduce the confounding in the case of IQ.

## How are the statistical models validated?

Do psychometricians justify the choice of their model, outside of the model itself? It fits, after a bit of tweaking. Ok, so what? Fitness alone is not meaningful. Importantly, any residual variance not fitted by the model is assumed to be noise. But what if it is not? There needs to be at least some amount of mechanistic understanding as to why a given model should be meaningful in explaining the process in question. Is this ever done?
