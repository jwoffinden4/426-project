# 426-project
State of the State Addresses, Topics, and Political Party Relationships

### Introduction
Every year, the President of the United States gives an address before Congress called "The State of the Union". In this speech, he reviews the events and accomplishments of the previous year, and conveys his plans to address various topics and issues at the national level. Each governor across the country holds the same event at the state level, commonly referred to as "The State of the State", where they address more local matters. I was intrigued by this concept because the governors have a closer and more direct connection with their constituents than the President does, so what they say and address should theoretically give greater insight on the well-being of the country's citizens. 

I hope to answer two main questions from this project: 
- First, are there any common themes, policies, or topics that are addressed by many or all states? If so, what are they?
- Second, are there discrepancies in policy and protocol addressed in each "State of the State" address that be attributed to the governors' political party affliation?

In theory, the answers to these questions should provide some insight for the national leaders about what is on the mind of United States citizens generally and what are already doing to handle these topics. The first question hopes to illustrate common threads between states in terms of their attention and resources. The second question hopes to discover how each political party approaches these topics and highlight any differences. 

### Data
Originally, my plan was to include all "State of the State" speeches given from 2015 to 2020, totaling 300 speeches, and compile them into a Microsoft Excel spreadsheet, along with the year, each state, governor, and the governor's political party. This spreadsheet would be converted into a CSV file that I could reach into Python. I webscraped the transcipts of most of these speeches from either the governor's website or a local news station, then formatted each speech so that it could fit in one cell within the spreadsheet. There were two factors, however, that caused me to change my approach. The first problem was the presence of missing values. During my research, I discovered that several states, such as Nevada and North Carolina, only give "State of the State" addresses biannually instead of yearly. Other addresses were not given due to extraneous events such as impeachment, circumstances relating to recent inauguration, or the Covid-19 pandemic. Two transcripts could not be found due to a lack of public access. Overall, there were 32 out of those 300 transcipts that were labeled as missing values, but I was satisfied enough with how much data I had that I was willing to move on without them.

The second problem came from issues with the spreadsheet's capacity. It is worth noting that it is not uncommon for these speeches to last over an hour. Depending on the size and font of the text, each speech can range anywhere from 4 to over 20 pages of text each. Trying to stuff 278 speeches of that size into individual cells proved to be too much for the CSV file to handle. Microsoft Excel has character limits for each cell, so signficiant portions of each transcript were automatically deleted. Then when I converted the spreadsheet to a CSV file, the file crashed and wiped out whole cells of data.

In the interest of time (it took close to 12 hours of research to find and format all the texts previously), I adjusted my approach by doing two things. First, I chose to work with only the most recent "State of the State" address given by each state's governor. Most of these came from 2020 since they were given before the Covid-19 pandemic, but since 2020 was the "off year" for those states who gave their speeches biannually and Covid-19 impacted the delivery of two more, I settled with the 2019 address for those states instead. Then, to accommodate for the character limits, I chose to make each individual paragraph its own seperate cell and connect it to the appropriate state within the CSV file. This time, there were no complications with converting the spreadsheet into a CSV file and it read into Python just fine.

After the data was read in and stored in a Pandas dataframe, I cleaned the text of each address by first lowercasing all words so that my models would not be subjected to differeniating the same word with a capital letter. Afterwards, I eliminated the most common forms of punctuation, such as commas, semicolons, and periods. Finally, I eliminated any generic stopwords as defined by the package "nltk", then erased a list of more stopwords that I felt would detract from the model's accuracy. The final table has two columns, "Party" referring to the governor's political party, and "Text" referring to the text of each paragraph. This table has 6052 rows of seperate paragraphs. 3383 of those rows were given by Republican governors, while the other 2669 were given by Democrat governors.

### Methods & Results
To address the question about common themes, policies, or topics that are addressed by many or all states, I did a KMeans clustering analysis on this data. I chose to use this analysis because it takes the text data and clusters them into different groups. I can then find the most common words within each cluster to extract common themes found across all 50 "State of the State" addresses. Political party is the only factor we are weighing the text against, so scaling the data isn't an issue. The main disadvantage to KMeans is that I have to go a little out of my way to find the optimal number of clusters, and as I'll address later, the optimal number of clusters is not always easy to find.

I used a count vectorizer to first find the most frequently used words in the texts. When looking at the 15 most common non-stopwords in our data, "education" was at the top of the list. "School" was used the third-most frequently, and "students" was twelfth. That gives an early hint that education was high on governors' minds. The presence of "jobs" and "working" on this list indicate many discussions about the workforce, "budget" and "tax" highlight the governors' fiscal plans, and "health" shows the significance of healthcare in their speeches. The next step was to find the optimal number of clusters for our data. I used a TF-IDF vectorizer to transform the data, then compared the WCSS to a range of potential cluster amounts (see Figure 1 included in the repository). That way, I can minimize the WCSS without using too many clusters. The plot in this figure shows that there isn't a consistent "elbow bend" in the curve, but there is a small bend at k=30 before continuing the usual curve. I decided to use 30 clusters when continuing this analysis.

I performed a singular value decomposition analysis to make sure there was good seperation among the groups. It is a lot harder to tell how much seperation there is when using 30 clusters, but given that there were two clusters that were notably distinct from the rest of the points in the scatterplot, I trusted the data to say the seperation between the clusters is good enough to move forward (see Figure 2 included in the repository). Finally, I used the top words in each cluster to interpret potential themes. From these clusters, I identified seven different "themes" to group these clusters into. The largest group of clusters talked about the workforce, such as creating job opportunities, supporting small businesses, and improving the economy (found in Clusters 1, 3, 12, 18, 19, 23, and 29). Education was the next most popular theme, including mentions about teacher pay, higher education funding, and other beneficial services for children with concerns like mental health and addiction (found in Clusters 5, 14, 15, 17, and 26). Another common theme was about healthcare, including making it affordable and providing appropriate services (found in Clusters 9 and 26). I was a little surprised that the budget and fiscal-related matters didn't get more attention from the clusters, as only Cluster 2 seemed to mention it. I did not expect to see so many clusters talk about quality of living, including public safety, individual rights, and second-chance policies (found in Clusters 0, 4, 11, 13, 22, 24, and 25). It was refreshing to see that the family unit had a couple clusters, mentioning about topics such as foster care and child support (found in Clusters 10 and 20). The last theme was more fluffy, feel-good acknowledgements, such as recognizing distinguished officials, providing optimism for the future, and hopes of God blessing America (found in Clusters 6, 7, 8, 16, 21, 27, and 28). This last theme will be addressed in greater depth in the conclusion. The presence of these themes validated their earlier presence in the list of most commonly used words.

To address the question about identifying speeches by political party, I chose to build a Multinomial Naive Bayes classification model. I tested several other models, including Decision Tree, Random Forest, and Gradient Boosting classification as well. I ultimately chose to use the Multinomial Naive Bayes classification model because it had the best accuracy score. It helps that it also had the biggest AUC and precision scores as well. Gradient Boosting had the best F1 and recall scores. Multinomial Naive Bayes naturally works well with text classification, so it made sense that it would have the best accuracy score. It also computed the model the fastest of the four tested models. However, although it was the most accurate, the score was only about 61%. It's other metrics are dangerously low, such as a 70% F1 score, 61% precision score, had a recall of 81%, and an AUC of 64%. While those metrics were consistenly the best among the other models I tested, these metrics are certainly not good enough to trust this model when trying to predict political party based on the transcript. Since the other models performed at around the same range, I consider this to be good evidence that there is not that big of a difference in content between the two political parties as we might be accustomed to thinking. More commentary as to why my model performed so poorly is in my conclusion.

The closest I got to finding any notable distinctions between political party was by comparing the model's claimed "important" features. I found these values for each party by finding the 20 least likely words to appear in the opposite party's talks. One potential connection in the "important" Democrat list comes from the words "child", "raise", "young", and "college". There might be the occasional instance that "programs" applies to this as well. I don't think this directly correlates to the KMeans cluster theme of education, but it seems like Democratic governors have more interest about the future citizens of their states. Republican governors seem more likely to use words such as "rate", "growth", "expand", and "investment". It is harder to pinpoint these words to any one theme, but these words likely indicate a deeper focus on the trajectory and speed of the productivity and prosperity of the state. In other words, they care about how things are going now. This could apply to any of the KMeans cluster themes that I found. Based on these features, while there were no notable differences in the actual policies spoken of, there does seem to be a distinction in the emphasized timeframe. Democrats seem more likely to focus on the future, while Republicans prioritize the present.

### Conclusion
I wish really quickly to stay with the topic of political vision in the context of the model's important features. This is a trend I believe is generally true in today's political landscape. One excellent example is the response to the current Covid-19 pandemic. To me, Republican governors have been much more willing to loosen restrictions and open things back up. This is because their concern tends to focus on improving the economy immediately and lowering unemployment rates as soon as possible. In other words, their vision is more centered on the "now". Democratic governors have been much more apprehensive about loosening restrictions and have stricter protocols. This is because their concern tends to focus on preventing hospital overloading to assist healthcare workers and extending the lives of at-risk individuals. In other words, their vision is more driven on the future. The point of this is not to decide what is the right policy. Rather, it is to identify a real, in-the-moment example that supports the findings of my classification model. This "now or later" distinction may not hold true in every scenario, but I believe it is generally accurate.

I believe that there is a correlation between the results of the KMeans analysis and the lack of effectiveness with the Multinomial Naive Bayes classification model. The goal of the classification model was to determine discrepencies in subject matter between political parties. However, the results from the KMeans analysis showed that most of the clusters included the same general concepts (e.g. education, economy, healthcare, etc.). It is likely that each cluster was saturated with the same subject matter because every governor was addressing those same topics. Because the economy and finances and education and such were spoken of by both Democrat and Republican governors, I believe that the model had a hard time making any distinctions between the two parties when the text mentioned those subjects, and hence struggled to make accurate predictions on the speaker's political party. It is worth pointing out that the themes from the clusters were not the only items on the governors' agendas. Other topics that I noticed were addressed by individual governors while collecting the data include water quality, infrastructure, climate change, and abortion, among others. I infer that because not enough governors talked about these matters in general, let alone by those of the same party, the classification model struggled to illustrate any relationships between these issues and a particular party.

To build off this theory, another possible idea why the classification model struggled was the lack of consistency of agendas within the party. On the surface, there were too many things in common between Democrats and Republicans in their subject matter, but when diving a little deeper, there are too many discrepancies within each party for the model to accurately keep track. For example, California and Delaware both have Democratic governors, but have significantly different demographics (e.g. population, area, industry). With those in mind, Gavin Newsome, the governor of California, is going to handle affairs for education and taxes and healthcare much differently from how John Carney handles things in Delaware. A possible future analysis could be examining the inner-party differences in policies for the same political topics.

In the end, I believe I got the results that I did because of the lack of consistency between certain parties talking about certain subjects in a certain way. The KMeans analysis revealed there were too many commonalities in subject matter between the political parties, making it very difficult for the classification model to identify any contrasts. Classification models are likely not the best way to get answers to these particular questions listed in the introduction. Some of the blame, however, should be shared with how I collected the data. If I could change anything about how I conducted this project, I would change the way I organized my data. The major flaw with making each paragraph its own row in the dataset is that it gave the "fluff" of the speech too much weight in the classification model. These "State of the State" addresses aren't strictly business-related. As demonstrated by seven different clusters from my KMeans analysis, the governors will take to recognize certain individuals in attendance, tell some "feel-good" stories about the accomplishments of state constituents, and give lots and lots of thank yous. Most of the time, these moments are given their own individual paragraphs, which means they get their own individual rows in the dataset. Because each row is assigned to a political party, a line that says "Thank you" could be labeled as Republican when the phrase has no political connotation. That is a third possibility as to why the classification model's accuracy is so low. Another possible future analysis could be to repeat this project in the exact same way, but manually clean the data by reading through the addresses and filtering out the fluff. I believe that could have a direct impact on improving the model's accuracy.
