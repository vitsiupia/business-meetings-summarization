# Business meetings summarization

Since meetings are very important in companies, and time is even more important, it would be useful to have something that allows producing a reliable summary of the content of the meeting in the quickest way. 
We decided to use current NLP techniques to make this task possible.

We did the project according to the following steps:
1. Defining the problem
2. Choosing a database
3. Initial data preparation
4. Choosing a model
5. Data pre-processing (unique for a model)
6. Transfer learning
7. Validation
8. Conclusions

## 1. Defining the problem
What the model should do? What type of data will be fed into the model? What we want to achieve?

* The model should produce a summary of given meeting transcript
* We will fed the model with long text data
* We want to make a fairy good summarizer by training the NLP model with custom data

## 2. Choosing the database
We find a strong candidate: AMI corpus - https://groups.inf.ed.ac.uk/ami/corpus/
To get from the corpus only full transcripts and summaries we use code from this repository: https://github.com/gcunhase/AMICorpusXML

## 3. Initial data preparation
The data had to be pre-prepared to make our later work easier. We had to extract the text files that were useful in our project from all the rest, found in the AMI corpus.
For this purpose we used scripts based on os, shutil, str modules.

## 4. Choosing a model
At this stage we reviewed scientific literature and guidebooks that described a similar problem to ours to gather information on the techniques used and finally evaluated their applicability in the context of our project.

First model we choose was **T5** from **Transformers**.
You can see our struggles in the folder "xx". There is pre-processing of the data (to make it usable for this model) and the model training and evaluation itself. 

We decided **not to choose this model** ❌

That's because it only accepted up to 512 tokens as input, while our transcripts had up to 10000. There are solutions to this problem, but they have a drawbacks:
* Truncation of the document, which is commonly used for classification - we do more complex task: summarization, and we will loss a lot of data by doing it this way.
* Splitting the document into fragments of 512 tokens (they can overlap), processing each fragment separately, and then combining the activations using a sophisticated model.
* Using a two-stage model, where the first stage searches for the relevant fragment, which is passed to the second stage for answer extraction - sadly, cascade errors often occur with this solution.

**OUR SOLUTION**: 
Unlike the above approaches, the **Longformer** allows processing long sequences without truncating or splitting them into fragments, allowing a much simpler approach of concatenating the available context and processing it in a single pass ✔

## 5. Data pre-processing
You can see our noteboks with data pre-processing inside T5 and Longformer folders respectively.

## 6. Transfer learning
Our notebooks with fitting the model to the AMI data you will find in the folders.

At this stage, we encountered a new problem: ⏰
While training the Longformer, which is a large model, we encountered a technological problem. To train this model on the least demanding settings, at least 24 GB of memory was required on the graphics card. Unfortunately, the free version of Google Colab only provided half of this value. Neither of us had such a powerful graphics card, so we were forced to train the model on the CPU, with an additional 32 GB of RAM. However, training on the CPU took much longer than on the GPU. One epoch alone took almost an hour. 

**OUR APPROACH**: 
In order to speed up model training and allow evaluation at the end of the epoch, which would have included calculating Rouge Score values and generating a graph, we decided to dispense with this part of the process. On our available computing resources, calculating the Rouge Score for just 29 examples would have taken another hour. Therefore, we decided to skip this step to save time and continue training the model.

## 7. Validation
With our human eye, we judged the correctness of the summary.

A document with a comparison of the summaries produced by the Longformer as well as the T5 model after a different number of epochs can be found in the repository.

## 8. Conclusions

* Longformer surpasses the T5 model in generating better summaries of long documents. By taking into account long word relationships, Longformer is able to better understand and reflect the content of the original text in the summary.
* Experimenting with Longformer's hyperparameters, such as longer training, changing the batch size, etc., can lead to improved summaries. You may find that adjusting these parameters for specific data leads to better summary results.
* The BigBird model, like Longformer, is another model capable of processing long documents. A comparison between Longformer and BigBird can provide valuable information on the performance and quality of the summaries generated by these models.

In conclusion, Longformer offers a better ability to generate summaries of long documents than the T5 model. Comparing it with the BigBird model and experimenting with Longformer's hyperparameters can bring additional benefits in terms of the quality of generated summaries.
