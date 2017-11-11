# Unsupervised Stock Market Features Construction using Generative Adversarial Networks(GAN)
Deep Learning constructs feature using only raw data. The leaned representation of the data outperforms expert features for many modalities including Radio Frequency ([Convolutional Radio Modulation Recognition Networks](https://arxiv.org/pdf/1602.04105.pdf)), computer vision ([Convolutional Deep Belief Networks for Scalable Unsupervised Learning of Hierarchical Representations](https://www.cs.princeton.edu/~rajeshr/papers/icml09-ConvolutionalDeepBeliefNetworks.pdf)) and audio classification ([Unsupervised feature learning for audio classification using convolutional deep belief networks](http://www.robotics.stanford.edu/~ang/papers/nips09-AudioConvolutionalDBN.pdf)). 
# GAN 
For a full review of a GAN: [Generative Adversarial Nets](https://arxiv.org/pdf/1406.2661.pdf) 
In the case of Convolutional Neural Networks (CNN), the data representation is learned in a supervised fashion with respect to a task such as classification. For a typical CNN to generalize to unseen data in requires very large quantities of data. CNNs do not provide any mechanism to encourage searching the space that lays between training examples. Provide enough representative data CNN do generalize sufficiently to produce good results on out of bag data. GANs can lean features in a unsupervised fashion. The competitive learning process for GANs results in more the possible feature space being explored and therefore more repressive features. This reduces that potential for features being overfitted to the training data and in turn means that a classification algorithm trained on the features will generalize on a smaller amount of data. In fact, GANs promote generalization beyond the training data, as will be seen. 
![gan.png]({{site.baseurl}}/media/gan.png)
The Generator is trained to generate data that looks like historical price data of the target stocks over a uniform distribution. The Discriminator is trained to tell the difference between the data from the Generator and the real data. The loss from the Discriminator (how the Discriminator has learned to tell if a sample in real or fake) is used to train the Generator to defeat the Discriminator. The competition between the Generator and the Discriminator forces the Discriminator to distinguish random from real variability while the Generator learns to map a distribution into the sample space.    
# Approach 

**Data**
Historical prices of stocks are likely not very predictive of the future price of the stock, but it is free data. Technical indicators are calculated using the historical prices of stocks. Not being a trader I don't know the validity of technical indicators, but if a sufficient number investors use technical indicators to invest such that they move the market, then the historical prices of stocks should suffice to predict the direction of the market correctly more then 50% of the time.

**Training**
The GAN is trained on 96 stocks off the Nasdaq. Each stock is normalized using a 20-day rolling window (data-mean)/(max-min). The last 356 days (1.4 years) of trading are held out as a test set. Time series of 20 day periods are constructed and used as input to the GAN. Once the GAN is finished training, the activated weighs from the last convolutional lays in the Discriminator is used as the new representation of the data. These features have information that is useful for telling whether a given sample is real or fake. They are not guaranteed to be predictive of the direction of the stock market. XGBoost is trained to classify whether the stock will go up or down over some period of time using the features extracted from the Discriminator.

**Testing**
The data the was held out in the training phase is run through the Discriminator portion of the GAN and the activated weights from the last convolutional layer are extracted. The extracted features are then classified using the trained XGBoost model. Multiple models are trained to predict over different periods of time.

**Downloading Data**
The first step is to download the historicl stock data. I use ![Quandl](www.quandl.com) as my data source. They provide the basic stock data for free. You will need to create a free account to get an api key. 

```python
import urllib2
import os

#Your API key 
quandl_api_key = 'Your API KEY'

#List of stocks to download
stock_symbols = ['AAPL', 'GOOG', 'COST', 'FB', 'INTU', 'ISRG']

url = 'https://www.quandl.com/api/v3/datasets/WIKI/{}.csv?api_key={}'

if not os.path.exists('./stock_data'):
	os.makedirs('./stock_data')
    
for symbol in stock_symbols:
	try:
    	stock_url = url.format(symbol, quandl_api_key)
        response = urllib2.urlopen(stock_url)
        quotes = response.read()
        lines = quotes.strip().split('\n')
        with open(os.path.join('./stock_data', symbol), 'wb') as f:
        	f.write(lines)

```

**Results** 
The confusion matrix shows the results of the model's classification. The perfect confusion matrix would only have predictions on the main diagonal. Each number off the main diagonal is a misclassification.  


**Predictions of Up or Down movement over 10 Days**

The predictions over a 10 day period are quite good. 

![alt text]({{site.baseurl}}/media/XGB_GAN_Confusion_Matrix_Up_Or_Down_Over_10_Days_normalize.png)

**Predictions of Up or Down movement over 1 Day**

Predicting over a short time interval seems to be harder. Results loss significant accuracy when trying to predict the next day movement of the stock. 

![alt text]({{site.baseurl}}/media/XGB_GAN_Confusion_Matrix_Up_Or_Down_Over_1_Days_normalize.png)

**Predictions 10% Gain Over 10 Days**

Just knowing that the stock will go up or down is of limited use. A lot of stocks will go up in a day but an investor will want to only buy the stocks that will go up the most, maximizing returns. This time the XGBoost model was trained to predict stocks that would go up by 10% or more over the following 10 days. 

![cm]({{site.baseurl}}/media/XGB_GAN_Confusion_Matrix_Up_Or_Down_Over_10_Days_10_percent_normalize.png)
