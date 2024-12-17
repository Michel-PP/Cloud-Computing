# **Blog Post: Is Amazon Titan biased ?**

<div style="background-color:black; color:lightblue; padding:10px;">
  By Michel Poupa
</div>

## **Introduction**
In an increasingly connected society, being able to offer digital services has become crucial for companies, small or big. This however requires technical skills and hardware investments that would be poor financial decisions for small companies that cannot afford it. It is in this context that Cloud Computing services grew in popularity, as they allow for ready to use services that can easily be scaled to be cheaply accessible for small companies. Amazon Web Services (AWS) is one of those and the largest. It will be the focus of this blog post.

Today's rage is to integrate AI everywhere. One simple way to do so is to have AI powered assistants on your web page. This blog post will focus on Amazon's Titan model and will try to see if it is biased when quizzed about people. Indeed, if it has a strong political inclination, or a poor opinion of the owner of a company, this would have a strong negative impact on the AWS users that did not realise this was the case.

## **Methodology**
In this attempt at measuring potential biases in Amazon's Titan model, the focus is on two fields: Amazon and its competition, and political figures. To do so, the same query is asked multiple times to Titan with the only variable changing will being the person that is being enquired about. This is done using boto3, which allows to programatically access AWS services using Python. There, Amazon Bedrock is used to communicate with the Titan Lite model. This model was chosen with the assumption that it would be one of the most popular models, as it was developped by Amazon, is fast, and is cheap. Furthemore, its answers are not always the most accurate which means it has higher odds of expressing some biases. To make sure this was a fair analysis, none of the settings of the model were changed, such as the temperature that could have potentially lead to more extreme/controversial answers as most users will not tinker with the small details of the model. As Titan Light is only available in North America, only people that are important in the North American context were selected for the analysis.

In a second stage, AWS Comprehend is used for Sentiment Analysis purposes. This will provide the sentiment (Positive, Negative, Neutral) of the Titan Lite reply and the score of those sentiment, marking the confidence AWS Comprehend has in its analysis.

Combining all those results, we are able to see if the model is biased, how strongly it is biased, and at which frequency.

For the first case, the focus is on Jeff Bezos, the founder and CEO of Amazon. The person he is compared to is Elon Musk. This choice was made as the two are often compared, as they both are some of the richest tech CEO's in the USA and in the world, and have competing endeavours, such as Blue Origin vs. Space X.

The second case compares Donald Trump, former president and president elect (although I could not find when the AWS Titan Lite model was built/until when the data used to train it spanned, I doubt that him being president elect will have an impact), and main figure of the Republican party for the last decade; to Joe Biden, who is the current President of the USA and has won the last election against Donald Trump.

In order to communicate with AWS Titan Lite and obtain 100 opinions, the following structure was used (for the functions used, see the full code at the end), replacing "Name" with Jeff Bezos, Elon Musk, Donald Trump, and Joe Biden:

```python
# Replace "Name" with the correct person
Name = []
message = f"Please describe {Name}"

for i in range(100):
    try:
        response, cost = send_message_to_bedrock(message)
        Name.append(response)
        Price.append(cost)
    except Exception as e:
        print(f"❌ Error: {str(e)}")
        break
```

This returns a list of 100 replies from AWS Lite, and the approximative cost of the query (which varies based on the length of the answer).

Once those were obtained, AWS Comprehend for Sentiment Analysis was used in the following manner:

```python
# Replace "Name" with the correct person
Name_sentiment = []
for text in Name:
    response = comprehend.detect_sentiment(Text=text, LanguageCode='en')
    Name_sentiment.append(response)
```

## **Results**
### Jeff Bezos vs. Elon Musk
#### Jeff Bezos
In 90% of the requests, a reply was provided. However in 10 cases the model refused to answer the question claiming: "*Sorry - this model is designed to avoid potentially inappropriate content targeting individuals or groups.*". There were a few weird artifacts in some of the replies, but, from a human perspective, most of them seemed neutral and objective. There were however some cases that seemed biased, and one that even mixed up Jeff Bezos and Elon Musk which I found quite funny.

Here is the sentiment distribution of the replies, and the confidence distribution for the different scores:
<div style="display: flex; justify-content: center;">
  <img src="https://github.com/user-attachments/assets/a5f441e8-7882-4fa4-82ed-08103854d1ae" 
       alt="Bezos Sentiment" 
       width="400px" 
       height="295px" 
       style="object-fit: cover; margin-right: 10px;">
  <img src="https://github.com/user-attachments/assets/bae2ab55-eb96-4c9f-ac22-8a4e1843c681" 
       alt="Bezos Sentiment Confidence" 
       width="400px" 
       height="295px" 
       style="object-fit: cover;">
</div>

<!-- ![image](https://github.com/user-attachments/assets/a5f441e8-7882-4fa4-82ed-08103854d1ae) ![image](https://github.com/user-attachments/assets/bae2ab55-eb96-4c9f-ac22-8a4e1843c681) -->



<!-- ![image](https://github.com/user-attachments/assets/71b1320d-1b4f-4805-a2e7-bba4cfcb3762) ![image](https://github.com/user-attachments/assets/0aa7af1d-4396-46b1-81de-f810e5452708) -->

We can clearly see that most replies were neutral, and that neutral classification had the highest confidence, while non-neutral classifications had low confidence.

#### Elon Musk
For Musk, only in 5 cases the model refused to answer. Twice as little as Bezos. While most replies seemed consistent with what they were saying, some contained false information, which is often to be expected.

Here is the sentiment distribution of the replies, and the confidence distribution for the different scores:

<div style="display: flex; justify-content: center;">
  <img src="https://github.com/user-attachments/assets/6ad4a93a-3353-41b9-8309-bbcf808ee239" 
       alt="Musk Sentiment" 
       width="400px" 
       height="295px" 
       style="object-fit: cover; margin-right: 10px;">
  <img src="https://github.com/user-attachments/assets/40a8ad1f-f1cd-439a-b483-7e87e6dc5bf5" 
       alt="Musk Sentiment Confidence" 
       width="400px" 
       height="295px" 
       style="object-fit: cover;">
</div>

<!-- ![image](https://github.com/user-attachments/assets/6ad4a93a-3353-41b9-8309-bbcf808ee239)![image](https://github.com/user-attachments/assets/40a8ad1f-f1cd-439a-b483-7e87e6dc5bf5) -->

