# Criterions
## Intro
The aim was to make two linear regression models and two quadratic regression models based on different loss functions and compare the results.
## The Linear Regression Models :
Both the models give very close results compared to the weights that were actually used to build the datasets. The loss curve for the model which uses the cost function |y-z| seems to have a linear decrease in the start, and then it suddenly changes its slope, whereas the loss curve of the other model with the cost function as |y-z|^3 has a very steep decrease just in the beginning and then the loss remains more or less the same in the rest of the epochs. With this we can say that the second model seems to be a bit faster than the first one.
## The Quadratic Regression Models :
Similar to what we observed in the case of Linear Regression models, here also, the loss function of the higher power cost function i.e. |y-z|^7 decreases much steeply as compared to the lower power cost function |y-z|^4, and it also remains more or less the same after. We can say that the model with cost function |y-z|^7 is faster than the other model.
## Problems :
The higher power cost functions are faster, but a problem that can occur with them is overflow. We can solve this by scaling the y-values of the train data before training and then finally scaling up the final weigths that are learned by the same factor. But note that this may also decrease the accuracy as the error in weigths is also scaled up.
Other way could be to update the weights with new random values during the training process in case of overflow. This has been implemented in my code.
