**Presidential speech project**

This project is an analysis of many presidential speeches from the site millercenter.org.  The site contains the text of at least one speech (and almost always more) given by each of the 44 presidents.

The goal is to cluster presidents into three clusters based on the words that they use in their speeches.  Words are given more weight if (1) they are frequent in that president's speeches *and* (2) they are more common in that president's speeches than in the English language generally.  Based on this weight function, presidents who use similar words are said to be similar and are grouped together.

The conclusion is that the three clusters of presidents are more or less consecutive in time.  The first cluster runs from George Washington to Andrew Johnson; the second from Ulysses S. Grant to Herber Hoover; and the third from Franklin D. Roosevelt to Donald Trump.  A holdout test set is used to verify that the clustering does not depend very much on the particular choice of speeches for each president.

Based on the weight function, we can calculate the "characteristic words" of each cluster.  

*NOTE: the characteristic words and clusters reported below correspond to one run of the program.  Due to the random nature of the program, the same words and clusters will not necessarily result from every run.*

**Characteristic words for cluster A (Washington through Johnson):** intrusted, heretofore, effectually, herewith, effectual, intercourse, salutary, continuance, constitution, expedience

Counting only nouns and words with semantic content: constitution, postmaster, duties, framers, treasury, commerce, vessels, harbors, cargoes, provisions

Analysis: Presidents are talking a lot more formally in this time period. They are still talking about the recently-devised constitution and its framers. Trade, commerce, and shipping are clearly important; perhaps the main function of the federal government at this time is to manage commerce?

**Characteristic words for cluster B (Grant through Hoover):** speedily, gratifying, continuance, intrusted, tariff, appropriation, furnish, guaranty, enactment, appropriations

Counting only nouns and words with semantic content: tariff, appropriation, appropriations, navy, receipts, postmaster, fiscal, commissioners, legislation, treasury

Conclusion: There seems to be more talk of financial issues than with the other clusters (tariff, appropriation, receipts, fiscal, treasury).

**Characteristic words for cluster C (FDR through Trump):** america's, america, americans, american, peace, bipartisan, allies, nation's, strengthen, nation

Conclusion: this is the "America as a great power and centralized government" cluster. America has allies, wants to preserve international peace, and has a strong identity as a nation. There is also some partisan division that leads to valuing "bipartisan" legislation.

We can also examine the characteristic words for individual presidents.  In both cases, I count only words that have semantic content and are not obscure proper names:

**Characteristic words for Trump:** DACA, opioid, scouts, dreamers, opioids, jamboree, Trump, trafficker, overprescribing, businessperson

**Characteristic words for Obama:** bipartisan, hardworking, Trayvon, Americans, undocumented, Isil's, Isil, childcare, autoworker, deficits

**Weight function:**

The "weight" for each word is calculated as follows. The goal is to have a weight that reflects (1) the relative incidence of the word in the speech corpus compared with its usual incidence in the English language, (2) the overall incidence of the word in the speech corpus. Thus, a word that is highly unusual ("supercalifragilisticexpealidocious") but only appears once in the speech corpus is not that interesting. In contrast, a word that is very common, but appears only the usual amount in the corpus ("the") is also not interesting. The weight for each word is computed as a product of these two factors.

Getting into the details: To compute the weight, the incidence count of the word is first tallied in the combined speeches of a president. Using the wordfreq library, each word's frequency in English is obtained (actually the log of the English frequency), so that it can be compared with the log of the frequency in the speeches. Using the statsmodels library, a least squares regression is used to find the linear relationship between these two logs.

This linear relationship can then be used to compute the predicted English frequency for a word that has the given corpus frequency.  This predicted English frequency can be compared to the word's actual English frequency. The difference between predicted and actual (using subtraction) is essentially a measure of how much more common the word is in the speech corpus compared to the English language. A weight value of 1 would mean that the word is "e" times more common in the speech corpus as compared with English.

Next, the number of words in each set of speeches can range from 20,000 to 800,000, so the number of appearances is multiplied by (800,000 / length) to get the number that would appear in 800,000. Then, the log is taken. This number is multiplied by the weight to get the final weight value. This takes into account that we care more about words that appear many times as opposed to only once.

**Similarity score**

The similarity score for two presidents M and N is calculated as follows: the similarity score for a word is the minimum of the word's weight for president M and for president N. Only words that appear in both president M and president N's speeches will get a score.

The total similarity between two presidents is the sum of the individual similarities for each word.

Thus, presidents who say more words will have higher similarities with other presidents.  However, since they have higher similarities with *all* other presidents, this should not unduly affect the clustering process.

**Spectral Clustering**

A 44 x 44 affinity matrix is constructed based on similarity scores, and Spectral Clustering is used to perform the cluster analysis.  Spectral Clustering was selected because: (1) it lets us easily select the number of clusters, unlike Affinity Propagation which requires us to manually set a "preference" level (2) Agglomerative clustering was tried, but it tended to produce one really big cluster with one or two singletons.

Before clustering, the speeches are randomly divided into a training and a testing set for each president.  The clustering is then performed twice: once for the training data and once for the testing data.  (In fact, there is no real difference in the way the two data sets are treated.)

**Compute Cluster Overlap**

We then compute the overlap between the clusters from the training and testing data. The overlap is defined by trying to match the clusters in the training set to the clusters in the testing set to see how much overlap there is.  All presidents which can be matched from one set to the other are counted as matches, and the ratio of matches to the total number of presidents is reported.

**Characteristic Words**

To find the characteristic words in a cluster (or any group of presidents - it doesn't have to be one of the clusters we found), we look at each word that appears in more than half of the presidents.  We then find the median weight of that word.  If there are an even number of words, we use the lower of the middle two weights rather than the average.  Next, we find the median weight of the word in the complement set (the presidents who are not in our chosen group.)  We subtract the two median weights to find the score of that word.  The words with the highest score are those that are much more characteristic of our chosen group than of the other presidents who are not in that group.
