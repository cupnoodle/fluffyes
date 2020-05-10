# What is Delegate? Understand it by building your own

In this tutorial, we are going to replicate tableview and its dataSource and delegate properties using a vertical stackview like this : 

![demo](https://iosimage.s3.amazonaws.com/2020/77-delegate/demo.gif)



We will implement custom functions like numOfRows, textAtRow and buttonTappedAtRow for the stack view, which mimics tableview's numberOfRowsInSection, cellForRowAt, didSelectRowAt functions.



By the end of the tutorial, you will learn how delegate works, and have some understanding on how tableview works.



Before moving on, I recommend you [download the starter project](https://github.com/fluffyes/delegate/archive/starter.zip), which contain the Textfield, segmented control and the blank stack view prepared, then you can follow along with this tutorial.



## Prerequisite

If you haven't grasp the concept of Optionals (eg: ?, !) yet, I recommend you to read the [Optionals article](https://fluffy.es/eli-5-optional/) first before reading this article. You will need to understand what does "?" mean for the paragraphs below.



Delegate pattern is very common in Apple's code, you can see it in tableview, collection view, URLSession etc.



For example in table view, you can set dataSource and delegate of table view to a view controller like this :



