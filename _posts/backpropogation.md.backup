# Intuitive Understanding of Backpropogation --

Backpropogation is one of the most skipped/overlooked algorithms in Deep Learning Space. Involves Calculus, Gradient tracking and stuffs. In this blog, i will try to decode Backpropogation intuitively. Let's see how it goes,

----

I will be explaining in 3 parts :
- FeedForward Algorithm [Optional To Read]
- Intuition + Maths
- Backpropogation Algorithm 


## FeedForward Algorithm :

Imagine we have a car. When we press gas pedal, due to the system gas pedal is integrated in, the result of pressing a pedal would follow several inhouse functions and finally the output that we desire in this case, propelling of vehicle.  

[![simple-things.gif](https://i.postimg.cc/nhDfCP3j/simple-things.gif)](https://postimg.cc/FYNB6D5N)

In a feedforward algorithm, consider a N1 neuron got value (x) & weight (w1). Then the output will be `w1.x` for this neuron that will be fed into next neuron N2, who will be having some weight (w2). And understandbly the output of N1 neuron, will be input for N2 neuron and output of N2 will be : (outputN1)*w2.

> Note : for understanding purpose, we have omitted bias.

If you look closely, N1,N2,N3...N100. Output till the last neuron will be linear in fashion. That is y=mx. Where y will be output of our N100 neuron, m will be N100 weight, and x will be input from N99 neuron. Every neuron in this case, got simple product of previous neuron. That means, its very linear function. It will struggle with nonlinear features which is requirement of real world.  
 
To solve this, widely used technique is `activation functions`. Which includes non-linearity in the input and output values. There are many activation functions like relu, leaky relu, sigmoid, tanh..etc. Mostly used activation function is Relu.  

Some activation functions -->

[![sample-activation-functions-square.webp](https://i.postimg.cc/nVdBRRpP/sample-activation-functions-square.webp)](https://postimg.cc/c6nv6cKR)


So if we go above and add activation function that will make things more reasonable.

N1 -> output -> activation_function(output) -> input to N2 --- goes on...


[![feedforward.png](https://i.postimg.cc/sfkBMtc7/feedforward.png)](https://postimg.cc/4n14M2jx)


As you can see in the image, input neurons gets multiplied by the weights and now are input to Next layer neuron. 

> Note : When one neuron getting more than one input. We use summation like shown in above image and and then use activation function over it.


And Finally the last layer, the outputs are evaluated on real labels. 


## BackPropogation Algorithm :



Another super great analogy, i heard was, imagine you are an intern in a company, you do some task and that task was head over to manager then to senior manager then so so ..to CEO. Company suffers in loss. CEO blames senior manager, senior manager blames manager, manager blames you. You then correct based on the feedback and then resend results. --- Well, this is how backpropogation works. Cost function evaluates how bad a network performed and backpropogation computes gradient telling neurons how much to change inorder to correct! This goes on and on. Until network has optimized weights.  


In theory, its easy to understand. But maths related to it what makes it confusing.

> Basic calculus is required for it. 

Backpropogation, even by the name of it. We can understand that it follows back to front approach of calculations.

Some basic shorthands we will use :

- C : Cost Function
- σ : Activation Function
- ŷ : prediction
- z : output of previous layer. Superscript shows number of layer.   


[![backprop1.png](https://i.postimg.cc/dQWgNPbh/backprop1.png)](https://postimg.cc/vDVPDjHy)


In the above pic, we can see we are taking partial differentiation of Cost function with respect to ŷ. 