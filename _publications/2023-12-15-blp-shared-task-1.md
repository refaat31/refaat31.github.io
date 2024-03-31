---
title: "Team CentreBack at BLP-2023 Task 1: Analyzing performance of different machine-learning based methods for detecting violence-inciting texts in Bangla"
collection: publications
permalink: /publication/2023-12-15-blp-shared-task-1
excerpt: 'This is a system description paper on BLP-2023 Shared Task 1 (Violence-inciting text detection in Bangla)'
date: 2023-12-15 
venue: 'First Workshop on Bangla Language Processing (BLP-2023) at EMNLP'
paperurl: 'https://aclanthology.org/2023.banglalp-1.18/'
citation: 'Alamgir & Haque, BanglaLP 2023'
# citation: 'Refaat Mohammad Alamgir and Amira Haque. 2023. Team CentreBack at BLP-2023 Task 1: Analyzing performance of different machine-learning based methods for detecting violence-inciting texts in Bangla. In Proceedings of the First Workshop on Bangla Language Processing (BLP-2023), pages 168–173, Singapore. Association for Computational Linguistics.'
---

Abstract
------

Like all other things in the world, rapid growth of social media comes with its own merits and demerits. While it is providing a platform for the world to easily communicate with each other, on the other hand the room it has opened for hate speech has led to a significant impact on the well-being of the users. These types of texts have the potential to result in violence as people with similar sentiments may be inspired to commit violent acts after coming across such comments. Hence, the need for a system to detect and filter such texts is increasing drastically with time. This paper summarizes our experimental results and findings for the shared task on The First Bangla Language Processing Workshop at EMNLP 2023 - Singapore. We participated in the shared task 1 : Violence Inciting Text Detection (VITD). The objective was to build a system that classifies the given comments as either non-violence, passive violence or direct violence. We tried out different techniques, such as fine-tuning language models, few-shot learning with SBERT and a 2 stage training where we performed binary violence/non-violence classification first, then did a fine-grained classification of direct/passive violence. We found that the best macro-F1 score of 69.39 was yielded by fine-tuning the BanglaBERT language model and we attained a position of 21 among 27 teams in the final leaderboard. After the competition ended, we found that with some preprocessing of the dataset, we can get the score up to 71.68.

